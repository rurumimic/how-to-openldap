# phpLDAPadmin

- [phpLDAPadmin](#phpldapadmin)
  - [패키지 설치](#%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
  - [phpldapadmin 설정](#phpldapadmin-%ec%84%a4%ec%a0%95)
  - [httpd 설정](#httpd-%ec%84%a4%ec%a0%95)
  - [필요없는 계정 삭제](#%ed%95%84%ec%9a%94%ec%97%86%eb%8a%94-%ea%b3%84%ec%a0%95-%ec%82%ad%ec%a0%9c)
  - [httpd 실행](#httpd-%ec%8b%a4%ed%96%89)
  - [phpLDAPadmin 접속](#phpldapadmin-%ec%a0%91%ec%86%8d)

---

## 패키지 설치

```bash
yum install -y epel-release
yum install -y phpldapadmin
```

## phpldapadmin 설정

```bash
vi /etc/phpldapadmin/config.php
```

VI 줄 이동 단축키: `ESC` + `:숫자`

- 298: `$servers->setValue('server','host','ldap://ldap1.example.com')`
- 340: `$servers->setValue('server','tls',true)`
- 397: `$servers->setValue('login','attr','dn')`

## httpd 설정

`Require local`를 `Require all granted`로 변경한다.

```bash
vi /etc/httpd/conf.d/phpldapadmin.conf
```

## 필요없는 계정 삭제

phpLDAPadmin을 설치하면서 생성된 *caddy* 계정을 삭제한다.

```bash
cat /etc/passwd
# caddy:x:996:993:Caddy web server:/var/lib/caddy:/sbin/nologin
```

```bash
userdel -r caddy
```

## httpd 실행

```bash
systemctl start httpd
systemctl enable httpd
systemctl status httpd
```

## phpLDAPadmin 접속

브라우저로 [192.168.9.101/phpldapadmin](http://192.168.9.101/phpldapadmin)에 접속한다.

- User Name: `cn=manager,ou=admins,dc=example,dc=com`
- Password: 관리자 비밀번호

---

## 기타 설정

### DN 설정: UID

기본 DN 설정은 CN으로 되어있다. UID로 바꿀 수 있다.

```bash
vi /usr/share/phpldapadmin/templates/creation/
```

9번째 줄을 변경한다.

```xml
<rdn>uid</rdn>
```

### Crypt

```bash
vi /usr/share/phpldapadmin/lib/functions.php
```

2150번째 줄을 다음과 같이 변경한다. `$5$`는 `SHA-256`을 의미한다.

```php
case 'crypt':
  if ($_SESSION[APPCONFIG]->getValue('password', 'no_random_crypt_salt'))
    $new_value = sprintf('{CRYPT}%s',crypt($password_clear,substr($password_clear,0,2)));
  else
    $salt = substr(str_replace('+','.',base64_encode(md5(mt_rand(), true))),0,16);
    $new_value = sprintf('{CRYPT}%s', crypt($password_clear, sprintf('$5$%s$', $salt)));
  break;
```

