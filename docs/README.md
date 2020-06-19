# OpenLDAP 서버 이중화 구축 및 TLS 보안 통신 방법

- [OpenLDAP 서버 이중화 구축 및 TLS 보안 통신 방법](#openldap-%ec%84%9c%eb%b2%84-%ec%9d%b4%ec%a4%91%ed%99%94-%ea%b5%ac%ec%b6%95-%eb%b0%8f-tls-%eb%b3%b4%ec%95%88-%ed%86%b5%ec%8b%a0-%eb%b0%a9%eb%b2%95)
  - [구성요소](#%ea%b5%ac%ec%84%b1%ec%9a%94%ec%86%8c)
  - [Servers](#servers)
  - [Vagrant](#vagrant)
  - [공통 설정](#%ea%b3%b5%ed%86%b5-%ec%84%a4%ec%a0%95)
  - [인증서](#%ec%9d%b8%ec%a6%9d%ec%84%9c)
  - [클라이언트 설정](#%ed%81%b4%eb%9d%bc%ec%9d%b4%ec%96%b8%ed%8a%b8-%ec%84%a4%ec%a0%95)
  - [Provider 1 서버 설정](#provider-1-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)
  - [Provider 2 서버 설정](#provider-2-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)
  - [Delta-Syncrepl 테스트](#delta-syncrepl-%ed%85%8c%ec%8a%a4%ed%8a%b8)
  - [phpLDAPadmin](#phpldapadmin)
  - [LDAP 데이터 확인 방법](#ldap-%eb%8d%b0%ec%9d%b4%ed%84%b0-%ed%99%95%ec%9d%b8-%eb%b0%a9%eb%b2%95)
  - [백업](#%eb%b0%b1%ec%97%85)
  - [Consumer 서버 설정](#consumer-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95)

---

## 구성요소

- MirrorMode
- Delta-syncrepl
- TLS

---

## Servers

| Role | Domain | IP |
|---|---|---|
| Provider | ldap1.example.com | 192.168.9.101 |
| Provider | ldap2.example.com | 192.168.9.102 |
| Consumer | consumer.example.com | 192.168.9.103 |

---

## Vagrant

[Vagrantfile](/src/Vagrantfile)

**서버 시작/종료**

```bash
vagrant up
vagrant halt
```

**서버 접속**

```bash
vagrant ssh ldap1 # provider 1
vagrant ssh ldap2 # provider 2
vagrant ssh consumer # consumer
```

---

## 공통 설정

- [Hosts 설정](common.md/#hosts-%ec%84%a4%ec%a0%95)
- [패키지 설치](common.md/#%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
- [기존 설정 제거](common.md/#%ea%b8%b0%ec%a1%b4-%ec%84%a4%ec%a0%95-%ec%a0%9c%ea%b1%b0)
- [데이터베이스 디렉터리 생성](common.md/#%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4-%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%83%9d%ec%84%b1)
- [로그 설정](common.md/#%eb%a1%9c%ea%b7%b8-%ec%84%a4%ec%a0%95)
  - [rsyslog 설정](common.md/#rsyslog-%ec%84%a4%ec%a0%95)
  - [logrotate 설정](common.md/#logrotate-%ec%84%a4%ec%a0%95)

---

## 인증서

- [RootCA 인증서](certificates.md/#rootca-%ec%9d%b8%ec%a6%9d%ec%84%9c)
- [LDAP Provider 인증서](certificates.md/#ldap-provider-%ec%9d%b8%ec%a6%9d%ec%84%9c)
- [Replicator 인증서](certificates.md/#replicator-%ec%9d%b8%ec%a6%9d%ec%84%9c)
- [결과 파일](certificates.md/#%ea%b2%b0%ea%b3%bc-%ed%8c%8c%ec%9d%bc)
- [인증서 전달](certificates.md/#%ec%9d%b8%ec%a6%9d%ec%84%9c-%ec%a0%84%eb%8b%ac)

---

## 클라이언트 설정

[클라이언트 설정 방법: 3가지](client.md)

---

## Provider 1 서버 설정

- [관리자 비밀번호 생성](provider-1.md/#%ea%b4%80%eb%a6%ac%ec%9e%90-%eb%b9%84%eb%b0%80%eb%b2%88%ed%98%b8-%ec%83%9d%ec%84%b1)
  - [관리자 비밀번호 ](provider-1.md/#관리자-비밀번호-변경)
- [LDIF 작성](provider-1.md/#ldif-%ec%9e%91%ec%84%b1)
- [설정 적용](provider-1.md/#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
- [서버 실행](provider-1.md/#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)
- [디렉터리 정보 초기화](provider-1.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ec%b4%88%ea%b8%b0%ed%99%94)
  - [디렉터리 정보 확인](provider-1.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)

---

## Provider 2 서버 설정

- [LDIF 작성](provider-2.md/#ldif-%ec%9e%91%ec%84%b1)
- [설정 적용](provider-2.md/#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
- [서버 실행](provider-2.md/#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)
- [디렉터리 정보 확인](provider-2.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)

--- 

## Delta-Syncrepl 테스트

- [디렉터리 정보 추가](test-syncrepl.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ec%b6%94%ea%b0%80)
- [추가 정보 확인](test-syncrepl.md/#%ec%b6%94%ea%b0%80-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)

---

## phpLDAPadmin

- [패키지 설치](phpldapadmin.md/#%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
- [phpldapadmin 설정](phpldapadmin.md/#phpldapadmin-%ec%84%a4%ec%a0%95)
- [httpd 설정](phpldapadmin.md/#httpd-%ec%84%a4%ec%a0%95)
- [필요없는 계정 삭제](phpldapadmin.md/#%ed%95%84%ec%9a%94%ec%97%86%eb%8a%94-%ea%b3%84%ec%a0%95-%ec%82%ad%ec%a0%9c)
- [httpd 실행](phpldapadmin.md/#httpd-%ec%8b%a4%ed%96%89)
- [phpLDAPadmin 접속](phpldapadmin.md/#phpldapadmin-%ec%a0%91%ec%86%8d)

---

## LDAP 데이터 확인 방법

- [관리자 DN으로 로컬 서버 TLS 접속하는 명령어](commands.md/#%ea%b4%80%eb%a6%ac%ec%9e%90-dn%ec%9c%bc%eb%a1%9c-%eb%a1%9c%ec%bb%ac-%ec%84%9c%eb%b2%84-tls-%ec%a0%91%ec%86%8d%ed%95%98%eb%8a%94-%eb%aa%85%eb%a0%b9%ec%96%b4)
- [ldapi 프로토콜과 관리자 DN으로 로컬 서버 TLS 접속하는 명령어](commands.md/#ldapi-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c%ea%b3%bc-%ea%b4%80%eb%a6%ac%ec%9e%90-dn%ec%9c%bc%eb%a1%9c-%eb%a1%9c%ec%bb%ac-%ec%84%9c%eb%b2%84-tls-%ec%a0%91%ec%86%8d%ed%95%98%eb%8a%94-%eb%aa%85%eb%a0%b9%ec%96%b4)
- [ldap 프로토콜과 관리자 DN으로 로컬 서버 TLS 접속하는 명령어](commands.md/#ldap-%ed%94%84%eb%a1%9c%ed%86%a0%ec%bd%9c%ea%b3%bc-%ea%b4%80%eb%a6%ac%ec%9e%90-dn%ec%9c%bc%eb%a1%9c-%eb%a1%9c%ec%bb%ac-%ec%84%9c%eb%b2%84-tls-%ec%a0%91%ec%86%8d%ed%95%98%eb%8a%94-%eb%aa%85%eb%a0%b9%ec%96%b4)
- [접속할 서버 지정 후 관리자 DN으로 TLS 접속하는 명령어](commands.md/#%ec%a0%91%ec%86%8d%ed%95%a0-%ec%84%9c%eb%b2%84-%ec%a7%80%ec%a0%95-%ed%9b%84-%ea%b4%80%eb%a6%ac%ec%9e%90-dn%ec%9c%bc%eb%a1%9c-tls-%ec%a0%91%ec%86%8d%ed%95%98%eb%8a%94-%eb%aa%85%eb%a0%b9%ec%96%b4)
- [`uid=keanu`을 검색하는 명령어](commands.md/#uidkeanu%ec%9d%84-%ea%b2%80%ec%83%89%ed%95%98%eb%8a%94-%eb%aa%85%eb%a0%b9%ec%96%b4)

---

## 백업

- [LDAP 서버 설정 백업](backup.md/#ldap-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95-%eb%b0%b1%ec%97%85)
- [Accesslog 데이터 백업](backup.md/#accesslog-%eb%8d%b0%ec%9d%b4%ed%84%b0-%eb%b0%b1%ec%97%85)
- [디렉터리 데이터 백업](backup.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%eb%8d%b0%ec%9d%b4%ed%84%b0-%eb%b0%b1%ec%97%85)
- [LDAP 서버 종료](backup.md/#ldap-%ec%84%9c%eb%b2%84-%ec%a2%85%eb%a3%8c)
- [기존 서버 설정 삭제](backup.md/#%ea%b8%b0%ec%a1%b4-%ec%84%9c%eb%b2%84-%ec%84%a4%ec%a0%95-%ec%82%ad%ec%a0%9c)
- [기존 디렉터리 데이터 삭제](backup.md/#%ea%b8%b0%ec%a1%b4-%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%eb%8d%b0%ec%9d%b4%ed%84%b0-%ec%82%ad%ec%a0%9c)
- [LDAP 설정 등록](backup.md/#ldap-%ec%84%a4%ec%a0%95-%eb%93%b1%eb%a1%9d)
- [Accesslog 데이터 등록](backup.md/#accesslog-%eb%8d%b0%ec%9d%b4%ed%84%b0-%eb%93%b1%eb%a1%9d)
- [디렉터리 데이터 등록](backup.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%eb%8d%b0%ec%9d%b4%ed%84%b0-%eb%93%b1%eb%a1%9d)
- [LDAP 서버 실행](backup.md/#ldap-%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)

---

## Consumer 서버 설정

- [LDIF 작성](consumer.md/#ldif-%ec%9e%91%ec%84%b1)
- [설정 적용](consumer.md/#%ec%84%a4%ec%a0%95-%ec%a0%81%ec%9a%a9)
- [서버 실행](consumer.md/#%ec%84%9c%eb%b2%84-%ec%8b%a4%ed%96%89)
- [디렉터리 정보 확인](consumer.md/#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)
