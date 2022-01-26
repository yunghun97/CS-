# AWS에 jenkins 설치법
### Ubuntu 20.04 LTS 기준
1. 
```bash
sudo apt-get update
```
2.
```bash
sudo apt-get install openjdk-8-jdk
```
3.
```bash
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
```
4.
```bash
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```
5.
```bash
sudo apt-get update
```
6.
```bash
sudo apt-get install jenkins
```
7.
```bash
sudo apt install git
```

8. 젠킨스 실행
```bash
sudo service jenkins start
sudo service jenkins status // 상태 확인
```
9. 포트 변경
```bash
sudo vim /etc/default/jenkins

HTTP_PORT=8080  // 원하는 포트로 변경
```
10. 재실행
```bash
sudo systemctl daemon-reload
sudo service jenkins restart 
```
11. 젠킨스 접속
```
httP://[aws주소]:포트번호
```