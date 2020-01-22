# LDAP Service Architecture Design

## Servers

| Role | Domain | IP |
|---|---|---|
| Provider | ldap1.example.com | 192.168.9.101 |
| Provider | ldap2.example.com | 192.168.9.102 |
| Consumer | consumer.example.com | 192.168.9.103 |

---

## Vagrant

[Vagrantfile](Vagrantfile)

**서버 시작**

```bash
vagrant up
```

**서버 접속**

```bash
# provider 1
vagrant ssh ldap1
# provider 2
vagrant ssh ldap2
# consumer
vagrant ssh consumer
```

**서버 종료**

```bash
vagrant halt
```

---

## 공통 설정

### Hosts 설정

각 서버 `/etc/hosts`마다 전체 주소 도메인 네임(**FQDN**)을 등록한다.  
FQDN이 LDAP 설정값과 일치하지 않으면 LDAP 서버를 실행할 수 없다.

```bash
cat > /etc/hosts << EOF
192.168.9.101    ldap1.example.com        ldap1
192.168.9.102    ldap2.example.com        ldap2
192.168.9.103    consumer.example.com    client
EOF
```

호스트 네임을 확인한다.

```bash
# 호스트 네임 확인 명령
hostname -f
# 호스트 네임 변경 명령
hostnamectl set-hostname ldap1
```

### 패키지 설치

OpenLDAP 라이브러리, 서버, 클라이언트를 설치한다.

```bash
yum install -y openldap openldap-servers openldap-clients;
```

---

## 인증서

### RootCA 인증서

**개인키 생성**

```bash
openssl genrsa -out rootca.key
```

**인증요청서 정보 작성**

```bash
cat > rootca.csr.conf << EOF
[ req ]
default_bits            = 2048
default_md              = sha256
distinguished_name      = req_distinguished_name
prompt                  = no
encrypt_key             = no

[ req_distinguished_name ]
countryName                     = "KR"
stateOrProvinceName             = "Seoul"
localityName                    = "Seoul"
0.organizationName              = "MyCompany"
organizationalUnitName          = "TopUnit"
commonName                      = "RootCA"
EOF
```

**인증요청서 생성**

```bash
openssl req -new -key rootca.key -config rootca.csr.conf -out rootca.csr
```

**RootCA 인증서 생성**

```bash
openssl req -x509 -sha256 -nodes -days 364 -key rootca.key -in rootca.csr -out rootca.crt
```

### LDAP Provider 인증서

**개인키 생성**

```bash
openssl genrsa -out example.com.key
```

**인증요청서 정보 작성**

```bash
cat > example.com.csr.conf << EOF
[ req ]
default_bits            = 2048
default_md              = sha256
distinguished_name      = req_distinguished_name
prompt                  = no
encrypt_key             = no

[ req_distinguished_name ]
countryName                     = "KR"
stateOrProvinceName             = "Seoul"
localityName                    = "Seoul"
0.organizationName              = "MyCompany"
organizationalUnitName          = "MyUnit"
commonName                      = "*.example.com"
EOF
```

**인증요청서 생성**

```bash
openssl req -new -key example.com.key -config example.com.csr.conf -out example.com.csr
```

**서버 인증서 생성**

```bash
openssl x509 -req -in example.com.csr -CAcreateserial -CA rootca.crt -CAkey rootca.key -out example.com.crt
```

### Replicator 인증서

**개인키 생성**

```bash
openssl genrsa -out replicator.key
```

**인증요청서 정보 작성**

```bash
cat > replicator.csr.conf << EOF
[ req ]
default_bits            = 2048
default_md              = sha256
distinguished_name      = req_distinguished_name
prompt                  = no
encrypt_key             = no

[ req_distinguished_name ]
countryName                     = "KR"
stateOrProvinceName             = "Seoul"
localityName                    = "Seoul"
0.organizationName              = "MyCompany"
organizationalUnitName          = "MyUnit"
commonName                      = "replicator"
EOF
```

**인증요청서 생성**

```bash
openssl req -new -key replicator.key -config replicator.csr.conf -out replicator.csr
```

**서버 인증서 생성**

```bash
openssl x509 -req -in replicator.csr -CAcreateserial -CA rootca.crt -CAkey rootca.key -out replicator.crt
```

### 인증서 전달

**기존 파일 삭제**

```bash
rm /etc/openldap/certs/*
```

**인증서 파일 저장**

```bash
cp ./{rootca.crt,example.com.crt,example.com.key,replicator.crt,replicator.key} /etc/openldap/certs/
```

**key 파일 권한 수정**

