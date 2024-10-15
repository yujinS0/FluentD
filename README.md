# FluentD
FluentD를 통한 Log 처리 학습 및 실습을 위한 레포지토리입니다.

- FluentD 학습 : [Study 디렉토리](./Study/)
- Integrating Fluentd 실습 과제

<br> 

## Integrating Fluentd 실습 과제

(Docker Container Logs 기능을 활용하여,)

게임 서버에서 출력된 로그를 Fluentd로 데이터베이스에 저장합니다.

- 이때 게임 서버는 [오목 게임 서버](https://github.com/yujinS0/Omok-Game) 레포지토리를 사용합니다.


## Requirements

### 0. ZLogger를 통한 JSON 포맷 콘솔 출력

ZLogger를 활용하여 콘솔에 JSON 포맷으로 로그가 출력 되도록 합니다.

### 1. Flunted를 사용하여 로그 전송

컨테이너화된 서버의 콘솔 출력물이 fluented로 전송 되게합니다.

방법은 크게 두가지(a, b)가 있습니다.

#### a. local 환경에서 fluent-package(과거 td-agent)를 활용

- local이나 VM 환경에서 fluentd를 설치하여 특정 경로의 로그를 수집하는 방식
- 도커는 기본적으로 컨테이너 로그를 `/var/lib/docker/containers/<containers-id>/<containers-id>-json.log`에 저장
	
#### b. 컨테이너화한 fluent로 docker container logs 전송

- docker logging driver를 통해 컨테이너화한 fluentd로 로그를 전송하는 방식
- fluentd 설정 파일에서 fluentd-input 플러그인을 사용해 로그를 받을 수 있음
	```xml
	<source>
	  @type forward  # forward input 플러그인 사용
	  port 24224     # Docker logging driver가 전송하는 포트를 지정 (fluentd 컨테이너 실행 시 설정한 포트)
	  bind 0.0.0.0   # 모든 IP로부터 로그 수신
	</source>

	<match docker.*>  # 도커에서 전송된 로그 처리
	  @type stdout    # 로그를 stdout으로 출력할 수도 있고, 필요한 경우 데이터베이스로 전송 가능
	</match>
	```

### 2. Fluentd를 통해 MySQL 데이터베이스에 저장

이후 Fluentd 에서 알맞은 MySQL 데이터 베이스의 테이블로 저장 되도록 합니다.

여기서 핵심은 ZLogger를 통해 사전 포맷팅된 로그를 최소의 프로세싱으로,

알맞은 fluentd 설정을 통해 MySQL DB로 로그가 저장되는것입니다.

<br>

#### 로그로 아래의 모델 통계 가능 해야 합니다

- [ ] Daily Active Users
- [ ] User-Specific Statistics
  - [ ] 로그인 횟수
  - [ ] 게임 플레이 횟수
- [ ] Time-Based Statistics
  - [ ] 기간 내 로그인 인구
  - [ ] 기간 내 매칭 요청 수
  - [ ] 기간 내 매칭 성사 수
  - [ ] 기간 내 게임 플레이 수
