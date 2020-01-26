# 클라이언트 설정 방법: 3가지

CentOS 클라이언트 설정 파일: `/etc/openldap/ldap.conf`

Self Signed Certificate를 사용한 경우, LDAP 클라이언트로 접속할 때 `ldap_start_tls: Connect error (-11)` 에러가 발생한다. 다음 중 한가지 방법으로 클라이언트 접속을 설정해야 한다.

**`/etc/openldap/ldap.conf`: TLS_CACERT**

```bash
TLS_CACERT /etc/openldap/certs/rootca.crt
```

**`/etc/openldap/ldap.conf`: TLS_REQCERT**

**TLS_REQCERT**의 기본값은 *demand*다. *allow*로 변경한다.

```bash
TLS_REQCERT allow
```

**`~/.ldaprc`: TLS_REQCERT**

사용자마다 다른 보안 설정이 가능하다.

```bash
TLS_REQCERT allow
```