```bash
chmod 440 /etc/openldap/certs/{example.com.key,replicator.key}
chgrp ldap /etc/openldap/certs/{example.com.key,replicator.key}
```

---

## 클라이언트 설정

CentOS 클라이언트 설정 파일: `/etc/openldap/ldap.conf`

### 설정 방법 3가지

Self Signed Certificate를 사용한 경우, LDAP 클라이언트로 접속할 때 `ldap_start_tls: Connect error (-11)` 에러가 발생한다. 다음 중 한가지 방법으로 클라이언트 접속을 설정해야 한다.

#### `/etc/openldap/ldap.conf`: TLS_CACERT

```bash
TLS_CACERT /etc/openldap/certs/rootca.crt
```

#### `/etc/openldap/ldap.conf`: TLS_REQCERT

**TLS_REQCERT**의 기본값은 *demand*다. *allow*로 변경한다.

```bash
TLS_REQCERT allow
```

#### `~/.ldaprc`: TLS_REQCERT

사용자마다 다른 보안 설정이 가능하다.

```bash
TLS_REQCERT allow
```

---

## Provider 공통 설정

### 기존 설정 제거

패키지에 자동으로 설정된 slapd 설정을 제거한다.

```bash
rm -rf /etc/openldap/slapd.d
mkdir /etc/openldap/slapd.d
```

### 데이터베이스 디렉터리 생성

slapd가 사용할 데이터베이스 저장 경로를 생성한다.

```bash
mkdir /var/lib/ldap/data /var/lib/ldap/accesslog
chown ldap:ldap /var/lib/ldap/data /var/lib/ldap/accesslog
```

slapd 데이터베이스 저장 경로의 SELinux 보안 정보를 확인한다.

```bash
ls -ldZ /var/lib/ldap
```

### 로그 설정

slapd 로그 디렉터리를 생성한다.

```bash
mkdir /var/log/slapd
```

#### [rsyslog](https://www.rsyslog.com/category/guides-for-rsyslog/) 설정

slapd의 로그를 수집한다.

```bash
cat > /etc/rsyslog.d/slapd.conf << EOF
$template slapdtmpl,"[%$DAY%-%$MONTH%-%$YEAR% %timegenerated:12:19:date-rfc3339%] %app-name% %syslogseverity-text% %msg%\n"
local4.*    /var/log/slapd/slapd.log;slapdtmpl
EOF
```

rsyslog를 재시작한다.

```bash
systemctl restart rsyslog
```

#### [logrotate](https://linux.die.net/man/8/logrotate) 설정

rsyslog가 생성한 로그를 백업한다.

```bash
cat > /etc/logrotate.d/slapd << EOF
/var/log/slapd/slapd.log {
    compress
    create 0644 root root
    daily
    dateformat
    notifempty
    maxage 31
    missingok
    rotate 31
}
EOF
```

logrotate를 강제 실행한다.

```bash
/usr/sbin/logrotate -d /etc/logrotate.conf
```

---

## Provder 1 서버 설정

### 관리자 비밀번호 생성

Python으로 LDAP 관리자 비밀번호를 생성한다.

```bash
python -c 'import sys, crypt; print("{CRYPT}" + crypt.crypt(sys.argv[1], crypt.mksalt(crypt.METHOD_SHA256)))' "password"
```

비밀번호를 기록한다.

`{CRYPT}$5$sHDQtlifSA.yjEzN$43PybXWc5gaZCFpXvqlHSJdy8mEQHLAg.Sp3.zrEQj1`

### LDIF 작성

1. 서버 설정 LDIF: [slapd1.ldif](slapd1.ldif)
   - 관리자 비밀번호를 *olcRootPW*의 값으로 넣는다.
2. 기본 디렉터리 정보 설정 LDIF: [directories.ldif](directories.ldif)

### 설정 적용

slapd 설정을 적용한다.

```bash
slapadd -v -F /etc/openldap/slapd.d -n 0 -l slapd.ldif
```

slapd 설정 디렉터리 권한을 변경한다.

```bash
chown -R ldap:ldap /etc/openldap/slapd.d
```

### 서버 실행

slapd 서버를 생성한다.

```bash
systemctl start slapd
systemctl enable slapd
systemctl status slapd.service -l
```

### 디렉터리 정보 초기화

bind DN를 LDAP 관리자 DN으로 설정하고, StartTLS으로 요청한다.

```bash
ldapadd -x -W -D "cn=manager,ou=admins,dc=example,dc=com" -f directories.ldif -Z
# 관리자 비밀번호 입력
```

