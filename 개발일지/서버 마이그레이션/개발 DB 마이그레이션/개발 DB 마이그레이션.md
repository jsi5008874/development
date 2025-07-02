
### postgresql 14.7버전으로 버전업
현재 개발서버 DB는 9.6 버전으로 오래된 버전인데 운영서버 DB와 버전을 통일하기 위해 버전업을 해서 설치했다.

```
#wget 설치
sudo dnf install wget
# 설치 후 다운로드
wget https://ftp.postgresql.org/pub/source/v14.7/postgresql-14.7.tar.gz

# 개발 도구 그룹 전체 설치
sudo dnf groupinstall "Development Tools"

# 14.7 소스 다운로드
cd /tmp
wget https://ftp.postgresql.org/pub/source/v14.7/postgresql-14.7.tar.gz
tar -xzf postgresql-14.7.tar.gz
cd postgresql-14.7

# 컴파일 및 설치 (기존과 다른 경로)
./configure --prefix=/usr/local/pgsql14
make
sudo make install

# postgres 사용자로 14.7 초기화
sudo -u postgres /usr/local/pgsql14/bin/initdb -D /usr/local/pgsql14/data
# 소유권 설정
sudo chown -R postgres:postgres /usr/local/pgsql14

#postgre 접속
sudo -u postgres /usr/local/pgsql14/bin/psql

# SQL 명령어 실행 
CREATE USER 유저ID WITH PASSWORD '비밀번호' CREATEDB LOGIN; CREATE DATABASE DB명 OWNER 유저ID; GRANT ALL PRIVILEGES ON DATABASE DB명 TO 유저ID; \q
```

### postgre 설정 파일 옮기기
기존 서버의 /usr/local/pgsql 폴더에 있는 각종 설정파일을 tar로 압축하여 옮긴다.

```
tar -xzf /usr/local/pgsql14 pgsql_backup.tar.gz
```

새 서버 /usr/local/pgsql14에서 압축 해제 후 DB 재시작(설정 적용)

### data 심볼릭 링크 활성화

```
# 심볼릭 링크 타겟 폴더 생성
mkdir /data/pgsql14/data

# 심볼릭 링크 지정
sudo ln -s /data/pgsql14/data /usr/local/pgsql14/data
```
심볼릭 링크를 지정하는 이유는 데이터는 용량을 많이 필요로 하기 때문이다.

![[Pasted image 20250626102354.png]]
사진으로 보다 싶이 /data에 1TB가 할당되어 용량이 넉넉하기 때문에 /data 하위에 폴더를 생성하여
심볼릭 링크를 지정하여 데이터가 /data 폴더에 쌓이도록 하는 것이다.

### WAL(Write-Ahead Log) 아카이브 폴더 지정
WAL 아카이브는 트랜잭션 로그 파일들이 저장되는 곳으로 DB 데이터와 같이 용량을 많이 필요로 하는 폴더이다.

postgresq.conf 파일에서 /data 쪽에 쌓이도록 명령어를 설정한다.
```
archive_command = 'test -d /data/backup2/db/together_pg_archive/date +20\%y\%m\%d || mkdir /data/backup2/db/together_pg_archive/date +20\%y\%m\%d && cp %p /data/backup2/db/together_pg_archive/date +20\%y\%m\%d/%f' 
```
'test -d /data/backup2/db/together_pg_archive/date +20\%y\%m\%d
먼저 /data/backup2/db/together_pg_archive에 오늘 날짜로 폴더가 있는지 확인

mkdir /data/backup2/db/together_pg_archive/date +20\%y\%m\%d
폴더가 없으면 오늘 날짜 폴더 생성

cp %p /data/backup2/db/together_pg_archive/date +20\%y\%m\%d/%f' 
오늘 날짜 폴더에 WAL 파일 저장


지정한 폴더에 트랜잭션 로그들이 쌓이는 것을 볼 수 있다.
### 데이터 백업
기존 서버에서 데이터를 백업한다.
```
sudo -u postgres pg_dump -d 데이터베이스명 | gzip > database_backup.sql.gz
```

새 서버에서 데이터 복원
```
sudo -u postgres psql -d 데이터베이스명 < 압축파일명
```
이후 

```
# 포스트그레 재시작
sudo -u postgres /usr/local/pgsql14/bin/pg_ctl restart -D /usr/local/pgsql14/data
```