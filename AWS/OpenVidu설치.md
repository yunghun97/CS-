## 기본 환경
- AWS EC2
- Ubuntu 20.04
- Maven 3.6.3
## OpenVidu 설치
1. 서버 포트 구성
```bash
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 3478/tcp
sudo ufw allow 3478/udp
sudo ufw allow 40000:57000/tcp
sudo ufw allow 40000:57000/udp
sudo ufw allow 57001:65535/tcp
sudo ufw allow 57001:65535/udp
```
2. 설치 권장 폴더 생성 및 이동
```bash
mkdir opt
cd opt
```
3. 설치 스크립트 다운로드 및  실행
```bash
curl https://s3-eu-west-1.amazonaws.com/aws.openvidu.io/install_openvidu_latest.sh | bash

```
![image](https://user-images.githubusercontent.com/71022555/151701914-276b12bb-700f-4b31-96a3-388c18cf26c7.png)
4. 사진에 있는 순서대로 실행
```bash
cd openvidu
vim nano.env #
```
5. nano.env 설정
```bash
DOMAIN_OR_PUBLIC_IP=k5팀코드.p.ssafy.io
OPENVIDU_SECRET=MY_SECRET # 변경 가능
CERTIFICATE_TYPE=selfsigned
#HTTP_PORT=80
#HTTPS_PORT=443
LETSENCRYPT_EMAIL=your@email.co
```
![image](https://user-images.githubusercontent.com/71022555/151702012-5e077a48-456d-4bec-aa0c-ae6310df0cb7.png)

## OpenVidu-Server Custom Dockerizing
1. 디렉토리 생성 후 진입 : /home/ubuntu/ 에서 
```bash
mkdir OpenVidu
cd OpenVidu
```
2. OpenVidu Git clone
```bash
git clone https://github.com/OpenVidu/openvidu.git
```
3. 받아 진 openVidu 폴더 진입
```bash
cd openvidu
```
4. 패키지 클린
```bash
mvn clean install -U
```
5. openvidu/openvidu-server 폴더 지입
```
cd openvidu-server
```
6. 패키지 클린
```bash
mvn clean install -U
```
7. 관리자 권한 부여
```bash
sudo su
```
8. 루트 디렉토리 이동
```bash
cd /
```
9. SSL 인증서 경로 진입
```bash
cd /etc/letsencrypt/live/{SSL 인증 받은 도메인}/
# 이떄 도메인은 아까 설정했던 k5팀코드.p.ssafy.io
```
10. ssl
```bash
openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -out k5팀코드.p.ssafy.io.p12 --name ssafy -CAfile chain.pem -caname root

# 이때 설정하는 비번 보관해두기
# k5팀코드.p.ssafy.io.p12 생성되는지 확인
```
11. openvidu-server 파일에 ssl 인증서 복사, 붙여넣기
a. openvidu-server 의 application.properties가 있는 경로 까지 이동
```bash
cd openvidu/openvidu-server/src/main/resources/
```
b. ssl 인증서 복붙
```bash
cp /etc/letsencrypt/live/k5c202.p.ssafy.io/k5팀코드.p.ssafy.io.p12./
```
12. application.properties 수정
```bash
# 편집기 들어가기
vi application.properties

# 수정하는 내용
server.address=0.0.0.0
server.ssl.enabled=true
server.ssl.key-store=classpath:k5팀코드.p.ssafy.io.p1
server.ssl.key-store-password=your-password # 위 10번에서 설정한 비번
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=ssafy
server.servlet.session.cookie.name=OVJSESSIONID
logging.level.root=info
spring.main.allow-bean-definition-overriding=true
SUPPORT_DEPRECATED_API=true
DOTENV_PATH=.
DOMAIN_OR_PUBLIC_IP=k5팀코드.p.ssafy.io
OPENVIDU_SECRET=MY_SECRET # openvidu .env 설정과 통일
CERTIFICATE_TYPE=selfsigned
HTTPS_PORT=5443 # 포트번호는 자유
KMS_URIS=["ws://localhost:8888/kurento"]
```
13. openvidu 최상위 파일로 이동
14. 메이븐 빌드
```bash
mvn package -DskipTests
```
15. openvidu-server 파일로 이동
```bash
cd openvidu-server
```
16. 파일 접근 권한 부여
```bash
chmode 777 create_image.sh
```
17. 도커 이미지 생성
```bash
./create_image.sh 2.20.1
# 이때 2.20.1은 기존의 openvidu server가 2.20.0 버전으로 도커 실행중이라 구분을 위해 버전을 달리 해줌
```
18. 
```bash
cd opt/openvidu
```
19. 도커 컴포즈 파일 수정
```bash
vi docker-compose.yml
# 내용
services:
openvidu-server:
image: openvidu/openvidu-server:2.20.1
restart: on-failure
network_mode: host
entrypoint: ['/usr/local/bin/entrypoint.sh']
volumes:
- /var/run/docker.sock:/var/run/docker.sock
- ${OPENVIDU_RECORDING_PATH}:${OPENVIDU_RECORDING_PATH}
- ${OPENVIDU_RECORDING_CUSTOM_LAYOUT}:${OPENVIDU_RECORDING_CUSTOM_LAYOUT}
- ${OPENVIDU_CDR_PATH}:${OPENVIDU_CDR_PATH}
env_file:
- .env
environment:
- SERVER_SSL_ENABLED=true
- SERVER_PORT=5443 # application.properties 설정과 일치
- KMS_URIS=["ws://localhost:8888/kurento"]
- COTURN_REDIS_IP=127.0.0.1
OpenVidu 설치부터 테스트까지 6
- COTURN_REDIS_PASSWORD=${OPENVIDU_SECRET}
- COTURN_IP=${COTURN_IP:-auto-ipv4}
```
## 실행
1. /opt/openvidu 경로에서 
```bash
./openvidu start
# 이전에 실행 했다면 stop -> start  또는 retart
```
2. 도커 확인
```bash
docker ps
```