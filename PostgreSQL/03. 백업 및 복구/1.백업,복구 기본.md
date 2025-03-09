#### Postgresql 백업 & 복구 

### 데이터 setting 
- sample 데이터가 없으므로 postgresql 에서 기본 제공하는 dvd rental 데이터 download

```shell
1. data download

postgres@lkmpg:~/tempdb$ wget https://www.postgresqltutorial.com/wp-content/uploads/2019/05/dvdrental.zip
--2025-02-22 13:29:50--  https://www.postgresqltutorial.com/wp-content/uploads/2019/05/dvdrental.zip
Resolving www.postgresqltutorial.com (www.postgresqltutorial.com)... 172.67.205.41, 104.21.61.22, 2606:4700:3031::ac43:cd29, ...
Connecting to www.postgresqltutorial.com (www.postgresqltutorial.com)|172.67.205.41|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 550906 (538K) [application/zip]
Saving to: ‘dvdrental.zip’

dvdrental.zip         0%[                    ]       0  --.-KB/s
dvdrental.zip        23%[===>                ] 128.80K   619KB/s
dvdrental.zip       100%[===================>] 537.99K  1.14MB/s    in 0.5s

2025-02-22 13:29:51 (1.14 MB/s) - ‘dvdrental.zip’ saved [550906/550906]

2. unzip
postgres@lkmpg:~/tempdb$ unzip dvdrental.zip
Archive:  dvdrental.zip
  inflating: dvdrental.tar
  
postgres@lkmpg:~/tempdb$ ll
total 3320
drwxrwxr-x 2 postgres postgres    4096 Feb 22 13:31 ./
drwxr-xr-x 4 postgres postgres    4096 Feb 22 13:29 ../
-rw-rw-r-- 1 postgres postgres 2835456 May 12  2019 dvdrental.tar
-rw-rw-r-- 1 postgres postgres  550906 May 12  2019 dvdrental.zip

3. postgres 계정으로 postgres 스키마에 데이터 import 

postgres@lkmpg:~/tempdb$ pg_restore -U postgres -d postgres dvdrental.tar
postgres@lkmpg:~/tempdb$

4. 접속 후 데이터 생성확인
postgres@lkmpg:~/tempdb$ psql
psql (15.12 (Ubuntu 15.12-1.pgdg24.04+1))
Type "help" for help.

postgres=# \d
                     List of relations
 Schema |            Name            |   Type   |  Owner
--------+----------------------------+----------+----------
 public | actor                      | table    | postgres
 public | actor_actor_id_seq         | sequence | postgres
 public | actor_info                 | view     | postgres
 public | address                    | table    | postgres
 public | address_address_id_seq     | sequence | postgres
 public | category                   | table    | postgres
 public | category_category_id_seq   | sequence | postgres
 public | city                       | table    | postgres
 public | city_city_id_seq           | sequence | postgres
 public | country                    | table    | postgres
 public | country_country_id_seq     | sequence | postgres
 public | customer                   | table    | postgres
 public | customer_customer_id_seq   | sequence | postgres
 public | customer_list              | view     | postgres
 public | film                       | table    | postgres
 public | film_actor                 | table    | postgres
 public | film_category              | table    | postgres
 public | film_film_id_seq           | sequence | postgres
 public | film_list                  | view     | postgres
 public | inventory                  | table    | postgres
 public | inventory_inventory_id_seq | sequence | postgres
 public | language                   | table    | postgres
 public | language_language_id_seq   | sequence | postgres
 public | nicer_but_slower_film_list | view     | postgres
 public | payment                    | table    | postgres
 public | payment_payment_id_seq     | sequence | postgres
 public | rental                     | table    | postgres
 public | rental_rental_id_seq       | sequence | postgres
 public | sales_by_film_category     | view     | postgres
 public | sales_by_store             | view     | postgres
 public | staff                      | table    | postgres
 public | staff_list                 | view     | postgres
 public | staff_staff_id_seq         | sequence | postgres
 public | store                      | table    | postgres
 public | store_store_id_seq         | sequence | postgres
(35 rows)

```



