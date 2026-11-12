# Codyssey Workstation

> 터미널 · Docker · Git/GitHub로 “누구나 같은 방식으로 실행할 수 있는” 개발 워크스테이션을 직접 구축하고, 그 과정을 로그와 스크린샷으로 검증한 기록입니다.
> 

---

## 1. 프로젝트 개요

- 리눅스 CLI로 작업 디렉토리를 구성하고 파일 권한을 실습한다.
- Docker를 설치·점검하고 컨테이너를 실행·관리한다.
- `nginx:alpine` 기반 Dockerfile로 커스텀 웹 서버 이미지를 만들고 포트 매핑으로 접속을 확인한다.
- 바인드 마운트로 “변경 반영”, Docker 볼륨으로 “데이터 영속성”을 검증한다.
- Git을 설정하고 VSCode에서 GitHub와 연동해 저장소를 올린다.

---

## 2. 실행 환경

| 항목 | 내용 |
| --- | --- |
| OS | Windows 11 + WSL2 (Ubuntu) |
| Shell | bash |
| Terminal | Ubuntu 터미널 (WSL), VSCode 통합 터미널 |
| Docker | Docker Desktop (WSL 2 backend) / Docker version 29.9.3 |
| Git | git version 2.53.0 |
| Editor | Visual Studio Code + WSL 확장 |

> ⚠️ **환경 관련 주의사항**
> 
> - 본 과제는 Windows + WSL2 환경에서 수행했다. 모든 작업은 Windows 드라이브(`/mnt/c/...`)가 아닌 **리눅스 홈(`~`) 아래**에서 진행했다. Windows 드라이브에서는 `chmod`가 정상 반영되지 않기 때문이다.

---

## 3. 수행 체크리스트

- [x]  터미널 기본 조작 (위치 확인, 목록, 이동, 생성, 복사, 이동/이름변경, 삭제, 내용 확인, 빈 파일)
- [x]  권한 변경 실습 (파일 1개, 디렉토리 1개)
- [x]  Docker 설치 및 점검 (`docker --version`, `docker info`)
- [x]  Docker 기본 운영 명령 (`images`, `ps`, `logs`, `stats`, `stop`)
- [x]  hello-world 실행
- [x]  ubuntu 컨테이너 진입 및 attach / exec 비교
- [x]  Dockerfile 기반 커스텀 이미지 빌드·실행
- [x]  포트 매핑 접속 (8080, 8081 / curl + 브라우저)
- [x]  바인드 마운트 변경 반영
- [x]  Docker 볼륨 영속성
- [x]  Git 설정 + VSCode GitHub 연동

---

## 4. 저장소 구조

```
workstation/
├── README.md
├── images/              # 스크린샷 증거
├── my-web/
│   ├── Dockerfile       # 커스텀 웹 서버 이미지
│   └── site/
│       └── index.html   # 정적 웹 페이지
├── practice/            # 터미널 실습용 폴더
│   └── greeting.txt
├── empty.txt            # touch로 만든 빈 파일 (권한 실습 대상)
└── hello.txt
```

---

## 5. 수행 로그

### 5-1. 터미널 기본 조작

**현재 위치 확인 → 홈으로 이동 → 숨김 파일 포함 목록 확인**

```bash
$ pwd
/mnt/c/WINDOWS/system32
$ cd ~
$ pwd
/home/eun
$ ls -la
total 28
drwxr-x--- 4 eun  eun  4096 Nov  9 14:00 .
drwxr-xr-x 3 root root 4096 Nov  9 14:00 ..
-rw-r--r-- 1 eun  eun   220 Nov  9 14:00 .bash_logout
-rw-r--r-- 1 eun  eun  3771 Nov  9 14:00 .bashrc
drwxr-xr-x 4 eun  eun  4096 Nov  9 14:00 .cache
drwxr-xr-x 3 eun  eun  4096 Nov  9 14:00 .config
-rw-r--r-- 1 eun  eun     0 Nov  9 14:00 .motd_shown
-rw-r--r-- 1 eun  eun   807 Nov  9 14:00 .profile
```

