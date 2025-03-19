## 아래의 타임라인을 기준으로 PITR 시나리오

아래 사유로 인해 T3으로 복구한 후에도 T0으로 다시 복구할 수 있습니다.

-   **백업 독립성**:
    -   WAL-G의 풀 백업(base\_000000010000000000000001, base\_000000010000000000000002)은 각각 독립적으로 저장됨.
    -   T5에서 T3으로 복구했다고 해서 백업 1(T0)이 손상되거나 삭제되지 않음 (이전 질문에서 삭제하지 않았다고 가정).
-   **WAL 파일 유지**:
    -   T0 시점 복구에 필요한 WAL 파일(000000010000000000000001.lz4 등)이 /backups/wal\_005에 남아 있다면, T0으로의 복구 가능.
    -   T3 복구 시 사용된 WAL 파일은 별도로 관리되며 T0 복구에 영향을 주지 않음.
-   **복구 메커니즘**:
    -   PostgreSQL의 PITR(Point-in-Time Recovery)은 매번 새롭게 데이터 디렉토리를 초기화하고 복구하므로, 이전 복구(T5)가 다음 복구(T0)에 영향을 미치지 않음.

아래 시나리오 시점 기준으로 백업과 복구를 진행하였다.

```
시간 축 (2025-03-10 기준)
--------------------------------------------------------->
T0       T1       T2       T3       T4       T5       T6

[상태]
T0: 초기 데이터 (Alice, Bob)
    └─── 백업 1 (base_000000010000000000000001)
T1: 트랜잭션 1 (Charlie 추가, Alice 삭제 → Bob, Charlie)
T2: 첫 번째 복구 (T0 시점으로 복구 → Alice, Bob)
T3: 트랜잭션 2 (David 추가, Bob → Bobby → Alice, Bobby, David)
    └─── 백업 2 (base_000000010000000000000002)
T4: 데이터 손실 (users 테이블 삭제)
T5: 두 번째 복구 (T3 시점으로 복구 → Alice, Bobby, David)
T6: 현재 상태 (복구 완료)

[그래프 요소]
- T0, T3: 백업 시점 (Backup Point)
- T2, T5: 복구 시점 (Restore Point)
- 화살표: 트랜잭션 및 복구 흐름
```

### T0: 초기 데이터 (Alice, Bob)

#### └─── 백업 1 (base\_000000010000000000000001)

> 데이터 생성

```
postgres@lkmpg:~$ sudo -u postgres psql -c "CREATE DATABASE test\_db;"  
sudo -u postgres psql -d test\_db -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name TEXT);"  
sudo -u postgres psql -d test\_db -c "INSERT INTO users (name) VALUES ('Alice'), ('Bob');"  
CREATE DATABASE  
CREATE TABLE  
INSERT 0 2  
postgres@lkmpg:~$  
```

> 백업전 백업 리스트 확인

```
postgres@lkmpg:/backups$ sudo -u postgres /usr/local/bin/wal-g backup-list --config=/etc/wal-g.yaml  
INFO: 2025/03/10 19:57:59.568992 List backups from storages: \[default\]  
INFO: 2025/03/10 19:57:59.569111 No backups found  
```

> full backup 진행

