# 공통 설정

- [공통 설정](#%ea%b3%b5%ed%86%b5-%ec%84%a4%ec%a0%95)
  - [Hosts 설정](#hosts-%ec%84%a4%ec%a0%95)
  - [패키지 설치](#%ed%8c%a8%ed%82%a4%ec%a7%80-%ec%84%a4%ec%b9%98)
  - [기존 설정 제거](#%ea%b8%b0%ec%a1%b4-%ec%84%a4%ec%a0%95-%ec%a0%9c%ea%b1%b0)
  - [데이터베이스 디렉터리 생성](#%eb%8d%b0%ec%9d%b4%ed%84%b0%eb%b2%a0%ec%9d%b4%ec%8a%a4-%eb%94%94%eb%a0%89%ed%84%b0%eb%a6%ac-%ec%83%9d%ec%84%b1)
  - [로그 설정](#%eb%a1%9c%ea%b7%b8-%ec%84%a4%ec%a0%95)
    - [rsyslog 설정](#rsyslog-%ec%84%a4%ec%a0%95)
    - [logrotate 설정](#logrotate-%ec%84%a4%ec%a0%95)

## Hosts 설정

각 서버 `/etc/hosts`마다 전체 주소 도메인 네임(**FQDN**)을 등록한다.  
FQDN이 LDAP 설정값과 일치하지 않으면 LDAP 서버를 실행할 수 없다.

예를 들어, ldap1 서버에서 `/etc/hosts`를 고친다.

```bash
127.0.0.1        ldap1.example.com        ldap1
192.168.9.101    ldap1.example.com        ldap1
192.168.9.102    ldap2.example.com        ldap2
192.168.9.103    consumer.example.com     consumer
```

호스트 네임을 확인한다.

```bash
# 호스트 네임 확인 명령
hostname # ldap1
hostname -f # ldap1.example.com
# 호스트 네임 변경 명령
hostnamectl set-hostname ldap1
```

나머지 서버들도 `/etc/hosts`를 설정한다.

---

## 패키지 설치

OpenLDAP 라이브러리, 서버, 클라이언트를 설치한다.

```bash
yum install -y openldap openldap-servers openldap-clients
```

## 기존 설정 제거

패키지에 자동으로 설정된 slapd 설정을 제거한다.

```bash
rm -rf /etc/openldap/slapd.d
mkdir /etc/openldap/slapd.d
```

## 데이터베이스 디렉터리 생성

slapd가 사용할 데이터베이스 저장 경로를 생성한다.

```bash
mkdir /var/lib/ldap/data /var/lib/ldap/accesslog
chown ldap:ldap /var/lib/ldap/data /var/lib/ldap/accesslog
```

slapd 데이터베이스 저장 경로의 SELinux 보안 정보를 확인한다.

```bash
ls -ldZ /var/lib/ldap
# drwx------. ldap ldap system_u:object_r:slapd_db_t:s0  /var/lib/ldap
```

---

## 로그 설정

slapd 로그 디렉터리를 생성한다.

```bash
mkdir /var/log/slapd
```

### [rsyslog](https://www.rsyslog.com/category/guides-for-rsyslog/) 설정

slapd의 로그를 수집한다.

```bash
cat > /etc/rsyslog.d/slapd.conf << \EOF
$template slapdtmpl,"[%$DAY%-%$MONTH%-%$YEAR% %timegenerated:12:19:date-rfc3339%] %app-name% %syslogseverity-text% %msg%\n"
local4.*    /var/log/slapd/slapd.log;slapdtmpl
EOF
```

rsyslog를 재시작한다.

```bash
systemctl restart rsyslog
```

### [logrotate](https://linux.die.net/man/8/logrotate) 설정

rsyslog가 생성한 로그를 백업한다.

```bash
cat > /etc/logrotate.d/slapd << EOF
/var/log/slapd/slapd.log {
    compress
    copytruncate
    create 0600 root root
    daily
    dateext
    notifempty
    maxage 31
    missingok
    rotate 31
}
EOF
```

logrotate를 강제 실행한다.

```bash
/usr/sbin/logrotate -f /etc/logrotate.conf
```