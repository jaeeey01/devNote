# docker 명령어

### help
docker --help <br/>
docker ps --help

### docker 이미지 빌드
docker build . <br/>
-t 이름:태그 조합 생성 가능

### 이미지를 기반으로 새 컨테이너 생성 및 실행
#### 이미지명을 다 쓰지 않아도 고유하다면 fullname을 안써도 동작함

docker run[이미지명] <br/>
#default : attached mode <br/>
-d detached mode 설정 <br/>
-p 로컬포트:컨테이너 노출 포트 <br/>
-i 인터렉티브 모드 (표준 입력을 열린 상태로 유지, attach mode가 아닌 경우에도 입력가능) <br/>
-t 터미널 생성 <br/>
--rm 컨테이너가 중지될 때 자동으로 해당컨테이너 제거 <br/>
--name 컨테이너 이름설정

### detached mode 로 분리된 컨테이너 다시 연결시 <br/> 컨테이너를 다시 시작하지 않고도 attach mode 로 연결 가능
docker attach [컨테이너명]

### 해당 컨테이너에 의해 출력된 과거 로그 확인 가능
docker logs [컨테이너명]
-f  follow 옵션

### 모든 도커 컨테이너 조회
docker ps -a

### 현재 실행중인 컨테이너 조회
docker ps

### 실행 중인 컨테이너(밖으)로 파일 또는 폴더를 복사
#### 컨테이너를 다시시작하지 않거나 이미지를 다시 작성하지않고도 컨테이너에 추가 가능
docker cp [복사 대상 폴더 or 파일] [복사 위치] <br/>
ex)<br/> 
docker cp [복사하려는 폴더 or 파일] 복사하려는 컨테이너명:/내부경로<br/>
docker cp 복사하려는 컨테이너명:/내부경로 [복사하려는 폴더 or 파일]


### 이미지 삭제
#### 이미지가 포함된 컨테이너 먼저 제거 후 이미지 제거가능
docker rmi [이미지명] [이미지명] ...

#### 사용되지 않는 모든 이미지 삭제
docker image prune<br/>
-a 태그된 이미지 포함하여 모든 이미지 삭제

### 이미지 정보표시
docker image inspect [이미지명]

### 컨테이너 삭제(실행중인 컨테이너는 삭제불가)
docker rm [컨테이너명] [컨테이너명] ....

### 중지된 컨테이너 실행
#### default : detached mode  
docker start [컨테이너 ID or name] <br/>
-a attach mode로 실행가능(출력모드) <br/>
-i 인터렉티브 모드(입력모드) 

### 컨테이너 정지
docker stop [컨테이너명]

### docker 허브 로그인 
docker login -u [계정명]

### docker 허브 로그아웃
docker logout

### 이미지 태그, 이미지명 변경
docker tag 기존의 이미지명:[기존의 태그명] 새로운 이미지명:[새로운 태그명] <br/>

### 도커 허브에 이미지 푸쉬
#### 태그명 latest 일경우 태그명 생략 가능 <br/>
docker push [도커허브 계정명]/[도커허브 레포지토리:태그명]

### 도커허브에서 이미지 가져오기
docker pull [도커허브 계정명]/[도커허브 레포지토리:태그명]
####  도커허브 이미지를 pull 하지 않고 run을 먼저 실행 할 수도 있음. <br/> 
* run을 실행하게되면 로컬에서 이미지를 찾지 못하면 이미지 이름이 사용한 컨테이너 히스토리에 자동으로 접근함.(도커허브) 
* 이미지를 찾으면 그 이미지를 사용하여 자동으로 풀링함
* 업데이트를 위해 자동으로 체크하는 것이 중요!! 
* 이전에 풀링하여 이미지가 로컬에 있는 경우 이미지를 기반으로 컨테이너를 다시 실행하면 docker run은 그 이미지의 최신버전이 로컬에 있는지 체크하지 않음
* = 그동안 이미지 업데이트를 하여 도커허브에 푸쉬하였다면 docker run으로는 최신 이미지를 업데이트 받지 못함 
* *** 최신 이미지를 받기 위해 수동으로 docker pull과 이미지 이름을 입력하여 실행해야함!! 
### ec2 도커 시작
sudo service docker start

### 도커 안꺼지게 하기
sudo systemctl enable docker

### 이미지플랫폼 관련에러
#### 에러명 : The requested image's platform (linux/arm64/v8) does not match the detected host platform (linux/amd64) and no specific platform was requested
#### = 빌드하는 노트북이 Apple M1칩인데 도커가 해당 이미지를 빌드할때 생성된 빌드 플랫폼이 ec2서버와 m1 맥북간의 호환성이 안맞는 문제
docker build --platform amd64 --build-arg DEPENDENCY=build/dependency -t 도커허브아이디/도커허브 레포지토리명 .