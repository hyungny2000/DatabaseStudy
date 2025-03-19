**환경**: Ubuntu, PostgreSQL 15

1.  **pgBackRest 설치 확인**  
    sudo apt update  
    sudo apt install pgbackrest  
    pgbackrest version

```
postgres@lkmpg:~$ sudo apt update
Get:1 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Hit:2 http://kr.archive.ubuntu.com/ubuntu noble InRelease
Get:3 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [670 kB]
Get:4 http://kr.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:5 https://apt.postgresql.org/pub/repos/apt noble-pgdg InRelease [129 kB]
Get:6 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [130 kB]
Get:7 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [8,960 B]
Get:8 http://security.ubuntu.com/ubuntu noble-security/main amd64 c-n-f Metadata [6,912 B]
Get:9 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [726 kB]
Get:10 http://kr.archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
Get:11 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [146 kB]
Get:12 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 Packages [309 kB]
Get:13 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Components [212 B]
Get:14 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 c-n-f Metadata [432 B]
Get:15 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [819 kB]
Get:16 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [177 kB]
Get:17 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [51.9 kB]
Get:18 http://security.ubuntu.com/ubuntu noble-security/universe amd64 c-n-f Metadata [16.9 kB]
Get:19 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 Components [212 B]
Get:20 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 c-n-f Metadata [448 B]
Get:21 http://kr.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [919 kB]
Get:22 http://kr.archive.ubuntu.com/ubuntu noble-updates/main Translation-en [208 kB]
Get:23 http://kr.archive.ubuntu.com/ubuntu noble-updates/main amd64 Components [151 kB]
Get:24 http://kr.archive.ubuntu.com/ubuntu noble-updates/main amd64 c-n-f Metadata [13.4 kB]
Get:25 http://kr.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Packages [759 kB]
Get:26 http://kr.archive.ubuntu.com/ubuntu noble-updates/restricted Translation-en [153 kB]
Get:27 http://kr.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Components [212 B]
Get:28 http://kr.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 c-n-f Metadata [464 B]
Get:29 http://kr.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1,037 kB]
Get:30 http://kr.archive.ubuntu.com/ubuntu noble-updates/universe Translation-en [261 kB]
Get:31 http://kr.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [364 kB]
Get:32 http://kr.archive.ubuntu.com/ubuntu noble-updates/universe amd64 c-n-f Metadata [25.8 kB]
Get:33 http://kr.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:34 http://kr.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 c-n-f Metadata [656 B]
Get:35 http://kr.archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [208 B]
Get:36 http://kr.archive.ubuntu.com/ubuntu noble-backports/restricted amd64 Components [216 B]
Get:37 http://kr.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [20.0 kB]
Get:38 http://kr.archive.ubuntu.com/ubuntu noble-backports/universe amd64 c-n-f Metadata [1,256 B]
Get:39 http://kr.archive.ubuntu.com/ubuntu noble-backports/multiverse amd64 Components [212 B]
Fetched 7,484 kB in 10s (764 kB/s)

Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
131 packages can be upgraded. Run 'apt list --upgradable' to see them.
postgres@lkmpg:~$
postgres@lkmpg:~$ sudo apt install pgbackrest
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libssh2-1t64
Suggested packages:
  pgbackrest-doc check-pgbackrest
The following NEW packages will be installed:
  libssh2-1t64 pgbackrest
0 upgraded, 2 newly installed, 0 to remove and 131 not upgraded.
Need to get 673 kB of archives.
After this operation, 1,862 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://kr.archive.ubuntu.com/ubuntu noble/main amd64 libssh2-1t64 amd64 1.11.0-4.1build2 [120 kB]
Get:2 https://apt.postgresql.org/pub/repos/apt noble-pgdg/main amd64 pgbackrest amd64 2.54.2-1.pgdg24.04+1 [553 kB]
Fetched 673 kB in 5s (126 kB/s)
Selecting previously unselected package libssh2-1t64:amd64.
(Reading database ... 124284 files and directories currently installed.)
Preparing to unpack .../libssh2-1t64_1.11.0-4.1build2_amd64.deb ...
Unpacking libssh2-1t64:amd64 (1.11.0-4.1build2) ...
Selecting previously unselected package pgbackrest.
Preparing to unpack .../pgbackrest_2.54.2-1.pgdg24.04+1_amd64.deb ...
Unpacking pgbackrest (2.54.2-1.pgdg24.04+1) ...
Setting up libssh2-1t64:amd64 (1.11.0-4.1build2) ...
Setting up pgbackrest (2.54.2-1.pgdg24.04+1) ...
Processing triggers for libc-bin (2.39-0ubuntu8.4) ...
Processing triggers for man-db (2.12.0-4build2) ...
Scanning processes...
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
postgres@lkmpg:~$
postgres@lkmpg:~$ pgbackrest version
pgBackRest 2.54.2
postgres@lkmpg:~$
```

**2\. pgBackRest **설정 파일 구성****

```
postgres@lkmpg:/usr/bin$ sudo mkdir -p -m 770 /var/log/pgbackrest
postgres@lkmpg:/usr/bin$ sudo chown postgres:postgres /var/log/pgbackrest
        sudo mkdir -p /etc/pgbackrest
        sudo mkdir -p /etc/pgbackrest/conf.d
        sudo touch /etc/pgbackrest/pgbackrest.conf
        sudo chmod 640 /etc/pgbackrest/pgbackrest.conf
        sudo chown postgres:postgres /etc/pgbackrest/pgbackrest.conf
postgres@lkmpg:/usr/bin$
postgres@lkmpg:/usr/bin$
postgres@lkmpg:/usr/bin$
postgres@lkmpg:/usr/bin$ ls -lrt /etc/pgbackrest/pgbackrest.conf
-rw-r----- 1 postgres postgres 0 Mar 15 12:10 /etc/pgbackrest/pgbackrest.conf
```

```
[demo] 
pg1-path=/var/lib/postgresql/15/main 

[global] 
repo1-path=/var/lib/pgbackrest 
repo1-ret조  
https://pgbackrest.org/user-guide.html#installation