**작업 디렉토리 생성 및 이동**

```bash
$ mkdir -p ~/codyssey/workstation
$ cd ~/codyssey/workstation
$ pwd
/home/eun/codyssey/workstation
```

**빈 파일 생성 / 파일 작성 및 내용 확인**

```bash
$ touch empty.txt
$ echo "hello codyssey" > hello.txt
$ cat hello.txt
hello codyssey
```

**디렉토리 생성 및 복사**

```bash
$ mkdir practice
$ cp hello.txt practice/
$ ls -la practice
total 12
drwxr-xr-x 2 eun eun 4096 Nov  9 14:05 .
drwxr-xr-x 3 eun eun 4096 Nov  9 14:05 ..
-rw-r--r-- 1 eun eun   15 Nov  9 14:05 hello.txt
```

**이름 변경 → 다른 폴더로 이동**

```bash
$ mv hello.txt greeting.txt
$ ls -la
-rw-r--r-- 1 eun eun    0 Nov  9 14:04 empty.txt
-rw-r--r-- 1 eun eun   15 Nov  9 14:04 greeting.txt
drwxr-xr-x 2 eun eun 4096 Nov  9 14:05 practice

$ mv greeting.txt practice/
$ ls -la practice
-rw-r--r-- 1 eun eun   15 Nov  9 14:04 greeting.txt
-rw-r--r-- 1 eun eun   15 Nov  9 14:05 hello.txt
```

**삭제**

```bash
$ rm practice/hello.txt
$ ls -la practice
-rw-r--r-- 1 eun eun   15 Nov  9 14:04 greeting.txt
```

**상대 경로로 이동했다가 상위 폴더로 복귀**

```bash
$ cd practice
$ pwd
/home/eun/codyssey/workstation/practice
$ cd ..
$ pwd
/home/eun/codyssey/workstation
```

📷 증거: 작업 폴더 준비 · 터미널 조작 1 · 터미널 조작 2

---

### 5-2. 권한 실습

**파일: 644 → 755**

```bash
$ ls -l empty.txt
-rw-r--r-- 1 eun eun 0 Nov 11 10:28 empty.txt
$ chmod 755 empty.txt
$ ls -l empty.txt
-rwxr-xr-x 1 eun eun 0 Nov 11 10:28 empty.txt
```

**디렉토리: 755 → 700**

```bash
$ ls -ld practice
drwxr-xr-x 2 eun eun 4096 Nov  9 14:07 practice
$ chmod 700 practice
$ ls -ld practice
drwx------ 2 eun eun 4096 Nov  9 14:07 practice
```

| 대상 | 변경 전 | 변경 후 | 의미 |
| --- | --- | --- | --- |
| `empty.txt` | `rw-r--r--` (644) | `rwxr-xr-x` (755) | 모든 사용자에게 실행 권한 추가 |
| `practice/` | `rwxr-xr-x` (755) | `rwx------` (700) | 소유자만 읽기·쓰기·진입 가능 |

📷 증거: 파일 권한 · 디렉토리 권한

---

### 5-3. Docker 설치 및 점검

```bash
$ docker --version
Docker version 29.9.3, build 5457199

$ docker info
Client:
 Version:    29.9.3
 Context:    default
 ...
Server:
  Containers: 7
  Running: 4
  Paused: 0
  Stopped: 3
 Images: 4
 Server Version: 29.9.3
 Storage Driver: overlayfs
  driver-type: io.containerd.snapshotter.v1
 Logging Driver: local
 Cgroup Driver: cgroupfs
```

- `Server:` 섹션이 출력되면 Docker 데몬이 정상 동작 중임을 의미한다.

📷 증거: docker version · docker info Server

---

### 5-4. 컨테이너 실행 실습

**hello-world**

```bash
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
...
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

**이미지 / 컨테이너 목록**

```bash
$ docker images
IMAGE                ID             DISK USAGE   CONTENT SIZE
hello-world:latest   cf1b20a0c89a   25.9kB       9.49kB

