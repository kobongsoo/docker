# docker

## 명령어
- **마운트된 볼륨 데이터를 호스트로 백업 하기**
```
docker run --rm -v <volume명>:/source -v .\study\backup:/target busybox tar cvzf /target/backup.tar.gz -C /source .
```
- **docker 컨테이너에서 빠져 나오기(컨테이너 실행상태 유지)**
```
ctrl+P, Q 
```
- docker 버전 확인
```
docker -v
docker version
```
- docker 실행 확인
```
docker ps
docker ps -a
```
- docker 실행
```
docker run <옵션(-d/-it)> --name <컨테이너명> -p <호스트포트>:<컨테이너포트> <이미지명>
```
- docker 종료
```
docker stop  # docker 실행 중지
docker rm    # docker 삭제

# stop/rm 을 한꺼번에 
docker rm -f <name>

# 모든 docker 종료/rm
docker container prune
```
- docker 이미지 확인 / 이미지 내용 상세 보기 / 이미지 삭제
```
docker image ls / docker image inspect <name> / docker image rm <name>
```
- docker 설치 로그 보기
```
docker logs name
```
- docker 볼륨 확인 / 볼륨 삭제 
```
docker volume ls / docker image rm <name>
```
- docker 네트워크 확인 / 네트워크 삭제
```
docker network ls / docker network rm <name>
```
- docker cp 컨테이너<->로컬 파일 복사 
```
# docker 컨테이너 안에 있을 파일을 로컬로 복사
docker cp <컨테이너명>:<컨테이너파일> <로컬경로>
docker cp tmp_container:/root/data/test.md ~/data/ 
```
```
# docker 로컬파일을 컨테이너 안에 폴더로 복사
docker cp <로컬파일> <컨테이너명>:<컨테이너경로>
docker cp ~/data/test.md tmp_container:/root/data/
```
- **docker 이미지 옮기기**
```
# tar 로 이미지를 압축해서 옮김.
docker save -o <파일명.tar> <이미지 명>
docker save -o apache.tar httpd:latest
```
- **컨테이너 내부 shell 실행.**
```
docker exec -it <실행중인 컨테이너명> /bin/bash
```
- 컨테이너 이미지 만들기
```
docker commit <실행중인 컨테이너명> <이미지파일명>
```
- 리눅스 버전 확인
```
$ cat /etc/*release*
```
- hub.docker.com 로그아웃 하기
```
# docker push 에러 (denied: requested access to the resource is denied)나면 hub.docker 로그아웃하고 나서 다시 로그인 하면됨.
docker logout

docker login
Username : bong9431
Password :
```

