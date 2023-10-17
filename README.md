# docker
## 1. docker 설치
- 윈도우 os : WSL2 기반으로 [Docker desktop](https://docs.docker.com/desktop/install/windows-install/) 다운로드 해서 설치
- 리눅스 :  get.docker.com 스크립트 이용, root 권한으로 전환( sudo su - root) 후 아래 순서대로 실행.
1. install-docker.sh 파일 다운로드 : curl -fsSL https://get.docker.com -o install-docker.sh   
2. install-docker.sh 파일 확인 (pass해도 됨) : cat install-docker.sh                                   
3. install-docker.sh 테스트로 실행과정 진행해봄(설치는 안됨)(pass해도 됨): sh install-docker.sh --dry-run 
4. install-docker.sh 실제 설치: sh install-docker.sh

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
**jupyter notebook --ip=0.0.0.0 --port=8888 --allow-root --notebook-dir=/notebooks &** <br>
ctrl+P, Q<br>
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
RUN pip3 install jupyterlab
RUN mkdir /jupyterdir
VOLUME /jupyterdir
CMD jupyter lab --ip=0.0.0.0 --port=8888 --allow-root --notebook-dir=/jupyterdir
```

2. Dockerfile 빌드 / 실행<br>
**docker build -t jupyterlab:1.0 -f Dockerfile .**
<br>**docker run -it --name jupyter jupyterlab:1.0**
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
- [es-compose.yml](https://github.com/kobongsoo/docker/blob/master/es-compose.yml)
- elsticsearch compose 내용은 [여기](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/docker.html) 참조

## 에러 
- 소켓 bind 에러 => **netcfg -d** 실행
<br> Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:9200 -> 0.0.0.0:0: listen tcp 0.0.0.0:92

## 명령어
- docker 버전 확인
```
docker ps -a
```
- docker 실행 확인
```
docker ps
docker ps -a
```
- docker 종료
```
docker rm -f <name>
docker rm -f $(docker ps -a -q)
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



