# Provider 2 서버 설정

- [Provider 2 서버 설정](#provider-2-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)
  - [LDIF 작성](#ldif-%ec%9e%91%ec%84%b1)
  - [설정 적용](#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
  - [서버 실행](#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)
  - [디렉터리 정보 확인](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)

---

## LDIF 작성

서버 설정 LDIF: [slapd2.ldif](src/slapd2.ldif)

---

## 설정 적용

slapd 설정을 적용한다. 100.00%이라고 결과가 나타나면 설정이 잘 된 것이다.

```bash
slapadd -v -F /etc/openldap/slapd.d -n 0 -l slapd2.ldif
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

ldap2 서버에 로컬로 접속

```bash
ldapsearch -x -W -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

ldap1 서버에 원격 접속

```bash
ldapsearch -x -W -H ldap://ldap1.example.com -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```