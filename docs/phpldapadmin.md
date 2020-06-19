# phpLDAPadmin

- [phpLDAPadmin](#phpldapadmin)
  - [패키지 설치](#%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
  - [phpldapadmin 설정](#phpldapadmin-%ec%84%a4%ec%a0%95)
  - [httpd 설정](#httpd-%ec%84%a4%ec%a0%95)
  - [필요없는 계정 삭제](#%ed%95%84%ec%9a%94%ec%97%86%eb%8a%94-%ea%b3%84%ec%a0%95-%ec%82%ad%ec%a0%9c)
  - [httpd 실행](#httpd-%ec%8b%a4%ed%96%89)
  - [phpLDAPadmin 접속](#phpldapadmin-%ec%a0%91%ec%86%8d)
  - [기타](#기타)
    - [DN 설정](#dn-설정)
    - [Crypt](#crypt)
    - [SSL 설정](#ssl-)

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

브라우저로 [192.168.9.101/phpldapadmin](http://192.168.9.101/phpldapadmin) 혹은 `192.168.9.101/ldapadmin` 에 접속한다.

- User Name: `cn=manager,ou=admins,dc=example,dc=com`
- Password: 관리자 비밀번호

URL 수정: `/phpldapadmin`, `ldapadmin`은 아파치 httpd 설정에서 변경할 수 있다.

---

## 기타 설정

### DN 설정

기본 DN 설정은 CN으로 되어있다. UID로 바꿀 수 있다.

```bash
vi /usr/share/phpldapadmin/templates/creation/posixAccount.xml
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

### SSL 설정

[SSL/TLS Strong Encryption: How-To](https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html)

#### mod_ssl 설치

`mod_ssl.so` 모듈을 설치한다.

설치 위치: `/etc/httpd/modules` → `/usr/lib64/httpd/modules`

```bash
yum install -y mod_ssl
```

`/etc/httpd/conf.modules.d/00-ssl.conf`에서 다음을 확인한다.

```
LoadModule ssl_module modules/mod_ssl.so
```

#### httpd 설정

`/etc/httpd/conf.d/ssl.conf` 설정을 확인한다.

```config
Listen 443 https

<VirtualHost _default_:443>
    SSLEngine on
    SSLCertificateFile /etc/openldap/certs/example.com.crt
    SSLCertificateKeyFile /etc/openldap/certs/example.com.key
    SSLCACertificateFile /etc/openldap/certs/rootca.crt
</VirtualHost>
```

```bash
systemctl restart httpd
```

#### Port 변경

`/etc/httpd/conf.d/ssl.conf` 설정을 변경한다.

`80, 81, 443, 488, 8008, 8009, 8443, 9000` 중 포트 하나를 선택하면 좋다.

```config
Listen 8443

<VirtualHost _default_:8443>
</VirtualHost>
```

서버를 재시작한다.

```bash
systemctl restart httpd
```

`https://HOST_NAME:8443/phpldapadmin`으로 접속한다.

만약 httpd 실행 오류가 발생했다면 아래 과정처럼 포트 보안 설정 변경 한다.

##### SELinux 설정 변경

SELinux를 확인한다.

```bash
sestatus
```

```bash
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      31
```

##### setenforce

setenforce 명령으로 SELinux 모드를 변경할 수 있다.

```bash
setenforce 0
```

```bash
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive # 변경됨
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      31
```

##### semanage

```bash
yum install -y policycoreutils-python
```

HTTP 포트들을 확인한다.

```bash
semanage port -l | grep http
```

```bash
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
```

사용할 포트가 설정되어 있는지 확인한다.

```bash
semanage port -m -t http_port_t -p tcp 7443
```

```bash
ValueError: Port @tcp/7443 is not defined
```

포트를 추가한다.

```bash
semanage port -a -t http_port_t -p tcp 7443
```

포트를 다시 확인한다.

```bash
semanage port -l | grep http
```
