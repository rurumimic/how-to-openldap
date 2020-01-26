# Delta-Syncrepl 테스트

- [Delta-Syncrepl 테스트](#delta-syncrepl-%ed%85%8c%ec%8a%a4%ed%8a%b8)
  - [디렉터리 정보 추가](#%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%a0%95%eb%b3%b4-%ec%b6%94%ea%b0%80)
  - [추가 정보 확인](#%ec%b6%94%ea%b0%80-%ec%a0%95%eb%b3%b4-%ed%99%95%ec%9d%b8)

Delta-syncrepl을 확인하는 과정이다.

## 디렉터리 정보 추가

ldap2 서버에서 다음 명령을 실행한다.

```bash
ldapadd -x -W -D "cn=manager,ou=admins,dc=example,dc=com" -Z << EOF
dn: cn=Keanu Reeves,ou=people,dc=example,dc=com
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Keanu Reeves
uid: keanu
sn: Reeves
givenName: Keanu
uidNumber: 1001
gidNumber: 500
homeDirectory: /home/users/keanu
loginShell: /bin/bash
EOF
```

```bash
Enter LDAP Password: # 관리자 비밀번호 입력
adding new entry "cn=Keanu Reeves,ou=people,dc=example,dc=com"
```

## 추가 정보 확인

ldap1 서버에서 확인한다.

```bash
ldapsearch -x -W -D "cn=manager,ou=admins,dc=example,dc=com" uid=keanu -b dc=example,dc=com -Z
```

```bash
# Keanu Reeves, people, example.com
dn: cn=Keanu Reeves,ou=people,dc=example,dc=com
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
cn: Keanu Reeves
uid: keanu
sn: Reeves
givenName: Keanu
uidNumber: 1001
gidNumber: 500
homeDirectory: /home/users/keanu
loginShell: /bin/bash
```

Provider 2개로 MirrorMode 설정을 완료했다.