```
postgres@lkmpg:/backups$ sudo -u postgres /usr/local/bin/wal-g backup-push /var/lib/postgresql/15/main --config=/etc/wal-g.yaml  
INFO: 2025/03/10 19:58:05.767393 Backup will be pushed to storage: default  
INFO: 2025/03/10 19:58:05.786174 Calling pg\_start\_backup()  
INFO: 2025/03/10 19:58:05.836034 Initializing the PG alive checker (interval=1m0s)...  
INFO: 2025/03/10 19:58:05.836082 Starting a new tar bundle  
INFO: 2025/03/10 19:58:05.836099 Walking ...  
INFO: 2025/03/10 19:58:05.836214 Starting part 1 ...  
INFO: 2025/03/10 19:58:06.039177 Packing ...  
INFO: 2025/03/10 19:58:06.043156 Finished writing part 1.  
INFO: 2025/03/10 19:58:06.043209 Starting part 2 ...  
INFO: 2025/03/10 19:58:06.043229 /global/pg\_control  
INFO: 2025/03/10 19:58:06.043375 Finished writing part 2.  
INFO: 2025/03/10 19:58:06.043409 Calling pg\_stop\_backup()  
INFO: 2025/03/10 19:58:06.062901 Starting part 3 ...  
INFO: 2025/03/10 19:58:06.064087 backup\_label  
INFO: 2025/03/10 19:58:06.064416 tablespace\_map  
INFO: 2025/03/10 19:58:06.065080 Finished writing part 3.  
INFO: 2025/03/10 19:58:06.065743 Querying pg\_database  
INFO: 2025/03/10 19:58:06.139750 Wrote backup with name base\_000000010000000000000004 to storage default  
```

> 백업전 백업 리스트 확인

```
postgres@lkmpg:/backups$ sudo -u postgres /usr/local/bin/wal-g backup-list --config=/etc/wal-g.yaml  
INFO: 2025/03/10 19:58:08.309726 List backups from storages: \[default\]  
backup\_name                   modified                  wal\_file\_name            storage\_name  
base\_000000010000000000000004 2025-03-10T19:58:06+09:00 000000010000000000000004 default  
```

### T1: 트랜잭션 1 (Charlie 추가, Alice 삭제 → Bob, Charlie)

> 트랜잭션 발생

```
postgres@lkmpg:/backups$ psql -d test\_db  
psql (15.12 (Ubuntu 15.12-1.pgdg24.04+1))  
Type "help" for help.

test\_db=# INSERT INTO users (name) VALUES ('Charlie');  
DELETE FROM users WHERE name = 'Alice';  
INSERT 0 1  
DELETE 1  
test\_db=#  
test\_db=# \\q
```

### T2: 첫 번째 복구 (T0 시점으로 복구 → Alice, Bob)

> 백업본 무결성 검증

```
postgres@lkmpg:~/15/main/pg\_wal/archive\_status$ sudo -u postgres /usr/local/bin/wal-g wal-verify integrity --config=/etc/wal-g.yaml  
INFO: 2025/03/10 20:01:38.149658 Current WAL segment: 000000010000000000000005  
INFO: 2025/03/10 20:01:38.152044 Building check runner: integrity  
INFO: 2025/03/10 20:01:38.152739 Detected earliest available backup: base\_000000010000000000000004  
INFO: 2025/03/10 20:01:38.152989 Running the check: integrity  
\[wal-verify\] integrity check status: OK  
\[wal-verify\] integrity check details:  
+-----+--------------------------+--------------------------+----------------+--------+  
| TLI | START | END | SEGMENTS COUNT | STATUS |  
+-----+--------------------------+--------------------------+----------------+--------+  
| 1 | 000000010000000000000004 | 000000010000000000000004 | 1 | FOUND |  
+-----+--------------------------+--------------------------+----------------+--------+
```

> db 중지 및 복구진행

```
postgres@lkmpg:/backups/wal\_005$ sudo systemctl stop postgresql  
postgres@lkmpg:/backups/wal\_005$ rm -rf /var/lib/postgresql/15/main/\*  
postgres@lkmpg:/backups/wal\_005$ wal-g backup-fetch /var/lib/postgresql/15/main^C  
```

> 백업본 copy

```
postgres@lkmpg:/backups/wal\_005$ sudo -u postgres /usr/local/bin/wal-g backup-fetch /var/lib/postgresql/15/main base\_000000010000000000000004 --config=/etc/wal-g.yaml  
INFO: 2025/03/10 20:04:42.095101 Selecting the backup with name base\_000000010000000000000004...  
INFO: 2025/03/10 20:04:42.095210 Backup to fetch will be searched in storages: \[default\]  
INFO: 2025/03/10 20:04:42.982699 Finished extraction of part\_001.tar.lz4  
INFO: 2025/03/10 20:04:42.985059 Finished extraction of pg\_control.tar.lz4  
INFO: 2025/03/10 20:04:42.991694 Finished extraction of backup\_label.tar.lz4  
INFO: 2025/03/10 20:04:42.991735  
Backup extraction complete.  
```