$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      NAMES
7f9db7b862bd   hello-world   "/hello"   52 seconds ago   Exited (0) 52 seconds ago   elastic_noether
```

**ubuntu 컨테이너 진입 후 명령 실행**

```bash
$ docker run -it --name my-ubuntu ubuntu bash
root@0bf5bb0325b8:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@0bf5bb0325b8:/# echo "hello from container"
hello from container
root@0bf5bb0325b8:/# exit
```

📷 증거: hello-world · images / ps -a · ubuntu 컨테이너

---

### 5-5. attach vs exec 비교

```bash
$ docker start my-ubuntu
$ docker ps
CONTAINER ID   IMAGE    COMMAND   STATUS         NAMES
0bf5bb0325b8   ubuntu   "bash"    Up 4 seconds   my-ubuntu

# exec: 새 bash 프로세스로 접속 후 exit
$ docker exec -it my-ubuntu bash
root@0bf5bb0325b8:/# exit
$ docker ps
CONTAINER ID   IMAGE    COMMAND   STATUS          NAMES
0bf5bb0325b8   ubuntu   "bash"    Up 29 seconds   my-ubuntu      ← 계속 실행 중

# attach: 메인 프로세스에 직접 연결 후 exit
$ docker attach my-ubuntu
root@0bf5bb0325b8:/# exit
$ docker ps -a
CONTAINER ID   IMAGE    COMMAND   STATUS                    NAMES
0bf5bb0325b8   ubuntu   "bash"    Exited (0) 5 seconds ago  my-ubuntu      ← 종료됨
```

**관찰 결과**

- `exec`는 실행 중인 컨테이너에 **새 프로세스를 추가로** 띄워 접속한다. 그 프로세스를 exit해도 메인 프로세스는 살아 있으므로 컨테이너가 유지된다.
- `attach`는 컨테이너의 **메인 프로세스(bash)에 직접** 연결한다. exit하면 메인 프로세스가 끝나므로 컨테이너도 종료된다.
- attach 상태에서 컨테이너를 끄지 않고 빠져나오려면 `Ctrl+P`, `Ctrl+Q`를 사용한다.

📷 증거: attach vs exec

---

### 5-6. Docker 운영 명령 (logs / stats / stop)

```bash
$ docker run -d --name test-nginx nginx
$ docker logs test-nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
...
/docker-entrypoint.sh: Configuration complete; ready for start up
2026/11/12 04:50:14 [notice] 1#1: using the "epoll" event method
2026/11/12 04:50:14 [notice] 1#1: nginx/1.31.8
2026/11/12 04:50:14 [notice] 1#1: start worker processes
...
2026/11/12 04:56:28 [notice] 1#1: signal 3 (SIGQUIT) received, shutting down
...
- 로그에는 nginx 시작 과정과, 'docker stop' 시 SIGQUIT 신호를 받고 종료된 기록이 남아 있다. (로그 시간은 컨테이너 기준 UTC)

$ docker stats --no-stream
CONTAINER ID   NAME         CPU %   MEM USAGE / LIMIT     MEM %   NET I/O          BLOCK I/O     PIDS
d37c5f8fbbb17  test-nginx   0.00%   13.74MiB / 15.37GiB   0.09%   1.17kB / 126B    0B / 12.3kB   17

