목차
1. 개요
2. 사전 설치 환경
3. nagios 서버 설치
4. 웹 인터페이스 설치
5. nagios 플러그인 설치
6. 실행 및 nagios 모니터링 연동
 
 
 
 
1. 개요
Nagios는 Windows, Linux, 라우터 및 기타 네트워크 장치에서 실행되는 서비스 및 애플리케이션을 모니터링 할 수 있는 널리 사용되는 오픈 소스 모니터링 도구이다.

Nagios로 기본 서비스 및 속성을 모니터링 할 수 있으며 함께 제공되는 웹 인터페이스를 사용하여 모니터링 콘솔에 접근할 수 있다.

 
 
 
2. 사전 설치 환경
Nagios를 설치하기 전에, 시스템은 Nagios를 설치하기 위한 사전의 요구사항을 충족시켜야 한다. 웹 서버 (httpd), PHP, 컴파일러 및 개발 라이브러리를 먼저 설치한다.


$ yum -y install httpd php gcc glibc glibc-common wget perl gd gd-devel unzip zip
 
웹 인터페이스를 통해 외부 명령을 실행할 수 있도록 nagios 사용자 및 nagcmd 그룹을 만들고 nagcmd 및 apache 사용자를 nagcmd 그룹의 일부로 추가한다.

$ useradd nagios
$ groupadd nagcmd
$ usermod -a -G nagcmd nagios
$ usermod -a -G nagcmd apache
	
setlinux 설정을 off 한다.
$ vi /etc/selinux/config

....

SELINUX=disabled
 
3. Nagios 서버 설치
Nagios core를 다운로드 한다.


# wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.3.4.tar.gz

# tar xzf nagioscore.tar.gz

# cd nagioscore-nagios-4.3.4/
 

Nagios를 컴파일 하고 설치한다.


$ ./configure --with-nagios-group=nagios --with-command-group=nagcmd
$ make all
$ make install
$ make install-init
$ make install-config
$ make install-commandmode
	
 
 
4. 웹 인터페이스 설치
Nagios 웹 컨피규레이션을 설치한다.

$ make install-webconf
 

Nagios 웹 인터페이스에 로그인하기위한 사용자 계정 (nagiosadmin)을 만든다. 나중에 필요하므로 암호는 꼭 기억한다.

htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
 

Apache  웹 서버를 재시작한다.


$ systemctl restart httpd
$ systemctl enable httpd
 

샘플 구성 파일이  /usr /local/nagios /etc 디렉토리에 설치되었다. 

 

모니터링 시 통지내용을 메일로 받기 위해서 메일 주소를 연결해야 한다.

편집기로 /usr/local/nagios/etc/objects/contacts.cfg 설정 파일을 편집하고 nagiosadmin 연락처와 연결된 메일 주소를 변경하자.

$ vi /usr/local/nagios/etc/objects/contacts.cfg

define contact{
        contact_name                    nagiosadmin             ; Short name of user
        use                             generic-contact         ; Inherit default values from generic-contact template (defined above)
        alias                           Nagios Admin            ; Full name of user

        email                           hs04******@samsung.com      ; <<***** CHANGE THIS TO YOUR EMAIL ADDRESS ******
        }
 

 

5. Nagios 플러그인 설치
 

Nagios Plugins를 /tmp 디렉토리에 다운로드 한다.


$ cd /tmp
$ wget https://nagios-plugins.org/download/nagios-plugins-2.2.1.tar.gz
$ tar -zxvf nagios-plugins-2.2.1.tar.gz
$ cd /tmp/nagios-plugins-2.2.1/
플러그인을 컴파일 하고 설치한다.


$ ./configure --with-nagios-user=nagios --with-nagios-group=nagios
$ make
$ make install
 

 

6. 실행 및 nagios 모니터링 연동
샘플 구성파일을 검증해 본다.

$ /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
출력창에 에러가 없는지 확인한다.


Checking global event handlers...
Checking obsessive compulsive processor commands...
Checking misc settings...

Total Warnings: 0
Total Errors:   0

Things look okay - No serious problems were detected during the pre-flight chec
에러가 없으면 Nagios 서비스를 시작한다.

$ service nagios start
아래와 같은 메세지가 나오면 nagios 서비스가 정상실행되지 못한 것이다.

메세지는 nagios.servuce 를 찾을 수 없다고 나온다.

Redirecting to /bin/systemctl start nagios. service
Failed to start nagios. service: Unit not found.
아래와 같이 http://{서버 ip}/nagios 로 접근하면 Nagios Core가 Not running이라고 나온다.

그럼 nagios.service를 /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg 를 지정하도록 생성해야 한다.

nagios_login.png

[그림-1] Nagios 콘솔 접근

 

nagios.service 파일을 만든다.

# sudo vi /etc/systemd/system/nagios.service
해당 nagios.service에 다음의 내용을 추가한다.

[Unit]
Description=Nagios
BindTo=network.target


[Install]
WantedBy=multi-user.target


[Service]
User=nagios
Group=nagios
Type=simple
ExecStart=/usr/local/nagios/bin/nagios /usr/local/nagios/etc/nagios.cfg
nagios.service를 시작한다.

# systemctl enable /etc/systemd/system/nagios.service
# systemctl start nagios
# systemctl restart nagios
다시 Nagios의 서버로 접근해 본다. 다음과 같이 정상적으로 서비스가 시작되었음을 확인할 수 있다.

nagios_init.png

[그림-2] Nagios 콘솔의 서비스 정상동작

 

왼쪽의 Services를 클릭해 보면 다음과 같이 Nagios가 모니터링하고 있는 각각의 서비스에 대한 Status를 볼 수 있다.

nagios_service_monitoring.png

[그림-3] Nagios로 서버의 Services들에 대한 상태 확인

 

추가적으로 아래와 같이 아까 등록해 놓은 이메일 주소로 문제가 있는 서비스에 대한 모니터링 메세지가 전송된다.

***** Nagios *****

Notification Type: PROBLEM

Service: Swap Usage
Host: localhost
Address: 127.0.0.1
State: CRITICAL

Date/Time: Tue Jul 2 01:36:57 EDT 2019

Additional Info:

SWAP CRITICAL - 0% free (0 MB out of 0 MB) - Swap is either disabled, not present, or of zero size.
 

 
참고
Nagios 공식사이트: https://www.nagios.org/
