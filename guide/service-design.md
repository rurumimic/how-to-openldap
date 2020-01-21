# Service Design

[3. The Big Picture - Configuration Choices](https://www.openldap.org/doc/admin24/config.html)

## Local Directory Service

![](https://www.openldap.org/doc/admin24/config_local.png)

가장 간단한 설정이다. 로컬에서 사용하기 편한 설계다.

## Local Directory Service with Referrals

![](https://www.openldap.org/doc/admin24/config_ref.png)

클라이언트에게 받은 요청을 다른 서버에게 위임한다. 로컬 서비스와 글로벌 디렉터리를 제공할 수 있다.

## Replicated Directory Service

![](https://www.openldap.org/doc/admin24/config_repl.png)

*syncrepl*은 동기화 기반 복제 기능이다. 여러 디렉터리 서버에서 디렉터리 정보는 섀도우 복제본으로 저장된다. 기본 설정에서 *master*는 syncrepl 공급자이고 *slave/shadow*는 syncrepl 소비자다.

## Distributed Local Directory Service

- [Constructing a Distributed Directory Service](https://www.openldap.org/doc/admin24/referrals.html)

로컬 디렉터리 서비스들에게 정보를 복제해 저장할 수 있다.