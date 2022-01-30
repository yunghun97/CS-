# redis AWS Docker로 설치
1. docker pull redis
```bash
docker pull redis 
```
2. 이미지 확인
``` 
docker images
```
3. 도커로 만들기
```bash
sudo docker run -d -p 6380:6379 --name 이름 redis --requirepass 비번
```
4. 접속
```bash
docker exec -it 컨테이너아이디 redis-cli 
```