$ docker stop test-nginx
test-nginx
$ docker ps -a
CONTAINER ID   IMAGE   COMMAND                  STATUS                    NAMES
d37c5f8fbbb17  nginx   "/docker-entrypoint.…"   Exited (0) 5 seconds ago  test-nginx
```

📷 증거: logs · stats / stop

---

### 5-7. Dockerfile 기반 커스텀 이미지

**선택한 방식:** (A) 웹 서버 베이스 이미지 활용 + 정적 콘텐츠 교체
**선택한 베이스:** `nginx:alpine` (공식 NGINX 이미지의 경량 Alpine 버전)

`my-web/Dockerfile`

```docker
FROM nginx:alpine
LABEL org.opencontainers.image.title="eun-custom-web"
ENV APP_ENV=dev
COPY site/ /usr/share/nginx/html/
```

| 커스텀 포인트 | 목적 |
| --- | --- |
| `FROM nginx:alpine` | 검증된 웹 서버를 바탕으로 시작해 직접 설치할 필요를 없애고, 가벼운 이미지를 사용 |
| `LABEL ...title` | 이미지에 이름 메타데이터를 붙여 식별·관리를 쉽게 함 |
| `ENV APP_ENV=dev` | 실행 환경을 환경변수로 표시 (설정과 코드 분리) |
| `COPY site/ ...` | NGINX 기본 페이지를 직접 작성한 정적 페이지로 교체 |

**빌드**

```bash
$ cd ~/codyssey/workstation/my-web
$ docker build -t my-web:1.0 .
[+] Building 7.0s (7/7) FINISHED
 => [1/2] FROM docker.io/library/nginx:alpine@sha256:713fc2...
 => [2/2] COPY site/ /usr/share/nginx/html/
 => => naming to docker.io/library/my-web:1.0

$ docker images
IMAGE                ID             DISK USAGE   CONTENT SIZE
my-web:1.0           01a8b778bae7   93.7MB       26.4MB
```

📷 증거: docker build

---

### 5-8. 포트 매핑 접속

```bash
$ docker run -d -p 8080:80 --name my-web-8080 my-web:1.0
$ curl http://localhost:8080
<h1>Hello! This is eun's customweb server</h1>

$ docker run -d -p 8081:80 --name my-web-8081 my-web:1.0
$ curl http://localhost:8081
<h1>Hello! This is eun's customweb server</h1>

$ docker ps
CONTAINER ID   IMAGE        STATUS         PORTS                                     NAMES
0d3caf3cd8a5   my-web:1.0   Up 11 seconds  0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   my-web-8081
bfa3ac239a14   my-web:1.0   Up 2 minutes   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   my-web-8080
```

- 같은 이미지로 컨테이너 2개를 띄워도 동일하게 동작한다. 컨테이너 내부 포트는 둘 다 80이지만, 호스트 포트(8080/8081)가 달라 충돌하지 않는다.

**브라우저 접속 (주소창 포함)**

브라우저 접속

!브라우저 접속

브라우저 접속

📷 증거: 8080 실행 + curl · 8081 실행 + docker ps

---

### 5-9. 바인드 마운트 변경 반영

```bash
$ docker run -d -p 8082:80 --name my-web-bind \
    -v $(pwd)/site:/usr/share/nginx/html my-web:1.0

# 변경 전
$ curl http://localhost:8082
<h1>Hello! This is eun's customweb server</h1>

# 호스트에서 파일 수정
$ echo "<h1>Bind mount test: changed from host!</h1>" > site/index.html

# 변경 후 (재빌드 없음)
$ curl http://localhost:8082
<h1>Bind mount test: changed from host!</h1>

# 비교: 바인드 마운트 없이 COPY된 이미지로 실행한 컨테이너
$ curl http://localhost:8080
<h1>Hello! This is eun's customweb server</h1>
```

| 컨테이너 | 방식 | 호스트 파일 수정 후 |
| --- | --- | --- |
| `my-web-bind` (8082) | 바인드 마운트 | 즉시 반영됨 |
| `my-web-8080` (8080) | 이미지에 COPY | 변경 없음 |

> 참고: 이 실습으로 `my-web/site/index.html` 내용이 바인드 마운트 테스트 문구로 바뀌었으므로, 지금 다시 빌드하면 바뀐 문구가 이미지에 들어간다.
> 

📷 증거: 바인드 마운트

---

### 5-10. Docker 볼륨 영속성

```bash
# 볼륨 생성 및 확인
$ docker volume create eun-data
eun-data
$ docker volume ls
DRIVER    VOLUME NAME
local     eun-data

