# 백업

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

---

## LDAP 서버 설정 백업

`olcServerID`가 다른 백업 파일을 사용하면 slapd는 실행되지 않는다.  

```bash
slapcat -n 0 -l config.ldif
```

## Accesslog 데이터 백업

```bash
slapcat -n 2 -l accesslog.ldif
```

## 디렉터리 데이터 백업

```bash
slapcat -n 3 -l example.com.ldif
```

## LDAP 서버 종료

```bash
systemctl stop slapd
```

## 기존 서버 설정 삭제

```bash
rm -rf /etc/openldap/slapd.d
mkdir /etc/openldap/slapd.d
```

## 기존 디렉터리 데이터 삭제

```bash
rm -f /var/lib/ldap/accesslog/{data.mdb,lock.mdb} /var/lib/ldap/data/{data.mdb,lock.mdb}
```

## LDAP 설정 등록

`olcServerID`가 다른 백업 파일을 사용하면 slapd는 실행되지 않는다.  

```bash
slapadd -n 0 -F /etc/openldap/slapd.d -l config.ldif
chown -R ldap:ldap /etc/openldap/slapd.d
```

## Accesslog 데이터 등록

옵션 `-w`: write syncrepl context information. After all entries are added, the contextCSN will be updated with the greatest CSN in the database.

```bash
slapadd -n 2 -F /etc/openldap/slapd.d -l accesslog.ldif
chown -R ldap:ldap /var/lib/ldap
```

## 디렉터리 데이터 등록

옵션 `-w`: write syncrepl context information. After all entries are added, the contextCSN will be updated with the greatest CSN in the database.

```bash
slapadd -n 3 -F /etc/openldap/slapd.d -l example.com.ldif
chown -R ldap:ldap /var/lib/ldap
```

## LDAP 서버 실행

```bash
systemctl start slapd
```