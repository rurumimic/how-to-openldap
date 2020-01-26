# Consumer 서버 설정

- [Consumer 서버 설정](#consumer-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)
  - [LDIF 작성](#ldif-%ec%9e%91%ec%84%b1)
  - [설정 적용](#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
  - [서버 실행](#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)
  - [디렉터리 정보 확인](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)

---

## LDIF 작성

서버 설정 LDIF: [consumer.ldif](src/consumer.ldif)

---

## 설정 적용

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

---

## 서버 실행

slapd 서버를 생성한다.

```bash
systemctl start slapd
systemctl enable slapd
systemctl status slapd.service -l
```

---

## 디렉터리 정보 확인

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