# 볼륨을 연결한 컨테이너에서 데이터 작성
$ docker run -d --name vol-test1 -v eun-data:/data ubuntu sleep infinity
$ docker exec vol-test1 bash -c "echo 'data saved by eun' > /data/hello.txt"
$ docker exec vol-test1 cat /data/hello.txt
data saved by eun

# 컨테이너 삭제
$ docker rm -f vol-test1
vol-test1

# 새 컨테이너에 같은 볼륨 연결 후 확인
$ docker run -d --name vol-test2 -v eun-data:/data ubuntu sleep infinity
$ docker exec vol-test2 cat /data/hello.txt
data saved by eun
```

- `vol-test1`을 삭제한 뒤에도 새 컨테이너 `vol-test2`에서 같은 데이터가 확인되어, 볼륨 데이터가 컨테이너 수명과 무관하게 유지됨을 확인했다.

📷 증거: 볼륨 영속성

---

### 5-11. Git 설정 및 GitHub 연동

```bash
$ git --version
git version 2.53.0

$ git config --global user.name "eun"
$ git config --global user.email "355717054+gx2un@users.noreply.github.com"
$ git config --global init.defaultBranch main

$ git config --list
user.name=eun
user.email=355717054+gx2un@users.noreply.github.com
init.defaultbranch=main
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
remote.origin.url=https://github.com/gx2un/workstation.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.main.remote=origin
branch.main.merge=refs/heads/main
```

- 이메일은 개인 메일 노출을 막기 위해 GitHub에서 제공하는 **noreply 주소**를 사용했다.

**첫 커밋 및 업로드**

```bash
$ git init
$ git add .
$ git commit -m "Initial commit: terminal, docker practice files"
[main (root-commit) e65c675] Initial commit: terminal, docker practice files
 6 files changed, 8 insertions(+)
$ git log --oneline
e65c675 (HEAD -> main) Initial commit: terminal, docker practice files
```

- 이후 VSCode Source Control의 **Publish Branch**로 GitHub public 저장소(`gx2un/workstation`)에 업로드했다.
- 첫 커밋에 개인 이메일이 기록되어 있어 작성자 정보를 수정했다. (트러블슈팅 6 참고)

```bash
$ git commit --amend --reset-author --no-edit
$ git log --format="%h %an %ae"
e610709 eun 355717054+gx2un@users.noreply.github.com
$ git push --force
To https://github.com/gx2un/workstation.git
 + e65c675...e610709 main -> main (forced update)
