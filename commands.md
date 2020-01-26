# slapd 명령어

- [slapd 명령어](#slapd-%eb%aa%85%eb%a0%b9%ec%96%b4)
  - [LDAP 데이터 확인 방법](#ldap-%eb%8d%b0%ec%9d%b4%ed%84%b0-%ed%99%95%ec%9d%b8-%eb%b0%a9%eb%b2%95)
    - [관리자 DN으로 로컬 서버 TLS 접속하는 명령어](#%ea%b4%80%eb%a6%ac%ec%9e%90-dn%ec%9c%bc%eb%a1%9c-%eb%a1%9c%ec%bb%ac-%ec%84%9c%eb%b2%84-tls-%ec%a0%91%ec%86%8d%ed%95%98%eb%8a%94-%eb%aa%85%eb%a0%b9%ec%96%b4)
    - [ldapi 프로토콜과 관리자 DN으로 로컬 서버 TLS 접속하는 명령어.](#ldapi-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c%ea%b3%bc-%ea%b4%80%eb%a6%ac%ec%9e%90-dn%ec%9c%bc%eb%a1%9c-%eb%a1%9c%ec%bb%ac-%ec%84%9c%eb%b2%84-tls-%ec%a0%91%ec%86%8d%ed%95%98%eb%8a%94-%eb%aa%85%eb%a0%b9%ec%96%b4)
    - [ldap 프로토콜과 관리자 DN으로 로컬 서버 TLS 접속하는 명령어.](#ldap-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c%ea%b3%bc-%ea%b4%80%eb%a6%ac%ec%9e%90-dn%ec%9c%bc%eb%a1%9c-%eb%a1%9c%ec%bb%ac-%ec%84%9c%eb%b2%84-tls-%ec%a0%91%ec%86%8d%ed%95%98%eb%8a%94-%eb%aa%85%eb%a0%b9%ec%96%b4)
    - [접속할 서버 지정 후 관리자 DN으로 TLS 접속하는 명령어.](#%ec%a0%91%ec%86%8d%ed%95%a0-%ec%84%9c%eb%b2%84-%ec%a7%80%ec%a0%95-%ed%9b%84-%ea%b4%80%eb%a6%ac%ec%9e%90-dn%ec%9c%bc%eb%a1%9c-tls-%ec%a0%91%ec%86%8d%ed%95%98%eb%8a%94-%eb%aa%85%eb%a0%b9%ec%96%b4)
    - [`uid=keanu`을 검색하는 명령어](#uidkeanu%ec%9d%84-%ea%b2%80%ec%83%89%ed%95%98%eb%8a%94-%eb%aa%85%eb%a0%b9%ec%96%b4)

## LDAP 데이터 확인 방법

### 관리자 DN으로 로컬 서버 TLS 접속하는 명령어

```bash
ldapsearch -x -W -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

### ldapi 프로토콜과 관리자 DN으로 로컬 서버 TLS 접속하는 명령어.

```bash
ldapsearch -x -W -H ldapi:/// -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

### ldap 프로토콜과 관리자 DN으로 로컬 서버 TLS 접속하는 명령어.

```bash
ldapsearch -x -W -H ldap:/// -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

### 접속할 서버 지정 후 관리자 DN으로 TLS 접속하는 명령어.

```bash
ldapsearch -x -W -H ldap://ldap1.example.com -D "cn=manager,ou=admins,dc=example,dc=com" objectClass=* -b dc=example,dc=com -Z
```

### `uid=keanu`을 검색하는 명령어

```bash
ldapsearch -x -W -D "cn=manager,ou=admins,dc=example,dc=com" uid=keanu -b dc=example,dc=com -Z
```