
# MySQL에서 항상 아이디와 호스트를 함게 명시해야 하는 이유는? (53p)

MySQL의 사용자는 다른 DBMS와는 다르게 사용자의 계정뿐 아니라 사용자의 접속 지점(클라이언트가 실행된 호스트명이나 도메인 또는 IP 주소)도 계정의 일부가 된다. 따라서 MySQL에서 계정을 언급할 때는 다음과 같이 항상 아이디와 호스트를 함께 명시해야 한다.


# 시스템 계정(System Account)과 일반 계정(Regular Account) ? (54p)

MySQL 8.0부터 계정은 SYSTEM_USER 권한을 가지고 있느냐에 따라 시스템 계정과 일반 계정으로 구분된다.
여기서 소개하는 시스템 계정은 MySQL 서버 내부적으로 실행되는 백그라운드 스레드와는 무관하며, 시스템 계정도 일반 계정과 같이 사용자를 위한 계정이다. 시스템 계정은 데이터베이스 서버 관리자를 위한 계정이며, 일반 계정은 응용 프로그램이나 개발자를 위한 계정 정도로 생각하자.

이렇게 시스템 계정과 일반 계정의 개념이 도입된 것은 DBA(데이터베이스 관리자) 계정에는 SYSTEM_USER 권한을 할당하고 일반 사용자를 위한 계정에는 SYSTEM_USER 권한을 부여하지 않게 하기 위해서다.

# MySQL 내장된 계정? 주의점은? (55p)

MySQL 서버에는 다음과 같이 내장된 계정들이 있는데, 'root' @ 'localhost'를 제외한 3개의 계정은 내부적으로 각기 다른 목적으로 사용되므로 삭제되지 않도록 주의하자.

- 'mysql.sys' @ 'localhost': MySQL 8.0부터 기본으로 내장된 sys 스키마의 객체(뷰나, 함수, 그리고 프로시저)들의 DEFINER로 사용되는 계정
- 'mysql.session' @ 'localhost' : MySQL 플러그인이 서버로 접근할 때 사용되는 계정
- 'mysql.infoschema' @'localhost' : information_schema 에 정의된 뷰의 DEFINER로 사용되는 계정

위에 언급한 3개의 계정은 처음부터 잠겨(account_locked 칼럼) 있는 상태이므로 의도적으로 잠긴 계정을 풀지 않는 한 악의적인 요도로 사용할 수 없으므로 보안을 걱정하지는 않아도 된다.

# IDENTIFIED WITH ??? (56p ~ 57p)

사용자의 인증 방식과 비밀번호를 설정한다. IDENTIFIED WITH 뒤에 반드시 인증 방식(인증 플러그인의 이름)을 명시해야 하는데, MySQL 서버의 기본 인증 방식을 사용하고자 한다면 IDENTIFIED BY 'password' 형식으로 명시해야 한다. MySQL 서버에서는 다양한 인증 방식을 플러그인 형태로 제공하며, 다음 4가지 방식이 가장 대표적이다.

 (자세한 설명은 56p ~ 57p 참고)
- Native Pluggable Authentication 
- Caching SHA-2 Pluggable Authentication
- PAM Pluggable Authentication
- LDAP Pluggable Authentication

# REQUIRE ?? (58p)

MySQL 서버에 접속할 때 암호화된 SSL/TLS 채널을 사용할지 여부를 설정한다. 만약 별도로 설정하지 않으면 비암호화 채널로 연결하게 된다. 하지만 REQUIRE 옵션을 SSL로 설정하지 않았다고 하더라도 Caching SHA-2 Authentication 인증 방식을 사용하면 암호화된 채널만으로 MySQL 서버에 접속할 수 있게 된다.


# PASSWORD EXPIRE ?? (58p)

비밀번호의 유효 기간을 설정하는 옵션이며, 별도로 명시하지 않으면 default_password_lifetime 시스템 변수에 저장된 기간으로 유효 기간이 설정된다. PASSWORD EXPIRE 절에 설정 가능한 옵션은 다음과 같다.

- PASSWORD EXPIRE: 계정 생성과 동시에 비밀번호의 만료 처리
- PASSWORD EXPIRE NEVER: 계정 비밀번호의 만료 기간 없음
- PASSWORD EXPIRE DEFAULT: default_password_lifetime 시스템 변수에 저장된 기간으로 비밀번호의 유효기간을 설정
- PASSWORD EXPRIE INTERVAL n DAY: 비밀번호의 유효 기간을 오늘부터 n일자로 설정

# PASSWORD HISTORY ?? (58p)

한 번 사용했던 비밀번호를 재사용하지 못하게 설정하는 옵션인데, PASSWORD HISTORY 절에 설정 가능한 옵션은 다음과 같다.

- PASSWORD HISTORY DEFALUT: password_history 시스템 변수에 저장된 개수만큼 비밀번호의 이력을 저장하며, 저장된 이력에 남아있는 비밀번호는 재사용할 수 없다.
- PASSWORD HISTORY n: 비밀번호의 이력을 최근 n개까지만 저장하며, 저장된 이력에 남아있는 비밀번호는 재사용할 수 없다.

# PASSWORD REUSE INTERVAL ?? (59p)

한 번 사용했던 비밀번호의 재사용 금지 기간을 설정하는 옵션이며, 별도로 명시하지 않으면 password_reuse_interval 시스템 변수에 저장된 기간으로 설정된다. PASSWORD REUSE INTERVAL 절에 설정 가능한 옵션은 다음과 같다.

- PASSWORD REUSE INTERVAL DEFAULT: password_reuse_interval 변수에 저장된 기간으로 설정
- PASSWORD REUSE INTERVAL n DAY: n 일자 이후에 비밀번호를 재사용할 수 있게 설정

# PASSWORD REQUIRE ?? (59p)

비밀번호가 만료되어 새로운 비밀번호로 변경할 때 현재 비밀번호(변경하기 전 만료된 비밀번호)를 필요로 할지 말지를 결정하는 옵션이며, 별도로 명시하지 않으면 password_require_current 시스템 변수의 값으로 설정된다. 옵션은 다음과 같다.

- PASSWORD REQUIRE CURRENT: 비밀번호를 변경할 때 현재 비밀번호를 먼저 입력하도록 설정
- PASSWORD REQUIRE OPTIONAL: 비밀번호를 변경할 때 현재 비밀번호를 입력하지 않아도 되도록 설정
- PASSWORD REQUIRE DEFAULT: password_require_current 시스템 변수의 값으로 설정

# ACCOUNT LOCK / UNLOCK (59p)

계정 생성 시 또는 ALTER USER 명령을 사용해 계정 정보를 변경할 때 계정을 사용하지 못하게 잠글지 여부를 결정한다.

- ACCOUNT LOCK: 계정을 사용하지 못하게 잠금
- ACCOUNT UNLOCK: 잠긴 계정을 다시 사용 가능 상태로 잠금 해제








