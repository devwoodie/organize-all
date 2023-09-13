## Aws ec2 인스턴스에 Apache 웹서버 띄우기
> - 프론트 인스턴스는 80포트, 443포트 열려있어야함
> - 80포트는 443포트로 리다이렉트 되게 설정해야함
---

1. 터미널에서 인스턴스 퍼블릭 IPv4 DNS, pem 키로 인스턴스 접속
2. 프로젝트 폴더 들어가서 아파치 설치

```bash
## 설치 가능한 리스트 업데이트
sudo atp-get update 
or
sudo yum update -y

## 아파치 설치
sudo apt-get install apache2
or
sudo yum install httpd -y

## 설치 확인
apache2-v

## 아파치 시작
sudo systemctl start httpd
or
sudo service apache2 start
```
3. 설치된 폴더 중 etc/apache2 접속 -> sites-available 폴더 접속
4. sites-available 폴더 내 도메인.cof 파일 생성

```bash 
sudo vi 도메인.conf
```
5. 도메인.conf 파일 내에 아래 코드 입력 후 도메인, 파일 경로 변경

```
<VirtualHost *:80>
ServerName (도메인)
ServerAlias (도메인)
ServerAdmin webmaster@도메인
DocumentRoot (index.html 경로)
<Directory (index.html 경로)>
Options Indexes Follow SymLinks
AllowOverride all
Require all granted
```
6. 변경 후 아래 명령어 입력하면 /etc/apache2/sites-enabled 에 링크 파일 생성
```bash
 sudo a2ensite 도메인.conf
```

`오류`
```bash
sudo a2enmod rewrite
or
sudo service apache2 restart
```


