# Codeigniter 3버전 환경 세팅
## 순서 
### 1. Bitnami 세팅 
>> 1. C:\work\wampstack-7.4.16-1\apache2\conf\httpd.conf     #Listen 에 포트 추가
(예시: Listen 8082) 
>> 2. C:\work\wampstack-7.4.16-1\apache2\conf\bitnami\bitnami.conf   #가상 호스트 추가
(예시: DocumentRoot "D:/hearald_project/spanishmaster_lms2022"
  	<Directory "D:/hearald_project/spanishmaster_lms2022"> ) 
>> 3. C:\work\wampstack-7.4.16-1\apache2\conf\bitnami\bitnami-apps-prefix.conf   #가상 호스트 conf 파일 include
(예시 : Include "C:/Bitnami/wampstack-7.4.27-0/apps/phpmyadmin/conf/httpd-prefix.conf"
	Include "D:/hearald_project/spanishmaster_lms2022/conf/httpd-prefix.conf" )
>> 4. 사용하고싶은 해당 디렉토리 내부에 conf 파일 생성후 그 아래 3개의 conf(메모장)을 만들어준다
(1) httpd-app-conf  
(2) httpd-prefix.conf
(3) httpd-vhosts.conf 
>>> (1) app-conf
경로 : D:\hearald_project\spanishmaster_lms2022\conf
예시 : <Directory "D:\hearald_project\spanishmaster_lms2022">
	Options +MultiViews
	AllowOverride All
	Order allow,deny
	allow from all
         </Directory>
>>> (2) prefix.conf 
경로 : D:\hearald_project\spanishmaster_lms2022\conf
예시 : Include "D:/hearald_project/spanishmaster_lms2022/conf/httpd-app.conf"
>>> (3) vhosts.conf
경로 : D:\hearald_project\spanishmaster_lms2022\conf
예시 : <VirtualHost *:8082>
  	ServerName spanishmaster_lms2022.example.com
  	DocumentRoot "D:/hearald_project/spanishmaster_lms2022"
  	Include "D:/hearald_project/spanishmaster_lms2022/conf/httpd-app.conf"
        </VirtualHost>

> 마지막으로 설정이 완료되면   
> 웹서버를 재시작해준다 (sql재시작은 할필요없음)
### 2. Codeigniter Framework 다운로드 
>> 해당 디렉토리 위치에 Codeigniter 3.13 압축 풀어주면 끝  