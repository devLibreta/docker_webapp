# WebApp 설치 가이드

## 파일 구조

	lib/jboss-eap-7.4.0.zip
	dockefile.rhel7jboss
	docker-compose.webapp.yml

## 최종 네트워크 오픈 포트

	JBOSS
	 - 18080 : http 포트. 컨테이너 내부 8080 포트로 연결됨.
	 - 18443 : https 포트. 컨테이너 내부 8443 포트로 연결됨.
	 - 19990 : 관리 콘솔 포트. 컨테이너 내부 9990 포트로 연결됨.
	ORACLE
	 - 11521 : 오라클 리스터 포트. 컨테이너 내부 1521 포트로 연결됨.
	 - 15500 : 오라클 관리 콘솔 포트. 컨테이너 내부 5500 포트로 연결됨.
	POSTGRE
	 - 15432 : postgre 리스너 포트. 컨테이너 내부 5432 포트로 연결됨.
	
## 윈도우 WSL 환경설정

1. Windows 기능 켜기/끄기
	- Hyper-V 관련 체크박스를 모두 선택하기.
2. BIOS에서 CPU의 가상화 기술 활성화하기.
3. 마이크로소프트 스토어에서 WSL 설치.
	- Windows Subsystem for Linux
4. 리눅스 배포판 설치.
	- Ubuntu 22.04.3 LTS
5. cmd 실행
6. WSL 실행 명령어 입력 및 실행 확인.

```
$ wsl
```

## 도커(docker) 설치

1. docker window 파일 다운로드 및 설치.

[도커 공식 홈페이지](https://www.docker.com/)

2. docker 버전 확인

```
$ docker --version
```

## rhel7jboss 환경설정

redhat 리눅스에 jboss 를 설치한 컨테이너 환경설정 가이드.

> JBOSS 설치파일 다운로드

[레드햇 JBOSS EAP 다운로드 페이지](https://developers.redhat.com/products/eap/download)

	1. Zip 파일 다운로드(redhat 계정 로그인 필요).
	2. 파일명.확장자를 dockerfile 환경변수 JBOSS_FILENAME 에 설정.
	3. JBOSS 파일을 dockerfile 과 같은 폴더 내 lib 폴더 내부에 위치.

> RHEL7JBOSS 이미지 빌드

1. wsl 실행 후 dockerfile이 있는 위치로 이동.

```
$ cd /mnt/c/Users/KTMP/Desktop/docker
```

2. dockefile에 있는대로 dockefile 빌드 명령어 실행.

```
$ docker buildx build -t rhel7_image -f dockerfile.rhel7jboss .
```
	docker buildx build -t [image_name] -f [dockerfile_name] .
	 - image_name
		dockerfile로 만들 이미지의 이름. 사용자가 정하지 않으면 랜덤문자열이 된다. 
		이미지 사용 시 입력하기에 이름을 설정하는게 좋다.
	 - dockerfile_name
		docker는 기본적으로 dockerfile 이라는 이름의 도커파일을 실행한다.
		다른 이름을 사용할 경우 해당파일 이름을 입력해야 한다.
	 - . 
		명령어에서 제일 마지막의 "."은 dockerfile이 위치한 경로.
		뺴먹으면 실행이 안된다.

## oracle DB 환경설정

오라클 DB 컨테이너 설치 가이드.

> oracle repository login

1. WSL 에서 오라클 컨테이너 이미지 repository에 로그인 명령어 입력.
```
$ docker login https://container-registry.oracle.com/database/enterprise:latest
```

2. 오라클 웹 계정 정보를 입력.
```
$ Username: [ID]
$ Password: [PASSWORD]
```

## postgre DB 환경설정

docker-compose 파일로 처리하기에 할일 없음.

## WebApp 컨테이너 실행

도커 컴포즈 파일(docker-compose.webapp.yml)을 사용하여 여러 컨테이너로 구성된 webapp 실행 가이드.

> 실행(create&run) 명령어
```
$ docker-compose -f docker-compose.webapp.yml up -d
```
	docker-compose -f [docker_compose_file_name] up -d
	 - docker_compose_file_name
		도커 컴포즈 파일(파일이름.확장자).

> 도커 컨테이너 접속 명령어
```
$ docker exec -it rhel7jboss /bin/bash
```

	docker exec -it [container_name] /bin/bash
	 - container_name
		컨테이너명.
		docker-compose 파일에서 image에 해당하는 값.

> 오라클 DB 설정

sqlplus 접속
```
$ docker exec -it ora21c /bin/bash
$ sqlplus / as sysdba
```

sqlplus 접속 후 SQL
```sql
ALTER SESSION SET "_ORACLE_SCRIPT"=true;
create user [id] identified by [root_pw];
grant dba to [id] with admin option;
alter database set time_zone = 'Asia/Seoul';
```

> jboss 기본페이지 해지

```
$ docker exec -it rhel7jboss /bin/bash
$ cd /opt/jboss-eap-7.4/standalone/configuration
$ vi standalone.xml
```

vi 에디터에서 해당 라인 검색
```xml
<location name="/" handler="welcome-content"/>
```
다음과 같이 주석처리.
```xml
<!--<location name="/" handler="welcome-content"/>-->
```

### 기타 컨테이너 명령어

> 중단(stop) 명령어
```
$ docker-compose -f docker-compose.webapp.yml stop
```
	docker-compose -f [docker_compose_file_name] stop
	 - docker_compose_file_name
		도커 컴포즈 파일(파일이름.확장자).
		
> 재시작(restart) 명령어
```
$ docker-compose -f docker-compose.webapp.yml restart
```
	docker-compose -f [docker_compose_file_name] 	 - - docker_compose_file_name
		도커 컴포즈 파일(파일이름.확장자).

> 정지및삭제(stop&delete) 명령어
```
$ docker-compose -f docker-compose.webapp.yml down
```	
	docker-compose -f [docker_compose_file_name] down
	 - docker_compose_file_name
		도커 컴포즈 파일(파일이름.확장자).
		