#### 디렉터리 정보 확인

```bash
ldapsearch -x -W -H ldap://ldap1.example.com -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

---

## Provider 2 서버 설정

### LDIF 작성

서버 설정 LDIF: [slapd2.ldif](slapd2.ldif)

### 설정 적용

slapd 설정을 적용한다.

```bash
slapadd -v -F /etc/openldap/slapd.d -n 0 -l slapd.ldif
```

slapd 설정 디렉터리 권한을 변경한다.

```bash
chown -R ldap:ldap /etc/openldap/slapd.d
```

### 서버 실행

slapd 서버를 생성한다.

```bash
systemctl start slapd
systemctl enable slapd
systemctl status slapd.service -l
```

### 디렉터리 정보 확인

ldap2 서버에 로컬로 접속

```bash
ldapsearch -x -W -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

ldap1 서버에 원격 접속

```bash
ldapsearch -x -W -H ldap://ldap1.example.com -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

--- 

## LDAP 디렉터리 정보 추가

Delta-syncrepl을 확인하는 과정이다.

ldap2 서버에서 다음 명령을 실행한다.

```bash
ldapadd -x -W -D "cn=manager,ou=admins,dc=example,dc=com" -Z << EOF
dn: cn=Keanu Reeves,ou=people,dc=example,dc=com
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Keanu Reeves
uid: keanu
sn: Reeves
givenName: Keanu
uidNumber: 1001
gidNumber: 500
homeDirectory: /home/users/keanu
loginShell: /bin/bash
EOF
```

ldap1 서버에서 확인한다.

```bash
ldapsearch -x -W -D "cn=manager,ou=admins,dc=example,dc=com" uid=keanu -b dc=example,dc=com -Z;
```

---

## phpLDAPadmin

### 패키지 설치

```bash
yum install -y epel-release
yum install -y phpldapadmin
```

### phpldapadmin 설정

```bash
vi /etc/phpldapadmin/config.php
```

### httpd 설정

`Require local`를 `Require all granted`로 변경한다.

```bash
vi /etc/httpd/conf.d/phpldapadmin.conf
```

### 필요없는 계정 삭제

phpLDAPadmin을 설치하면서 생성된 *caddy* 계정을 삭제한다.

```bash
cat /etc/passwd
# caddy:x:996:993:Caddy web server:/var/lib/caddy:/sbin/nologin
```

```bash
userdel -r caddy
```

### httpd 실행

```bash
systemctl start httpd
systemctl enable httpd
systemctl status httpd
```

---

## LDAP 데이터 확인 방법

관리자 DN으로 로컬 서버 TLS 접속하는 명령어.

```bash
ldapsearch -x -W -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

ldapi 프로토콜. 관리자 DN으로 로컬 서버 TLS 접속하는 명령어.

```bash
ldapsearch -x -W -H ldapi:/// -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

ldap 프로토콜. 관리자 DN으로 로컬 서버 TLS 접속하는 명령어.

```bash
ldapsearch -x -W -H ldap:/// -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

접속할 서버 지정. 관리자 DN으로 TLS 접속하는 명령어.

```bash
ldapsearch -x -W -H ldap://ldap1.example.com -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

`uid=keanu` 검색하는 명령어

```bash
ldapsearch -x -W -D "cn=manager,ou=admins,dc=example,dc=com" uid=keanu -b dc=example,dc=com -Z
```

---

## 백업

### LDAP 서버 설정 백업

```bash
slapcat -b cn=config > config.ldif
```

### 디렉터리 데이터 백업

```bash
slapcat -b dc=example,dc=com > example.com.ldif
```

### LDAP 서버 종료

```bash
systemctl stop slapd
```

### 기존 서버 설정 삭제

```bash
rm -rf /etc/openldap/slapd.d
mkdir /etc/openldap/slapd.d
```

### 기존 디렉터리 데이터 삭제

```bash
rm /var/lib/ldap/data/data.mdb /var/lib/ldap/data/lock.mdb
rm /var/lib/ldap/accesslog/data.mdb /var/lib/ldap/accesslog/lock.mdb
```

### LDAP 설정 등록

```bash
slapadd -v -F /etc/openldap/slapd.d -b cn=config -l  config.ldif
chown -R ldap:ldap /etc/openldap/slapd.d
```

### 디렉터리 데이터 등록

```bash
slapadd -b dc=example,dc=com -l example.com.ldif
chown -R ldap:ldap /var/lib/ldap
```

### LDAP 서버 실행

```bash
systemctl start slapd
```

---

## Consumer 서버 설정