> 복구 커맨드 작성  
> 복구시점 잘 기입해야함

```
postgres@lkmpg:/backups/wal\_005$ echo "restore\_command = '/usr/local/bin/wal-g wal-fetch %f %p --config=/etc/wal-g.yaml'" >> /etc/postgresql/15/main/postgresql.conf  
postgres@lkmpg:/backups/wal\_005$ echo "recovery\_target\_action = 'promote'" >> /etc/postgresql/15/main/postgresql.conf  
postgres@lkmpg:/backups/wal\_005$ echo "recovery\_target\_time = '2025-03-10 19:58:00'" >> /etc/postgresql/15/main/postgresql.conf  
```

> 복구신호 파일 생성

```
postgres@lkmpg:/backups/wal\_005$ sudo -u postgres touch /var/lib/postgresql/15/main/recovery.signal  
```

> db 기동 및 데이터 복구 확인

```
postgres@lkmpg:/backups/wal\_005$ sudo systemctl start postgresql

postgres@lkmpg:/backups/wal\_005$ psql -d test\_db  
psql (15.12 (Ubuntu 15.12-1.pgdg24.04+1))  
Type "help" for help.

test\_db=# select \* from users;  
id | name  
\----+-------  
1 | Alice  
2 | Bob  
(2 rows)

test\_db=# \\q
```

### T3: 트랜잭션 2 (David 추가, Bob → Bobby → Alice, Bobby, David)

## └─── 백업 2 (base\_000000010000000000000002)

> 복구 진행 후 타임라인이 변경되면서 history 파일 생성

```
postgres@lkmpg:/backups/wal\_005$ ls -lrt  
total 348  
\-rw------- 1 postgres postgres 66017 Mar 10 19:58 000000010000000000000003.lz4  
\-rw------- 1 postgres postgres 263 Mar 10 19:58 000000010000000000000004.00000028.backup.lz4  
\-rw------- 1 postgres postgres 66115 Mar 10 19:58 000000010000000000000004.lz4  
\-rw------- 1 postgres postgres 66451 Mar 10 19:59 000000010000000000000005.lz4  
\-rw------- 1 postgres postgres 66091 Mar 10 20:03 000000010000000000000006.lz4  
\-rw------- 1 postgres postgres 66400 Mar 10 20:04 000000010000000000000007.lz4  
\-rw------- 1 postgres postgres 69 Mar 10 20:05 00000002.history.lz4
```

> 트랜잭션 발생

```
postgres@lkmpg:/backups/wal\_005$ psql -d test\_db  
psql (15.12 (Ubuntu 15.12-1.pgdg24.04+1))  
Type "help" for help.

test\_db=# INSERT INTO users (name) VALUES ('David');  
UPDATE users SET name = 'Bobby' WHERE name = 'Bob';  
INSERT 0 1  
UPDATE 1  
test\_db=#  
test\_db=# select \* from users;  
id | name  
\----+-------  
1 | Alice  
36 | David  
2 | Bobby  
(3 rows)

test\_db=# \\q
```

> full backup

