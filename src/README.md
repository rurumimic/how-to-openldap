# LDAP Service Architecture Design

- [LDAP Service Architecture Design](#ldap-service-architecture-design)
  - [Servers](#servers)
  - [Vagrant](#vagrant)
  - [공통 설정](#%ea%b3%b5%ed%86%b5-%ec%84%a4%ec%a0%95)
    - [Hosts 설정](#hosts-%ec%84%a4%ec%a0%95)
    - [패키지 설치](#%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
    - [기존 설정 제거](#%ea%b8%b0%ec%a1%b4-%ec%84%a4%ec%a0%95-%ec%a0%9c%ea%b1%b0)
    - [데이터베이스 디렉터리 생성](#%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4-%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%83%9d%ec%84%b1)
    - [로그 설정](#%eb%a1%9c%ea%b7%b8-%ec%84%a4%ec%a0%95)
      - [rsyslog 설정](#rsyslog-%ec%84%a4%ec%a0%95)
      - [logrotate 설정](#logrotate-%ec%84%a4%ec%a0%95)
  - [인증서](#%ec%9d%b8%ec%a6%9d%ec%84%9c)
    - [RootCA 인증서](#rootca-%ec%9d%b8%ec%a6%9d%ec%84%9c)
    - [LDAP Provider 인증서](#ldap-provider-%ec%9d%b8%ec%a6%9d%ec%84%9c)
    - [Replicator 인증서](#replicator-%ec%9d%b8%ec%a6%9d%ec%84%9c)
    - [결과 파일](#%ea%b2%b0%ea%b3%bc-%ed%8c%8c%ec%9d%bc)
    - [인증서 전달](#%ec%9d%b8%ec%a6%9d%ec%84%9c-%ec%a0%84%eb%8b%ac)
  - [클라이언트 설정](#%ed%81%b4%eb%9d%bc%ec%9d%b4%ec%96%b8%ed%8a%b8-%ec%84%a4%ec%a0%95)
    - [설정 방법 3가지](#%ec%84%a4%ec%a0%95-%eb%b0%a9%eb%b2%95-3%ea%b0%80%ec%a7%80)
      - [`/etc/openldap/ldap.conf`: TLS_CACERT](#etcopenldapldapconf-tlscacert)
      - [`/etc/openldap/ldap.conf`: TLS_REQCERT](#etcopenldapldapconf-tlsreqcert)
      - [`~/.ldaprc`: TLS_REQCERT](#ldaprc-tlsreqcert)
  - [Provder 1 서버 설정](#provder-1-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)
    - [관리자 비밀번호 생성](#%ea%b4%80%eb%a6%ac%ec%9e%90-%eb%b9%84%eb%b0%80%eb%b2%88%ed%98%b8-%ec%83%9d%ec%84%b1)
    - [LDIF 작성](#ldif-%ec%9e%91%ec%84%b1)
    - [설정 적용](#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
    - [서버 실행](#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)
    - [디렉터리 정보 초기화](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ec%b4%88%ea%b8%b0%ed%99%94)
      - [디렉터리 정보 확인](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)
  - [Provider 2 서버 설정](#provider-2-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)
    - [LDIF 작성](#ldif-%ec%9e%91%ec%84%b1-1)
    - [설정 적용](#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9-1)
    - [서버 실행](#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89-1)
    - [디렉터리 정보 확인](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8-1)
  - [LDAP 디렉터리 정보 추가](#ldap-%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ec%b6%94%ea%b0%80)
  - [phpLDAPadmin](#phpldapadmin)
    - [패키지 설치](#%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98-1)
    - [phpldapadmin 설정](#phpldapadmin-%ec%84%a4%ec%a0%95)
    - [httpd 설정](#httpd-%ec%84%a4%ec%a0%95)
    - [필요없는 계정 삭제](#%ed%95%84%ec%9a%94%ec%97%86%eb%8a%94-%ea%b3%84%ec%a0%95-%ec%82%ad%ec%a0%9c)
    - [httpd 실행](#httpd-%ec%8b%a4%ed%96%89)
    - [phpLDAPadmin 접속](#phpldapadmin-%ec%a0%91%ec%86%8d)
  - [LDAP 데이터 확인 방법](#ldap-%eb%8d%b0%ec%9d%b4%ed%84%b0-%ed%99%95%ec%9d%b8-%eb%b0%a9%eb%b2%95)
  - [백업](#%eb%b0%b1%ec%97%85)
    - [LDAP 서버 설정 백업](#ldap-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95-%eb%b0%b1%ec%97%85)
    - [Accesslog 데이터 백업](#accesslog-%eb%8d%b0%ec%9d%b4%ed%84%b0-%eb%b0%b1%ec%97%85)
    - [디렉터리 데이터 백업](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%eb%8d%b0%ec%9d%b4%ed%84%b0-%eb%b0%b1%ec%97%85)
    - [LDAP 서버 종료](#ldap-%ec%84%9c%eb%b2%84-%ec%a2%85%eb%a3%8c)
    - [기존 서버 설정 삭제](#%ea%b8%b0%ec%a1%b4-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95-%ec%82%ad%ec%a0%9c)
    - [기존 디렉터리 데이터 삭제](#%ea%b8%b0%ec%a1%b4-%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%eb%8d%b0%ec%9d%b4%ed%84%b0-%ec%82%ad%ec%a0%9c)
    - [LDAP 설정 등록](#ldap-%ec%84%a4%ec%a0%95-%eb%93%b1%eb%a1%9d)
    - [Accesslog 데이터 등록](#accesslog-%eb%8d%b0%ec%9d%b4%ed%84%b0-%eb%93%b1%eb%a1%9d)
    - [디렉터리 데이터 등록](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%eb%8d%b0%ec%9d%b4%ed%84%b0-%eb%93%b1%eb%a1%9d)
    - [LDAP 서버 실행](#ldap-%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)
  - [Consumer 서버 설정](#consumer-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)
    - [LDIF 작성](#ldif-%ec%9e%91%ec%84%b1-2)
    - [설정 적용](#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9-2)
    - [서버 실행](#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89-2)
    - [디렉터리 정보 확인](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8-2)

---

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

예를 들어, ldap1 서버에서 `/etc/hosts`를 고친다.

```bash
127.0.0.1        ldap1.example.com        ldap1
192.168.9.101    ldap1.example.com        ldap1
192.168.9.102    ldap2.example.com        ldap2
192.168.9.103    consumer.example.com     consumer
```

호스트 네임을 확인한다.

```bash
# 호스트 네임 확인 명령
hostname # ldap1
hostname -f # ldap1.example.com
# 호스트 네임 변경 명령
hostnamectl set-hostname ldap1
```

나머지 서버들도 `/etc/hosts`를 설정한다.

### 패키지 설치

OpenLDAP 라이브러리, 서버, 클라이언트를 설치한다.

```bash
yum install -y openldap openldap-servers openldap-clients
```

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
# drwx------. ldap ldap system_u:object_r:slapd_db_t:s0  /var/lib/ldap
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
/usr/sbin/logrotate -f /etc/logrotate.conf
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

### 결과 파일

- `rootca.crt`, `rootca.csr`, `rootca.csr.conf`, `rootca.key`, `rootca.srl`
- `example.com.crt`, `example.com.csr`, `example.com.csr.conf`, `example.com.key`
- `replicator.crt`, `replicator.csr`, `replicator.csr.conf`, `replicator.key`

### 인증서 전달

LDAP 서버마다 `/etc/openldap/certs/` 경로에 `rootca.crt`, `example.com.crt`, `example.com.key`, `replicator.crt`, `replicator.key`를 복제한다.

**기존 파일 삭제**

```bash
rm -f /etc/openldap/certs/*
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

## Provder 1 서버 설정

### 관리자 비밀번호 생성

Python으로 LDAP 관리자 비밀번호를 생성한다.

```bash
python -c 'import sys, crypt; print("{CRYPT}" + crypt.crypt(sys.argv[1], crypt.mksalt(crypt.METHOD_SHA256)))' "password"
```

비밀번호를 기록한다.

`{CRYPT}$5$MTyIW7Nq/1hpAbav$dzp/GM6XreRoU0m7t4iphosNt7ltyOj6Uktg3W4DbbC`

### LDIF 작성

1. 서버 설정 LDIF: [slapd1.ldif](slapd1.ldif)
   - 관리자 비밀번호를 *olcRootPW*의 값으로 넣는다.
2. 기본 디렉터리 정보 설정 LDIF: [directories.ldif](directories.ldif)

### 설정 적용

slapd 설정을 적용한다.

```bash
slapadd -v -F /etc/openldap/slapd.d -n 0 -l slapd1.ldif
```

다음처럼 100.00%이라고 결과가 나타나면 설정이 잘 된 것이다.

```bash
added: "cn=config" (00000001)
added: "cn=module{0},cn=config" (00000001)
added: "cn=schema,cn=config" (00000001)
added: "cn={0}core,cn=schema,cn=config" (00000001)
added: "cn={1}cosine,cn=schema,cn=config" (00000001)
added: "cn={2}nis,cn=schema,cn=config" (00000001)
added: "cn={3}inetorgperson,cn=schema,cn=config" (00000001)
added: "cn={4}openldap,cn=schema,cn=config" (00000001)
added: "olcDatabase={-1}frontend,cn=config" (00000001)
added: "olcDatabase={0}config,cn=config" (00000001)
added: "olcDatabase={1}monitor,cn=config" (00000001)
added: "olcDatabase={2}mdb,cn=config" (00000001)
added: "olcOverlay={0}syncprov,olcDatabase={2}mdb,cn=config" (00000001)
added: "olcDatabase={3}mdb,cn=config" (00000001)
added: "olcOverlay={0}syncprov,olcDatabase={3}mdb,cn=config" (00000001)
added: "olcOverlay={1}accesslog,olcDatabase={3}mdb,cn=config" (00000001)
added: "olcOverlay={2}memberof,olcDatabase={3}mdb,cn=config" (00000001)
added: "olcOverlay={3}refint,olcDatabase={3}mdb,cn=config" (00000001)
_#################### 100.00% eta   none elapsed            none fast!         
Closing DB...
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
```

slapd 상태를 확인한다.

```bash
systemctl status slapd.service -l
```

```bash
● slapd.service - OpenLDAP Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/slapd.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2020-01-23 13:54:36 UTC; 8s ago
     Docs: man:slapd
           man:slapd-config
           man:slapd-hdb
           man:slapd-mdb
           file:///usr/share/doc/openldap-servers/guide.html
  Process: 4226 ExecStart=/usr/sbin/slapd -u ldap -h ${SLAPD_URLS} $SLAPD_OPTIONS (code=exited, status=0/SUCCESS)
  Process: 4213 ExecStartPre=/usr/libexec/openldap/check-config.sh (code=exited, status=0/SUCCESS)
 Main PID: 4228 (slapd)
   CGroup: /system.slice/slapd.service
           └─4228 /usr/sbin/slapd -u ldap -h ldapi:/// ldap:///

Jan 23 13:54:36 ldap1 systemd[1]: Starting OpenLDAP Server Daemon...
Jan 23 13:54:36 ldap1 runuser[4216]: pam_unix(runuser:session): session opened for user ldap by (uid=0)
Jan 23 13:54:36 ldap1 slapd[4226]: @(#) $OpenLDAP: slapd 2.4.44 (Jan 29 2019 17:42:45) $
                                           mockbuild@x86-01.bsys.centos.org:/builddir/build/BUILD/openldap-2.4.44/openldap-2.4.44/servers/slapd
Jan 23 13:54:36 ldap1 slapd[4228]: slapd starting
Jan 23 13:54:36 ldap1 systemd[1]: Started OpenLDAP Server Daemon.
Jan 23 13:54:39 ldap1 slapd[4228]: slap_client_connect: URI=ldap://ldap2.example.com:389 Warning, ldap_start_tls failed (-1)
Jan 23 13:54:42 ldap1 slapd[4228]: slap_client_connect: URI=ldap://ldap2.example.com:389 ldap_sasl_interactive_bind_s failed (-1)
Jan 23 13:54:42 ldap1 slapd[4228]: do_syncrepl: rid=001 rc -1 retrying (4 retries left)
```

`slap_client_connect: URI=ldap://ldap2.example.com:389 Warning, ldap_start_tls failed (-1)`:  
아직은 ldap2 서버에 연결을 하지 못한다.

### 디렉터리 정보 초기화

bind DN를 LDAP 관리자 DN으로 설정하고, StartTLS으로 요청한다.

```bash
ldapadd -x -W -D "cn=manager,ou=admins,dc=example,dc=com" -f directories.ldif -Z
```

```bash
Enter LDAP Password: # 관리자 비밀번호 입력
adding new entry "dc=example,dc=com"
adding new entry "ou=admins,dc=example,dc=com"
adding new entry "cn=manager,ou=admins,dc=example,dc=com"
adding new entry "cn=replicator,ou=admins,dc=example,dc=com"
adding new entry "ou=people,dc=example,dc=com"
adding new entry "ou=group,dc=example,dc=com"
```

#### 디렉터리 정보 확인

```bash
ldapsearch -x -W -H ldap://ldap1.example.com -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

```bash
# example.com
dn: dc=example,dc=com
dc: example
objectClass: top
objectClass: domain

# admins, example.com
dn: ou=admins,dc=example,dc=com
objectClass: organizationalUnit
ou: admins

# manager, admins, example.com
dn: cn=manager,ou=admins,dc=example,dc=com
objectClass: organizationalRole
cn: manager
description: LDAP Manager

# replicator, admins, example.com
dn: cn=replicator,ou=admins,dc=example,dc=com
objectClass: organizationalRole
cn: replicator
description: Replicator

# people, example.com
dn: ou=people,dc=example,dc=com
objectClass: organizationalUnit
ou: people

# group, example.com
dn: ou=group,dc=example,dc=com
objectClass: organizationalUnit
ou: group
```

---

## Provider 2 서버 설정

### LDIF 작성

서버 설정 LDIF: [slapd2.ldif](slapd2.ldif)

### 설정 적용

slapd 설정을 적용한다. 100.00%이라고 결과가 나타나면 설정이 잘 된 것이다.

```bash
slapadd -v -F /etc/openldap/slapd.d -n 0 -l slapd2.ldif
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

```bash
Enter LDAP Password: # 관리자 비밀번호 입력
adding new entry "cn=Keanu Reeves,ou=people,dc=example,dc=com"
```

ldap1 서버에서 확인한다.

```bash
ldapsearch -x -W -D "cn=manager,ou=admins,dc=example,dc=com" uid=keanu -b dc=example,dc=com -Z
```

```bash
# Keanu Reeves, people, example.com
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
```

Provider 2개로 MirrorMode 설정을 완료했다.

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

VI 줄 이동 단축키: `ESC` + `:숫자`

- 298: `$servers->setValue('server','host','ldap://ldap1.example.com')`
- 340: `$servers->setValue('server','tls',true)`
- 397: `$servers->setValue('login','attr','dn')`

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

### phpLDAPadmin 접속

브라우저로 [192.168.9.101/phpldapadmin](http://192.168.9.101/phpldapadmin)에 접속한다.

- User Name: `cn=manager,ou=admins,dc=example,dc=com`
- Password: 관리자 비밀번호

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

`olcServerID`가 다른 백업 파일을 사용하면 slapd는 실행되지 않는다.  

```bash
slapcat -n 0 -l config.ldif
```

### Accesslog 데이터 백업

```bash
slapcat -n 2 -l accesslog.ldif
```

### 디렉터리 데이터 백업

```bash
slapcat -n 3 -l example.com.ldif
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
rm -f /var/lib/ldap/accesslog/{data.mdb,lock.mdb} /var/lib/ldap/data/{data.mdb,lock.mdb}
```

### LDAP 설정 등록

`olcServerID`가 다른 백업 파일을 사용하면 slapd는 실행되지 않는다.  

```bash
slapadd -n 0 -F /etc/openldap/slapd.d -l config.ldif
chown -R ldap:ldap /etc/openldap/slapd.d
```

### Accesslog 데이터 등록

옵션 `-w`: write syncrepl context information. After all entries are added, the contextCSN will be updated with the greatest CSN in the database.

```bash
slapadd -n 2 -F /etc/openldap/slapd.d -l accesslog.ldif
chown -R ldap:ldap /var/lib/ldap
```

### 디렉터리 데이터 등록

옵션 `-w`: write syncrepl context information. After all entries are added, the contextCSN will be updated with the greatest CSN in the database.

```bash
slapadd -n 3 -F /etc/openldap/slapd.d -l example.com.ldif
chown -R ldap:ldap /var/lib/ldap
```

### LDAP 서버 실행

```bash
systemctl start slapd
```

---

## Consumer 서버 설정

### LDIF 작성

서버 설정 LDIF: [consumer.ldif](consumer.ldif)

### 설정 적용

slapd 설정을 적용한다.

```bash
slapadd -v -F /etc/openldap/slapd.d -n 0 -l consumer.ldif
```

100.00%이라고 결과가 나타나면 설정이 잘 된 것이다.

```bash
added: "cn=config" (00000001)
added: "cn=module{0},cn=config" (00000001)
added: "cn=schema,cn=config" (00000001)
added: "cn={0}core,cn=schema,cn=config" (00000001)
added: "cn={1}cosine,cn=schema,cn=config" (00000001)
added: "cn={2}nis,cn=schema,cn=config" (00000001)
added: "cn={3}inetorgperson,cn=schema,cn=config" (00000001)
added: "cn={4}openldap,cn=schema,cn=config" (00000001)
added: "olcDatabase={-1}frontend,cn=config" (00000001)
added: "olcDatabase={0}config,cn=config" (00000001)
added: "olcDatabase={1}monitor,cn=config" (00000001)
added: "olcDatabase={2}mdb,cn=config" (00000001)
_#################### 100.00% eta   none elapsed            none fast!         
Closing DB...
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

consumer 서버에 로컬로 접속

```bash
ldapsearch -x -W -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

Consumer 설정에 따라 제한된 필요한 정보만 볼 수도 있다.

```bash
# example.com
dn: dc=example,dc=com
objectClass: top
objectClass: domain

# admins, example.com
dn: ou=admins,dc=example,dc=com
objectClass: organizationalUnit

# manager, admins, example.com
dn: cn=manager,ou=admins,dc=example,dc=com
objectClass: organizationalRole
cn: manager

# replicator, admins, example.com
dn: cn=replicator,ou=admins,dc=example,dc=com
objectClass: organizationalRole
cn: replicator

# people, example.com
dn: ou=people,dc=example,dc=com
objectClass: organizationalUnit

# group, example.com
dn: ou=group,dc=example,dc=com
objectClass: organizationalUnit

# Keanu Reeves, people, example.com
dn: cn=Keanu Reeves,ou=people,dc=example,dc=com
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Keanu Reeves
givenName: Keanu
```
