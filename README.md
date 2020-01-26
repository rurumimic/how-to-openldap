# How to OpenLDAP

OpenLDAP 서버를 구축하는 방법

---

## Architectures

### Design 1

- MirrorMode
- Delta-syncrepl
- TLS

[OpenLDAP 서버 이중화 구축 및 TLS 보안 통신 방법](design.md)

### Design 2

- MirrorMode
- Delta-syncrepl
- SASL GSSAPI
- Kerberos V

Kerberos를 사용하는 아키텍처는 다음 GitHub를 확인한다.

[OpenLDAP과 Kerberos V 연동](https://github.com/rurumimic/how-to-openldap-with-kerberos-v)

---

## Cases: 경우의 수

OpenLDAP을 활용하여 구축하는 방법은 여러 가지가 있다.

### Provider & Consumer

1. Single Master: 기본 설정
2. Multiple Master
   - 장점: single-master의 단일 장애점 해결. 고가용성.
   - 단점: 로드밸런싱 기능 없음. single-master 보다 성능이 좋지 않음.
3. **MirrorMode**
   - 장점: 고가용성. Hot-Standby 혹은 Active-Active. 
   - 단점: 로드밸런싱 기능 필요. 
4. Syncrepl Proxy Mode: Consumer가 직접 Provider에 접근할 수 없을 때 사용

### Replication

1. Syncrepl: 객체 기반 복제
   - refreshOnly: 폴링
   - refreshAndPersist: 리스닝
2. **Delta-syncrepl**: 변경로그 기반 복제

### Security

#### TLS

X.509 인증서를 이용해 LDAP 서버와 보안 통신한다.

SASL EXTERNAL 매커니즘을 사용한다.

#### SASL

RFC 4422: [Simple Authentication and Security Layer](https://tools.ietf.org/html/rfc4422)

SASL GSSAPI 매커니즘으로 Kerberos V를 사용한다.

TLS 통신이나 `ldapi:///` 유닉스 도메인 소켓 통신을 할 때는 EXTERNAL 매커니즘을 사용한다.

나머지 매커니즘은 고려하지 않는다.

---

## Links

- [OpenLDAP](https://www.openldap.org/): 공식 홈페이지
- [Documentation](https://www.openldap.org/doc/): 문서
  - [2.4 Administrator's Guide](https://www.openldap.org/doc/admin24/): 설치, 환경설정, 사용법 등
- [Manual Pages](https://www.openldap.org/software/man.cgi): 설정 파일 옵션, 명령어 옵션 등
- [Faq-O-Matic](http://www.openldap.org/faq/data/cache/1.html): 자주 묻는 질문
- [Mirror Repository](https://github.com/openldap/openldap): GitHub 레포지터리

---

## 2.4 Administrator's Guide 해석

1. [LDAP](guide/ldap.md)
2. [Service Design](guide/service-design.md)
3. [Installation](guide/installation.md)
4. [Configuration](guide/configuration.md)
5. [Access Control](guide/access-control.md)
6. [Limits](guide/limits.md)
7. [Backends](guide/overlays.md)
8. [Overlays](guide/overlays.md)
9. [Schema](guide/schema.md)
10. [Security](guide/security.md)
11. [SASL](guide/sasl.md)
12. [TLS](guide/tls.md)
13. [Replication](guide/replication.md)
14. [Maintenance](guide/maintenance.md)
15. [Monitoring](guide/monitoring.md)
16. [Tuning](guide/tuning.md)
17. [Troubleshooting](guide/troubleshooting.md)