```
postgres@lkmpg:/backups/wal\_005$ sudo -u postgres /usr/local/bin/wal-g backup-push /var/lib/postgresql/15/main --config=/etc/wal-g.yaml  
INFO: 2025/03/10 20:10:48.339067 Backup will be pushed to storage: default  
INFO: 2025/03/10 20:10:48.367754 Calling pg\_start\_backup()  
INFO: 2025/03/10 20:10:48.404914 Initializing the PG alive checker (interval=1m0s)...  
INFO: 2025/03/10 20:10:48.404942 Starting a new tar bundle  
INFO: 2025/03/10 20:10:48.404954 Walking ...  
INFO: 2025/03/10 20:10:48.405065 Starting part 1 ...  
INFO: 2025/03/10 20:10:48.676452 Packing ...  
INFO: 2025/03/10 20:10:48.681736 Finished writing part 1.  
INFO: 2025/03/10 20:10:48.681759 Starting part 2 ...  
INFO: 2025/03/10 20:10:48.681773 /global/pg\_control  
INFO: 2025/03/10 20:10:48.681875 Finished writing part 2.  
INFO: 2025/03/10 20:10:48.681880 Calling pg\_stop\_backup()  
INFO: 2025/03/10 20:10:48.701675 Starting part 3 ...  
INFO: 2025/03/10 20:10:48.701762 backup\_label  
INFO: 2025/03/10 20:10:48.701768 tablespace\_map  
INFO: 2025/03/10 20:10:48.701810 Finished writing part 3.  
INFO: 2025/03/10 20:10:48.702337 Querying pg\_database  
INFO: 2025/03/10 20:10:48.813585 Wrote backup with name base\_000000020000000000000006 to storage default  
postgres@lkmpg:/backups/wal\_005$  
postgres@lkmpg:/backups/wal\_005$ sudo -u postgres /usr/local/bin/wal-g backup-list --config=/etc/wal-g.yaml  
INFO: 2025/03/10 20:10:59.929577 List backups from storages: \[default\]  
backup\_name modified wal\_file\_name storage\_name  
base\_000000010000000000000004 2025-03-10T19:58:06+09:00 000000010000000000000004 default  
base\_000000020000000000000006 2025-03-10T20:10:48+09:00 000000020000000000000006 default
```

### T4: 데이터 손실 (users 테이블 삭제)

> 테이블 삭제 트랜잭션 발생

```
postgres@lkmpg:/backups/wal\_005$ psql -d test\_db  
psql (15.12 (Ubuntu 15.12-1.pgdg24.04+1))  
Type "help" for help.

test\_db=# DROP TABLE users;  
DROP TABLE  
test\_db=# \\q
```

### T5: 두 번째 복구 (T3 시점으로 복구 → Alice, Bobby, David)

> db 중지 및 복구본 copy

```
postgres@lkmpg:/backups/wal\_005$ sudo systemctl stop postgresql  
postgres@lkmpg:/backups/wal\_005$ rm -rf /var/lib/postgresql/15/main/\*  
postgres@lkmpg:/backups/wal\_005$ sudo -u postgres /usr/local/bin/wal-g backup-fetch /var/lib/postgresql/15/main base\_000000020000000000000006 --config=/etc/wal-g.yaml  
INFO: 2025/03/10 20:12:26.020692 Selecting the backup with name base\_000000020000000000000006...  
INFO: 2025/03/10 20:12:26.020804 Backup to fetch will be searched in storages: \[default\]  
INFO: 2025/03/10 20:12:26.962325 Finished extraction of part\_001.tar.lz4  
INFO: 2025/03/10 20:12:26.963851 Finished extraction of pg\_control.tar.lz4  
INFO: 2025/03/10 20:12:26.967247 Finished extraction of backup\_label.tar.lz4  
INFO: 2025/03/10 20:12:26.967996  
Backup extraction complete.  
postgres@lkmpg:/backups/wal\_005$
```

> 복구신호 파일 생성 및 복구시점 변경  
> "recovery\_target\_time = '2025-03-10 20:10:00'"

```
postgres@lkmpg:/backups/wal\_005$ sudo -u postgres touch /var/lib/postgresql/15/main/recovery.signal  
postgres@lkmpg:/backups/wal\_005$ vi /etc/postgresql/15/main/postgresql.conf
```

> DB기동 및 T5시점 데이터 확인

```
postgres@lkmpg:/backups/wal\_005$ sudo systemctl start postgresql

postgres@lkmpg:/backups/wal\_005$ psql -d test\_db  
psql (15.12 (Ubuntu 15.12-1.pgdg24.04+1))  
Type "help" for help.

test\_db=# select \* from users;  
id | name  
\----+-------  
1 | Alice  
36 | David  
2 | Bobby  
(3 rows)

test\_db=#
```

### T6: 현재 상태 (복구 완료)