```

📷 증거: git config · VSCode GitHub 로그인 · VSCode WSL 연결

---

## 6. 검증 방법 요약

| 항목 | 검증 명령 | 확인 내용 |
| --- | --- | --- |
| 작업 위치 | `pwd` | 현재 절대 경로 |
| 권한 | `ls -l`, `ls -ld` | 변경 전/후 권한 문자열 |
| Docker 설치 | `docker --version` | 버전 출력 |
| Docker 데몬 | `docker info` | `Server:` 섹션 출력 |
| 컨테이너 상태 | `docker ps`, `docker ps -a` | Up / Exited 상태 |
| 이미지 빌드 | `docker images` | `my-web:1.0` 존재 |
| 포트 매핑 | `curl localhost:8080`, 브라우저 | 직접 작성한 HTML 응답 |
| 바인드 마운트 | `curl localhost:8082` (수정 전/후) | 재빌드 없이 내용 변경 |
| 볼륨 | 컨테이너 삭제 후 `cat /data/hello.txt` | 데이터 유지 |
| Git | `git config --list`, `git log` | 설정값, 커밋 기록 |

---

## 7. 재현 방법

```bash
git clone https://github.com/gx2un/workstation.git
cd workstation/my-web
docker build -t my-web:1.0 .
docker run -d -p 8080:80 --name my-web-8080 my-web:1.0
curl http://localhost:8080
```

- 8080 포트가 이미 사용 중이면 `p 8090:80`처럼 다른 호스트 포트를 사용한다.
- 바인드 마운트의 `$(pwd)`는 명령을 실행하는 위치 기준이므로 반드시 `my-web/` 폴더 안에서 실행한다.

---

## 8. 트러블슈팅

### 1) `wsl --install` 실행 시 403 에러

- **문제:** 관리자 PowerShell에서 `wsl --install` 실행 시 `(403)` 에러가 발생하며 설치 실패
- **원인 가설:** ① 관리자 권한 부족 ② 설치 파일 다운로드 경로 차단
- **확인:** PowerShell 창 제목에 “관리자”가 표시되어 있어 ①은 아님. 403은 HTTP “접근 거부” 코드이므로 다운로드 단계 문제로 판단
- **해결:** Microsoft Store를 거치지 않고 직접 내려받는 옵션으로 재시도하여 성공
    
    ```powershell
    wsl --install --web-download
    wsl --install -d Ubuntu --web-download
    ```
    
- 📷 403 에러 · web-download 성공

### 2) Ubuntu에서 `code .` 실행 시 `command not found`

- **문제:** VSCode 설치 후 Ubuntu에서 `code .` 입력 시 `Command 'code' not found` 출력
- **원인 가설:** Ubuntu 터미널을 VSCode 설치 **전에** 열어 두어, 새로 추가된 PATH가 반영되지 않음
- **확인:** VSCode 설치 시 PATH 추가 옵션은 기본 체크 상태였음
- **해결:** Ubuntu 창을 모두 닫고 PowerShell에서 `wsl --shutdown` 실행 후 다시 열어 `code .` 실행 → VS Code Server 설치 후 정상 연결
- **주의:** 안내 메시지의 `sudo snap install code`는 리눅스용 VSCode를 별도 설치하는 것이라 사용하지 않음
- 📷 command not found · 재실행 후 성공

### 3) VSCode GitHub 로그인 후 “Another instance of Code is already running as administrator”

- **문제:** 브라우저에서 GitHub 인증 후 VSCode로 돌아올 때 위 에러 발생
- **원인 가설:** 설치 직후 자동 실행된 VSCode가 관리자 권한으로 떠 있어(창 제목 `[Administrator]`), 일반 권한으로 열리려는 인스턴스와 충돌
- **해결:** VSCode 창을 모두 닫고 다시 실행한 뒤 로그인 재시도 → `gx2un (GitHub)` 로그인 확인
- 📷 관리자 인스턴스 충돌

### 4) `docker build` 실패 (`unknown shorthand flag`, `requires 1 argument`)

- **문제:** 커스텀 이미지 빌드 시 연속으로 에러 발생
- **원인 및 확인:**
    - `docker built ...` → `build` 오타로 존재하지 않는 명령
    - `docker build -t my-web:1.0.` → 마지막 `.`가 태그에 붙어 **빌드 컨텍스트(PATH)가 누락**됨. 에러의 `requires 1 argument`가 PATH 누락을 의미
- **해결:** 태그와 경로를 띄어 써서 `docker build -t my-web:1.0 .` 로 실행해 빌드 성공
- 📷 build 오타 · PATH 누락

### 5) 프롬프트가 `>`로 바뀌고 입력 대기 상태가 됨

- **문제:** heredoc(`<<'EOF'`)을 중복 입력하거나, 문자열의 닫는 따옴표를 빠뜨렸을 때 프롬프트가 `>`로 바뀌고 명령이 끝나지 않음
- **원인:** 쉘이 명령의 끝(`EOF` 또는 닫는 `"`)을 기다리는 상태
- **해결:** `Ctrl + C`로 입력을 취소하고 명령을 올바르게 다시 입력

### 6) 커밋 기록에 개인 이메일 노출

