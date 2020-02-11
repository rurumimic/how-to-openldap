# Provider 1 서버 설정

- [Provider 1 서버 설정](#provider-1-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)
  - [관리자 비밀번호 생성](#%ea%b4%80%eb%a6%ac%ec%9e%90-%eb%b9%84%eb%b0%80%eb%b2%88%ed%98%b8-%ec%83%9d%ec%84%b1)
  - [LDIF 작성](#ldif-%ec%9e%91%ec%84%b1)
  - [설정 적용](#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
  - [서버 실행](#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)
  - [디렉터리 정보 초기화](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ec%b4%88%ea%b8%b0%ed%99%94)
    - [디렉터리 정보 확인](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)
  - [포트 변경](#포트-변경)

---

## 관리자 비밀번호 생성

Python으로 LDAP 관리자 비밀번호를 생성한다.

```bash
python -c 'import sys, crypt; print("{CRYPT}" + crypt.crypt(sys.argv[1], crypt.mksalt(crypt.METHOD_SHA256)))' "password"
```

비밀번호를 기록한다.

`{CRYPT}$5$MTyIW7Nq/1hpAbav$dzp/GM6XreRoU0m7t4iphosNt7ltyOj6Uktg3W4DbbC`

---

## LDIF 작성

1. 서버 설정 LDIF: [slapd1.ldif](/src/slapd1.ldif)
   - 관리자 비밀번호를 *olcRootPW*의 값으로 넣는다.
2. 기본 디렉터리 정보 설정 LDIF: [directories.ldif](/src/directories.ldif)

---

## 설정 적용

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

---

## 서버 실행

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

---

## 디렉터리 정보 초기화

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

---

### 디렉터리 정보 확인

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

## 포트 변경

```bash
vi /etc/sysconfig/slapd
```

```bash
SLAPD_URLS="ldapi:/// ldap://<HOST_NAME>:9898"
```

변경 내용을 SLAPD.LDIF 파일에도 반영한다