## 1. docker 설치
- 윈도우 os : WSL2 기반으로 [Docker desktop](https://docs.docker.com/desktop/install/windows-install/) 다운로드 해서 설치
- 리눅스 :  get.docker.com 스크립트 이용, root 권한으로 전환( sudo su - root) 후 아래 순서대로 실행.
1. install-docker.sh 파일 다운로드 : curl -fsSL https://get.docker.com -o install-docker.sh   
2. install-docker.sh 파일 확인 (pass해도 됨) : cat install-docker.sh                                   
3. install-docker.sh 테스트로 실행과정 진행해봄(설치는 안됨)(pass해도 됨): sh install-docker.sh --dry-run 
4. install-docker.sh 실제 설치: sh install-docker.sh

### docker desktop 설정
- vm 이미지 경로 : settings-Resources-Disk image location 경로 변경
  
  ![image](https://github.com/kobongsoo/docker/assets/93692701/0b6a2851-f96d-494a-9dc0-7f8eb9ec346b)
  
- docker 용량 : settings-Docker engine 에서 defaultKeepSoorage 용량 변경후 [Apply&restart] 클릭
  
  ![image](https://github.com/kobongsoo/docker/assets/93692701/5f1b6a8f-7b0c-4bde-bc31-086ce20102ae)

- docker 메모리 설정 : C:\Users\{사용자} 경로에 ".wslconfig" 파일을 만들고, 아래처럼 입력
  ![image](https://github.com/kobongsoo/docker/assets/93692701/3cd2bda3-cbd9-43aa-ae46-f8618c876cff)

```
  [wsl2]
  memory=6GB
  processors=6
```
  
- docker desktop에서 볼륨설정 경로 찾기
  <br> 출처 : https://velog.io/@ette9844/Windows10-%EC%97%90%EC%84%9C-varlibdocker-%EC%B0%BE%EA%B8%B0
  <br>예시)
  <br> docker run -it --name embed -p 9000:9000 -p 8888:8888 --net k_es_network **-v /embed_data:/var/embed_data** bong9431/embed:1.0
  <br> 이때 embed_data 로컬 폴더는 docker desktop cmd 에서 볼수 없음.
  <br> 아래처럼 현재 디렉토리를 ubuntu 컨테이너를 실행해서 data 폴더로 마운트해서 /data 폴더에서 확인해야 함.(**docker run -v/:/data -it ubuntu /bin/bash**)
  <br> 참고로 이때 **docker 폴더 경로는 /data/var/lib/docker**
  ```
  F:\docker\embed>docker run -v/:/data -it ubuntu /bin/bash
  root@1bf0a2f2e670:/# chroot /data
  sh-5.1# ls
  A             D  H  L        O  S       Users    X  b     cores  embed_data  h     k      m      o        proc  run   sys  usr  x
  Applications  E  I  Library  P  System  V        Y  bin   d      etc         home  l      media  opt      q     s     t    v    y
  B             F  J  M        Q  T       Volumes  Z  boot  dev    f           i     lib    mnt    p        r     sbin  tmp  var  z
  C             G  K  N        R  U       W        a  c     e      g           j     lib64  n      private  root  srv   u    w
  sh-5.1# cd embed_data
  sh-5.1# ls
  test.ipynb  test.txt
  sh-5.1#
  ```
## 2. docker 이미지 만들기
- [hub.docker](https://hub.docker.com/)에 회원 가입 후 [Create repository] 로 rep 만듬.
  ![image](https://github.com/kobongsoo/docker/assets/93692701/420890ee-a8b2-486e-b896-d84dcacf1a60)

### 예시) jupyter lab 설치
1. base 이미지 설치<br>
**docker run -it --name jupyter -p 8888:8888 ubuntu:22.04** <br>

2. jupyter lab 설치 <br>
apt-get update<br>
apt-get install python3 python3-pip -y <br>
**pip3 install jupyter lab**<br>

3. jupyter lab 구동 & 동작확인 <br>
 mkdir /notebooks <br>
**jupyter lab --ip=0.0.0.0 --port=8888 --allow-root --notebook-dir=/notebooks &** <br>
**ctrl+P, Q** # app 실행상태로 쉘 빠저나옴<br> 
웹브라우저에서 접속 확인<br>

4. repo 만들고 hub.docker 로그인<br>
hub.docker.com 에 접속하여 jupyterlab repo 만듬 <br>
docker login<br>

5. commit으로 이미지 만듬<br>
**docker commit -a "bong9431" -m "jupyterlab commit" jupyter jupyterlab:1.0**<br>
docker image ls<br>

6. tag 만듬.<br>
: repo 이름에 맞게 이미지 복사본 만듬.<br>
**docker tag jupyterlab:1.0 bong9431/jupyterlab:1.0**<br>

7. hub.docker에 커밋(push로 올림)<br>
**docker push bong9431/jupyterlab:1.0**<br>

## 3. Dockerfile로 이미지 만들기

1. Dockerfile 생성<br>
- docker 폴더를 만들고, vi Dockerfile 열어서, 아래 내용 복사해서 Dockerfile 생성.
- Dokerfile reference는 [여기](https://docs.docker.com/engine/reference/builder/) 참조
```
# basic image
FROM ubuntu:22.04

# image creator
MAINTAINER bong

# key=value
LABEL "compay"="mocomsys"

# run command inside container
RUN apt-get update
RUN apt-get install python3 python3-pip -y
RUN pip3 install jupyter lab
RUN mkdir /jupyterdir
VOLUME /jupyterdir
CMD jupyter lab --ip=0.0.0.0 --port=8888 --allow-root --notebook-dir=/jupyterdir
```

2. Dockerfile 빌드 / 실행<br>
**docker build -t jupyterlab:1.0 -f Dockerfile . (반드시 뒤에 한깐 띄고 . 붙여야함)**
<br>docker run -d --name myjupyter -p 8888:8888 jupyterlab:1.0
```
F:\docker\dockerfile>docker build -t jupyterlab:1.0 -f Dockerfile .
[+] Building 2.6s (9/9) FINISHED                                                                         docker:default
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 2B                                                                                    0.0s
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 394B                                                                               0.0s
 => [internal] load metadata for docker.io/library/ubuntu:22.04                                                    0.0s
 => [1/5] FROM docker.io/library/ubuntu:22.04                                                                      0.0s
 => CACHED [2/5] RUN apt-get update                                                                                0.0s
 => CACHED [3/5] RUN apt-get install python3 python3-pip -y                                                        0.0s
 => CACHED [4/5] RUN pip3 install jupyterlab                                                                       0.0s
 => [5/5] RUN mkdir /jupyterdir                                                                                    0.4s
 => exporting to image                                                                                             2.1s
 => => exporting layers                                                                                            2.1s
 => => writing image sha256:8bcd5b3b53b1dc7bb16db42e48df743736932a484bb9b9e8aa408d5e3eff9d27                       0.0s
 => => naming to docker.io/library/jupyterlab:1.0                                                                  0.0s
``` 
4. hub.docker에 커밋<br>
docker tag jupyterlab:1.0 bong9431/jupyterlab:1.0<br>
docker image ls<br>
docker login<br>
docker push bong9431/jupyterlab:1.0<br>
```
F:\docker\dockerfile>docker image ls
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
bong9431/jupyterlab   1.0       8bcd5b3b53b1   53 seconds ago   659MB
jupyterlab            1.0       8bcd5b3b53b1   53 seconds ago   659MB
ubuntu                22.04     3565a89d9e81   2 weeks ago      77.8MB
```
```
F:\docker\dockerfile>docker push bong9431/jupyterlab:1.0
The push refers to repository [docker.io/bong9431/jupyterlab]
b05ecf5f512d: Pushed
6310d48066b7: Pushed
1e464e5e5fc2: Pushed
8de4c06f19af: Pushed
01d4e4b4f381: Layer already exists
1.0: digest: sha256:c87e6c9d9a7288dca5e14ee31880e690bc58da4e7b4d9b770222c6729a1925dc size: 1373
```

4. 기존이미지 삭제 후 설치 확인<br>
docker rmi jupyterlab:1.0<br>
docker rmi bong9431/jupyterlab:1.0<br>
**docker run -d -p 8888:8888 --name myjupyter -v /work:/jupyterdir bong9431/jupyterlab:1.0**<br>
```
F:\docker\dockerfile>docker image rm jupyterlab:1.0
Untagged: jupyterlab:1.0

F:\docker\dockerfile>docker image rm bong9431/jupyterlab:1.0
Untagged: bong9431/jupyterlab:1.0
Untagged: bong9431/jupyterlab@sha256:c87e6c9d9a7288dca5e14ee31880e690bc58da4e7b4d9b770222c6729a1925dc
Deleted: sha256:8bcd5b3b53b1dc7bb16db42e48df743736932a484bb9b9e8aa408d5e3eff9d27
```
```
F:\docker\dockerfile>docker run -d -p 8888:8888 --name myjupyter -v /work:/jupyterdir bong9431/jupyterlab:1.0
Unable to find image 'bong9431/jupyterlab:1.0' locally
1.0: Pulling from bong9431/jupyterlab
37aaf24cf781: Already exists
0cfbe9cf08d5: Already exists
26effc8586e1: Already exists
bf8919e76766: Already exists
56e635cd6dbb: Already exists
Digest: sha256:c87e6c9d9a7288dca5e14ee31880e690bc58da4e7b4d9b770222c6729a1925dc
Status: Downloaded newer image for bong9431/jupyterlab:1.0
e2cb797a2587ab78eecab70372007a917abc88703cf12b944b0f9d76a2a00edf

F:\docker\dockerfile>docker ps
CONTAINER ID   IMAGE                     COMMAND                   CREATED         STATUS         PORTS
   NAMES
e2cb797a2587   bong9431/jupyterlab:1.0   "/bin/sh -c 'jupyter…"   6 seconds ago   Up 5 seconds   0.0.0.0:8888->8888/tcp   myjupyter
```
## 4. compose로 실행
- 여러개의 컨테이너를 한꺼번에 순서대로 실행하는 방법임
- compose.yml 파일을 만들고, docker compose로 실행
- compose.yml reference는 [여기](https://docs.docker.com/compose/compose-file/) 참조
```
# 디렉토리에 있는 compose.yml 실행함.
docker compose up -d (백그라운드로 구동)
docker compose down

docker compose -p <prefix-name> up -d
docker compose -p <prefix-name>  down

# 파일명이 compose.yml 아닐때
docker compose -p <prefix-name>  -f ./docker-compose.yml up -d
docker compose -p <prefix-name>  -f ./docker-compose.yml down
```

### 예시 : joomla 실행
1. joomla-compose.yml 만든다.<br>
```
version: '3.1'

services:
  joomla:
    image: joomla:4.3
    restart: always
    depends_on:
      - joomladb
    links:
      - joomladb:mysql
    ports:
      - 80:80
    environment:
      JOOMLA_DB_HOST: mysql
      JOOMLA_DB_PASSWORD: password
    volumes:
      - /home/joomla_web:/var/www/html

  joomladb:
    image: mysql:5.6
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: password
    volumes:
      - /home/joomla_db:/var/lib/mysql
```
2. 실행<br>
docker compose -p wp -f ./joomla-compose.yml up -d<br>
```
F:\docker\joomla>docker compose -p wp -f ./joomla-compose.yml up -d
[+] Running 34/2
 ✔ joomla 21 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                              38.0s
 ✔ joomladb 11 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                      39.1s
[+] Building 0.0s (0/0)                                                                                                                docker:default
[+] Running 3/3
 ✔ Network wp_default       Created                                                                                                              0.1s
 ✔ Container wp-joomladb-1  Started                                                                                                              0.3s
 ✔ Container wp-joomla-1    Started                                                                                                              0.2s

F:\docker\joomla>docker ps
CONTAINER ID   IMAGE        COMMAND                   CREATED          STATUS          PORTS                NAMES
9aab929d5607   joomla:4.3   "/entrypoint.sh apac…"   23 seconds ago   Up 20 seconds   0.0.0.0:80->80/tcp   wp-joomla-1
87e2387dcb95   mysql:5.6    "docker-entrypoint.s…"   23 seconds ago   Up 21 seconds   3306/tcp             wp-joomladb-1
```
3. 종료<br>
docker compose -p wp -f ./joomla-compose.yml down<br>
```
F:\docker\joomla>docker compose -p wp -f ./joomla-compose.yml down
[+] Running 3/3
 ✔ Container wp-joomla-1    Removed                                                                                                              4.6s
 ✔ Container wp-joomladb-1  Removed                                                                                                              3.0s
 ✔ Network wp_default       Removed                                                                                                              0.3s

F:\docker\joomla>docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

F:\docker\joomla>docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

### 예시 : drupal 실행 .yml 파일
```
version: '3.0'
services:
  postgres-db:
    image: postgres:11
    environment:
      - POSTGRES_DB=drupal
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - /home/drupal_db:/var/lib/postgresql/data
  web:
    depends_on:
      - postgres-db
    links:
      - postgres-db:postgres
    image: drupal:9
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles
      - drupal-sites:/var/www/html/sites
      - drupal-themes:/var/www/html/themes
    ports:
      - "80:80"
volumes:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:
```

### 예시 : elasticsearch + kibana + jupyter 
- [es-compose.yml](https://github.com/kobongsoo/docker/blob/master/compose/es-compose.yml)
- elsticsearch compose 내용은 [여기](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/docker.html) 참조

### 예시 : elsticsearch + kibana + embed
- [es-embed-compose.yml](https://github.com/kobongsoo/docker/blob/master/compose/es-embed-compose.yml)
- 문장임베딩 후 chat 창을 통해 검색하고, LLM(gpt, bard) 연동하여 응답 받는 예제.
- es-embed-compose.yml 실행전, 실행 위치에 ./data/[settings.yml](https://github.com/kobongsoo/docker/blob/master/compose/data/settings.yaml) 파일 필요함. 해당 settings.yml 파일은 ES IP 등 환경에 따라 변경 필요.
- 구동 후에는 {ip}/doc 입력해서 elasticsearch에 임베딩 할수 있음.
- {ip}/chat 입력해서 채팅할 수 있음.
![image](https://github.com/kobongsoo/docker/assets/93692701/4af7109a-32ea-47ff-845a-7949bc1d1649)


## 에러 
- 소켓 bind 에러 => **netcfg -d** 실행
<br> Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:9200 -> 0.0.0.0:0: listen tcp 0.0.0.0:92
- docker daemon running? 에러 => root 권한으로 Docker 데몬 실행해줌.
<br>Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```
# Docker 데몬 확인:먼저 Docker 데몬이 실행 중인지 확인하십시오. 다음 명령어를 사용하여 Docker 데몬 상태를 확인합니다:
sudo systemctl status docker

# 결과에서 "Active" 항목이 "active (running)"으로 나와야 합니다. Docker 데몬이 실행 중이 아니면 다음 명령어로 Docker를 시작하십시오:
sudo systemctl start docker

# 사용자를 Docker 그룹에 추가:
# Docker 데몬에 액세스하기 위해서는 사용자가 Docker 그룹에 속해야 합니다. 사용자를 Docker 그룹에 추가하려면 다음 명령어를 사용합니다:
sudo usermod -aG docker $USER

#사용자를 Docker 그룹에 추가한 후 로그아웃하고 다시 로그인하여 변경 사항을 적용하십시오.
```
```



