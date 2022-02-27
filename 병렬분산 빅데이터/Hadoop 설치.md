## Hadoop 설치
    - Ubuntu 20.24 LTS 기준
1. 기본 설치    
```bash
wget http://kdd.snu.ac.kr/~kddlab/Project.tar.gz
```  

```bash
tar zxf Project.tar.gz
sudo chown -R hadoop:hadoop Project
cd Project
sudo mv hadoop-3.2.2 /usr/local/hadoop
sudo apt update
sudo apt install ssh openjdk-8-jdk ant -y
./set_hadoop_env.sh
source ~/.bashrc
```
2. 이동  
```bash
cd /home/hadoop 
```  

3. Empty 'ssh key' generation
```bash
ssh-keygen -t ras -P ""
# 저장할 파일을 물어보면 default로 enter만 친다.
cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys

# 제대로 생성되었는지 확인
ssh localhost
# 질문이 뜨면 yes를 입력하고
# 그 다음 비밀번호를 물어보지 않고 prompt가 뜨면 성공
```

### Hadoop 실행을 위한 준비
- 모든 명령은 hadoop 계정에서
    - Path를 지정하기 위해 /home/hadoop에서 source .bashrc를 실행
- Name node format
    - Disk format과 같은 개념
    - $ hadoop namenode -format
- Dfs daemon start
    - $ start-dfs.sh
- MapReduce daemon start(standalone 모드에서는 불필요)
    - $ start-mapred.sh ($ start-yarn.sh 으로 바뀐 듯 함)
- 확인
    - 수행중인 java 프로세스 리스트를 확인한다.
    - $ jps
        - SecNameNode
        - ondaryNameNode
        - DataNode
        - TaskTracker (Standalone에서는 불필요)
        - JobTracker (Standalone에서는 불필요)
  
1. hadoop 계정의 HDFS 디렉토리 파일 확인
```bash
hdfs dfs -ls /
```
2. hadoop 계정의 HDFS상에서의 맨 위에 user 디렉토리를 생성
```bash
hdfs dfs -mkdir /user
```
3. hadoop 계정의 HDFS상에서 /user 디렉토리 안에 hadoop 디렉토리 생성
```bash
hdfs dfs -mkdir /user/hadoop
```  
결과 
![결과](https://user-images.githubusercontent.com/71022555/155881714-194069d8-960c-40a1-9e6b-c2896079527b.png)  

 
