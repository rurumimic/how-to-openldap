# LDAP

[1. Introduction to OpenLDAP Directory Services](https://www.openldap.org/doc/admin24/intro.html)

## Directory Service

디렉터리는 정보 검색과 열람에 특화된 데이터베이스다.

- 디렉터리
  - 단순한 업데이트 처리
  - 필터링 기능 지원
  - 대량 조회/검색
  - 빠른 응답 시간을 위해 광범위하게 정보 복제
    - 높은 가용성, 안정성
    - 임시적인 정보 불일치
- 데이터베이스
  -  복잡한 업데이트를 처리하도록 설계
  -  복잡한 트랜잭션과 롤백 시스템 지원

## LDAP이란?

Lightweight Directory Access Protocol  
경량 디렉터리 액세스 프로토콜  

*어떤 정보를 저장할까?* LDAP 정보 모델은 *entry* 기반이다. 엔트리는 속성들이 모인 집합이다. **Distinguished Name**은 엔트리를 분명하게 구분하기 위해서 사용한다. 엔트리는 전체 디렉터리에서 유일한 DN을 하나씩 가지고 있다. 엔트리의 속성은 *type*과 하나 이상의 *value*를 가진다.

*정보는 어떻게 정리할까?* LDAP은 디렉터리 엔트리를 계층적 트리 구조로 관리한다. 다음은 직장인 정보가 담긴 디렉터리 예시다:

![LDAP directory tree](https://www.openldap.org/doc/admin24/intro_tree.png)

LDAP을 사용하면 `objectClass`를 사용해서 엔트리의 *schema* 규칙을 정할 수 있다.

*정보는 어떻게 참조될까?* 엔트리는 DN으로 참조한다. 엔트리의 **Relative Distinguished Name**과 부모 엔트리들의 RDN을 결합해서 DN을 정한다.

*정보에 어떻게 접근할까?* 검색 필터 기능을 사용해 조건과 일치하는 엔트리를 검색하고, 정보를 요청한다. 예를 들어, 이름이 `Barbara Jensen`인 사람에 대해 `dc=example,dc=com` 아래에서 하위 트리를 검색하여 발견된 각 항목의 이메일 주소를 검색 할 수 있다.

*정보를 어떻게 보호할까?* 인증이나 신원 증명을 통해 접근 제어를 한다. LDAP은 데이터 보안 (무결성 *integrity* 및 기밀성 *confidentiality*) 서비스도 지원한다.

## LDAP이 필요할 때

중앙에서 관리, 저장, 접근할 필요가 있는 데이터 저장소가 필요할 때 디렉터리 서버를 사용한다.

주로 사용하는 경우:
- Machine 인증
- 사용자 인증
- 사용자/시스템 그룹
- 주소록
- 기업/단체
- 자산 추적
- 인재 관리
- 애플리케이션 설정 저장소

다양한 경우에 알맞는 스키마를 사용하거나, 스키마를 생성할 수 있다.

## LDAP 동작 원리

LDAP은 *client-server 모델*을 사용한다. LDAP 서버는 **Directory Information Tree, DIT**를 쌓는다. 클라이언트는 어떤 LDAP 서버에 접속하든 똑같은 디렉터리를 바라본다. 서버는 클라이언트에게 바로 정보를 알려주거나, 다른 LDAP 서버에 문의하라고 지시한다.

## X.500

LDAP은 X.500(OSI directory service)와 연결하는 **Directory Access Protocol**이다. 클라이언트는 게이트웨이와 LDAP으로 통신하고, 게이트웨이는 X.500 서버와 X.500으로 통신한다. DAP는 OSI 프로토콜 스택을 모두 구현한 무거운 프로토콜이다. LDAP은 TCP/IP를 이용해 저비용으로 DAP의 대부분 기능을 지원한다.

이제는 LDAP이 X.500 서버에 포함되어 사용된다. *Standalone LDAP Daemon, slapd*는 가벼운 X.500 디렉터리 서버로 사용할 수 있다. X.500의 DAP을 전부 구현하지도, X.500 모델을 전부 지원하지도 않는다.

X.500 DAP을 사용할 계획이라면 OpenLDAP은 필요가 없다. X.500 DAP DSA로 데이터를 복제할 수는 있다. 하지만 LDAP/DAP 게이트웨이가 필요하다. OpenLDAP 소프트웨어는 게이트웨이 기능은 없다.

## [LDAP vs RDBMS](https://www.openldap.org/doc/admin24/intro.html#LDAP%20vs%20RDBMS)

LDAP은 키값 저장소 **Lightning Memory-Mapped Database, LMDB**를 사용한다. 

RDBMS를 사용하면 속도가 더 빨라지겠지만, 디렉터리 구조와는 알맞지 않다.

## slapd

*slapd*는 LDAP 디렉터리 서버다. 여러 플랫폼에서 실행가능하다. 

- LDAPv3: *slapd*는 LDAP 버전 3을 구현했다. IPv4, IPv6, Unix IPC를 지원한다.
- 간단한 인증과 보안 계층: *slapd*는 SASL을 통해서 강력한 인증과 데이터 보안 서비스를 지원한다. [Cyrus SASL](https://www.cyrusimap.org/sasl/)를 사용해서 DIGEST-MD5, EXTERNAL, GSSAPI를 포함해서 여러 메커니즘을 지원한다.
- 전송 계층 보안: *slapd*는 TLS 혹은 SSL을 통해서 인증서 기반 인증과 데이터 보안 서비스를 지원한다. [OpenSSL](https://www.openssl.org/), [GnuTLS](https://gnutls.org/), [MozNSS](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS)를 사용해서 TLS를 사용할 수 있다.
- 토폴로지 제어: *slapd*는 네트워크 토폴로지 정보를 기반으로 소켓 계층에서 접근을 제한할 수 있다. TCP 래퍼를 사용한다.
- 접근 제어: *slapd*는 풍부하고 강력한 접근 제어 기능을 제공하여 데이터베이스 정보 접근을 제어한다. LDAP 인증 정보, IP 주소, 도메인 이름 등을 기반으로 엔트리 접근을 제어한다. *slapd*는 정적 및 동적 접근 제어를 지원한다.
- 국제화: 유니코드와 언어 태그를 지원한다.
- 데이터베이스 백엔드 선택: *slapd*는 다양한 데이터베이스 백엔드를 지원한다. MDB는 계층적인 고성능 트랜잭션 데이터베이스다. BDB, HDB, SHELL, PASSWD 등이 사용 가능하다. MDB는 LMDB를 사용한다.
- 여러 데이터베이스 인스턴스: *slapd*는 여러 데이터베이스를 동시에 사용할 수 있다. 여러 데이터베이스 백엔드를 사용해서 LDAP 트리의 다양한 부분에 대한 응답을 처리할 수 있다.
- 제네릭 모듈 API: 더 많은 기능이 필요한 경우, 모듈을 추가할 수 있다. *slapd*는 두 가지 부분으로 구성된다. LDAP 클라이언트와 프로토콜 통신을 처리하는 프론트엔드, 데이터베이스 작업과 같은 특정 작업을 처리하는 모듈. C로 작성된 API로 통신을 하기 때문에 사용자가 직접 정의한 모듈을 추가할 수 있다. 수정이 가능한 데이터베이스 모듈도 제공된다. 다양한 언어로 *slapd*이 외부 데이터 자원을 사용할 수 있다.
- 스레드: *slapd*는 스레드를 사용해서 고성능을 낸다. 싱글 멀티스레드 *slapd* 프로세스는 모든 요청을 스레드 풀로 관리한다. 이로써 시스템 오버헤드를 줄인다.
- 복제: *slapd*는 디렉터리 정보를 shadow 복사본으로 관리할 수 있다. 단일 *slapd*에서는 가용성과 안정성이 지원되지 않는다. 따라서 대용량 환경을 구축하기 위해 *single-master/multiple-slave* 복제 체계가 필요하다. 단일 장애 지점을 예방하기 위해서 멀티 마스터 복제도 가능하다. LDAP 동기화 기반 복제도 지원한다.
- 프록시 캐시: *slapd*는 캐싱 LDAP 프록시 서비스를 설정할 수 있다.
- 설정: *slapd*는 설정 파일 하나로 모든 설정이 가능하다. 설정 기본값이 있어 편리하게 작업할 수 있다. 동적으로 설정을 관리할 수 있다.