- **문제:** 개인 이메일로 설정한 상태에서 첫 커밋을 만들어 GitHub에 push함 → 커밋 메타데이터에 개인 이메일이 공개됨
- **원인:** GitHub 이메일 비공개 설정 전에 `git config user.email`을 개인 메일로 설정함
- **해결:**
    1. GitHub Settings → Emails에서 “Keep my email addresses private” 활성화 후 noreply 주소 확인
    2. `git config --global user.email`을 noreply 주소로 변경
    3. `git commit --amend --reset-author --no-edit`로 커밋 작성자 정보 수정
    4. `git push --force`로 원격 기록을 덮어씀 (개인 저장소이므로 강제 push 사용)
- 📷 작성자 수정 확인 · 강제 push

### 7) 일반 Ubuntu 터미널에서 `git push` 시 인증 실패

- **문제:** `Password authentication is not supported for Git operations` 에러와 함께 push 실패
- **원인:** GitHub는 HTTPS push 시 계정 비밀번호 인증을 지원하지 않음 (토큰 또는 SSH 필요)
- **해결:** GitHub 로그인이 연동된 **VSCode 통합 터미널**에서 push하여 VSCode가 인증을 대신 처리하도록 함
- 📷 비밀번호 인증 실패
- **참고:** VSCode 터미널에서 Backspace가 한 칸씩 밀려 보이는 표시 오류가 있어, 영어 입력 모드로 전환하고 명령을 붙여넣어 해결함

---

## 9. 개념 정리

### 절대 경로 vs 상대 경로

- **절대 경로:** 루트(`/`)부터 전체를 적는 경로. 어디서 실행해도 같은 위치를 가리킨다. 예) `/home/eun/codyssey/workstation/practice`
- **상대 경로:** 현재 위치 기준 경로. 예) 현재 `workstation`에 있을 때 `practice`, 상위 폴더는 `..`

### 파일 권한 (r/w/x, 755, 644)

- `r`(읽기)=4, `w`(쓰기)=2, `x`(실행)=1을 더해 숫자로 표현하며, **소유자 / 그룹 / 기타** 순서로 3자리를 쓴다.
- `755` = `rwx` `r-x` `r-x` → 소유자 전체 권한, 나머지는 읽기·실행
- `644` = `rw-` `r--` `r--` → 소유자 읽기·쓰기, 나머지는 읽기만
- 디렉토리에서 `x`는 “폴더 안으로 진입(`cd`)” 권한을 의미한다.

### 이미지 vs 컨테이너

- **이미지:** 실행에 필요한 파일과 설정을 묶은 읽기 전용 템플릿
- **컨테이너:** 이미지를 실행한 격리된 인스턴스. 하나의 이미지로 여러 컨테이너를 만들 수 있다.

### 포트 매핑이 필요한 이유

- 컨테이너는 격리된 네트워크를 가지므로, 컨테이너 내부 포트(80)에 호스트가 바로 접근할 수 없다.
- `p <호스트포트>:<컨테이너포트>`로 호스트 포트를 컨테이너 포트에 연결해야 외부(브라우저)에서 접속할 수 있다.

### 바인드 마운트 vs Docker 볼륨

|  | 바인드 마운트 | Docker 볼륨 |
| --- | --- | --- |
| 저장 위치 | 사용자가 지정한 호스트 경로 | Docker가 관리하는 저장 공간 |
| 주 용도 | 개발 중 소스 변경 즉시 반영 | 데이터 영속 보관 (DB 등) |
| 컨테이너 삭제 시 | 호스트 파일 유지 | 볼륨 데이터 유지 |

### Git vs GitHub

- **Git:** 로컬에서 파일 변경 이력을 관리하는 **버전 관리 도구**
- **GitHub:** Git 저장소를 원격에 호스팅하고 공유·협업하는 **플랫폼**

---

## 10. 보안 및 개인정보

- Git 커밋 이메일은 GitHub noreply 주소를 사용하고, 이전에 노출된 개인 이메일은 커밋 히스토리에서 제거했다.
- GitHub 인증 코드 등 민감정보는 문서와 스크린샷에 포함하지 않았다.
- 토큰·비밀번호·개인키는 저장소에 포함하지 않았다.
