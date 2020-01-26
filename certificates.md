# 인증서

- [인증서](#%ec%9d%b8%ec%a6%9d%ec%84%9c)
  - [RootCA 인증서](#rootca-%ec%9d%b8%ec%a6%9d%ec%84%9c)
  - [LDAP Provider 인증서](#ldap-provider-%ec%9d%b8%ec%a6%9d%ec%84%9c)
  - [Replicator 인증서](#replicator-%ec%9d%b8%ec%a6%9d%ec%84%9c)
  - [결과 파일](#%ea%b2%b0%ea%b3%bc-%ed%8c%8c%ec%9d%bc)
  - [인증서 전달](#%ec%9d%b8%ec%a6%9d%ec%84%9c-%ec%a0%84%eb%8b%ac)

---

## RootCA 인증서

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

---

## LDAP Provider 인증서

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

---

## Replicator 인증서

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

---

## 결과 파일

- `rootca.crt`, `rootca.csr`, `rootca.csr.conf`, `rootca.key`, `rootca.srl`
- `example.com.crt`, `example.com.csr`, `example.com.csr.conf`, `example.com.key`
- `replicator.crt`, `replicator.csr`, `replicator.csr.conf`, `replicator.key`

---

## 인증서 전달

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