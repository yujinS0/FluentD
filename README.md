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
  - [ ] 기간 내 매칭 성사 수 (매칭 요청 중 성사된 것의 개수)
  - [ ] 기간 내 게임 플레이 수 (정상적으로 끝난 game 수)

  
  ---

 
 ### 로그 모델 정의 
 
1. 로그인 성공 시 (Daily Active Users)
 
 - 로그인 성공 여부는 Daily Active Users와 로그인 횟수 통계에 영향
 - 필드
    + action : login_success
    + playerId
 - 예시
    ```json
    {
        "timestamp":"2024-10-17T15:58:55.4085597+09:00",
        "tag":"Action.Login",
        "context":{
            "action":"login_success",
            "playerId":"test1@gmail.com"
        }
    }
    ```
  
2. 매칭 요청 성공 시 (Time-Based Statistics 기간 내 매칭 요청 수)

 - 필드
    + action : match_request
    + playerId
 - 예시
    ```json
    {
        "timestamp":"2024-10-17T15:58:55.4085597+09:00",
        "tag":"Action.RequestMatching",
        "context":{
            "action":"match_request",
            "playerId":"test2@gmail.com"
        }
    }
    ``` 

 3. 매칭 성사 시 (Time-Based Statistics 기간 내 매칭 성사 수)
 
 - 필드
    + action : match_success
    + playerId
 
 - 예시
    ```json
    {
        "timestamp":"2024-10-17T16:02:27.7802798+09:00",
        "tag":"Action.CheckAndInitializeMatch",
        "context":{
            "action":"match_success",
            "playerId":"test2@gmail.com"
        }
    }
    ``` 
 
 4. 게임 종료 시 (Time-Based Statistics 기간 내 게임 플레이 수)

 - 필드
    + action : game_end
    + winnerId
    + loserId

 - 예시
    ```json
    {
        "timestamp":"2024-10-17T16:02:27.7802798+09:00",
        "tag":"Action.UpdateGameResult",
        "context":{
            "action":"game_end",
            "winnerId":"test2@gmail.com",
            "loserId":"test1@gmail.com"
        }
    }
    ``` 
 



 <br>

 ### 로그 저장 테이블 구조 정의

 ```sql
 create database game_logs;

 ```

 #### 로그인 로그 테이블 login_logs

  ```sql
    CREATE TABLE `game_logs`.`login_logs`(
        id BIGINT AUTO_INCREMENT PRIMARY KEY,
        playerId VARCHAR(255) NOT NULL,    -- 플레이어 ID
        timestamp DATETIME NOT NULL        -- 로그인 발생 시간
    );

 ```
 - 로그인 관련 정보를 기록하는 테이블. 
 - Daily Active Users 및 로그인 횟수 계산에 사용됩니다.


 
 #### 매칭 로그 테이블 matching_logs

  ```sql
    CREATE TABLE `game_logs`.`matching_logs`(
        id BIGINT AUTO_INCREMENT PRIMARY KEY,
        playerId VARCHAR(255) NOT NULL,    -- 매칭을 요청한 플레이어 ID
        timestamp DATETIME NOT NULL,            -- 매칭 요청 또는 성사 시각
        tag VARCHAR(50) NOT NULL           -- 'match_request' 또는 'match_success' 구분을 위한 태그
    );

 ```
 - 매칭 요청과 성사 정보를 기록하는 테이블. 
 - 매칭 요청 수와 매칭 성사 수 통계에 사용됩니다.



 #### 게임 로그 테이블 play_logs

  ```sql
	CREATE TABLE `game_logs`.`play_logs`(
		id BIGINT AUTO_INCREMENT PRIMARY KEY,
		winnerId VARCHAR(255) NOT NULL,    -- 승자 ID
		loserId VARCHAR(255) NOT NULL,     -- 패자 ID
		timestamp DATETIME NOT NULL        -- 게임 종료 시간
	);

 ```

 - 게임 결과를 기록하는 테이블.
 - 게임 플레이 수 및 게임 승패 통계에 사용됩니다.

---

### 통계 쿼리 예시

#### 1. **Daily Active Users (DAU)**
   - 특정 날짜에 로그인한 고유 플레이어 수를 계산합니다.

```sql
SELECT COUNT(DISTINCT playerId) AS daily_active_users
FROM login_logs
WHERE DATE(timestamp) = CURDATE();
```

- **설명**: `DISTINCT`를 사용하여 중복 로그인을 제외한 고유한 플레이어 수를 구합니다. 날짜는 `CURDATE()`로 오늘의 날짜만 필터링합니다.



#### 2. **User-Specific Statistics**

##### 2.1 **로그인 횟수**
   - 각 플레이어의 로그인 횟수를 계산합니다.

```sql
SELECT playerId, COUNT(*) AS login_count
FROM login_logs
GROUP BY playerId;
```

- **설명**: 각 플레이어의 `playerId`를 기준으로 그룹화한 후, 로그인한 횟수를 계산합니다.

##### 2.2 **게임 플레이 횟수**
   - 각 플레이어의 게임 플레이 횟수를 계산합니다.

```sql
SELECT playerId, COUNT(*) AS play_count
FROM (
    SELECT winnerId AS playerId FROM play_logs
    UNION ALL
    SELECT loserId AS playerId FROM play_logs
) AS combined_players
GROUP BY playerId;
```

- **설명**: `winnerId`와 `loserId` 모두를 하나의 `playerId`로 합친 후, 각 플레이어의 게임 참여 횟수를 계산합니다. **UNION ALL**을 사용하여 승자와 패자를 모두 포함한 후 그룹화합니다.


#### 3. **Time-Based Statistics**

##### 3.1 **기간 내 로그인 인구**
   - 특정 기간 동안 로그인한 고유 플레이어 수를 계산합니다.

```sql
SELECT COUNT(DISTINCT playerId) AS unique_logins
FROM login_logs
WHERE timestamp BETWEEN '2024-10-01' AND '2024-10-31';
```

- **설명**: `DISTINCT`를 사용하여 중복 로그인을 제외한 고유한 플레이어 수를 구합니다. 특정 기간 내에 발생한 로그인을 필터링합니다.

##### 3.2 **기간 내 매칭 요청 수**
   - 특정 기간 동안 발생한 매칭 요청 수를 계산합니다.

```sql
SELECT COUNT(*) AS match_request_count
FROM matching_logs
WHERE tag = 'match_request'
  AND time BETWEEN '2024-10-01' AND '2024-10-31';
```

- **설명**: 매칭 요청 태그가 `"match_request"`인 로그만 필터링하여 매칭 요청 수를 계산합니다.

##### 3.3 **기간 내 매칭 성사 수 (매칭 요청 중 성사된 것의 개수)**
   - 특정 기간 동안 매칭 요청 중 성사된 매칭 수를 계산합니다.

```sql
SELECT COUNT(*) AS match_success_count
FROM matching_logs
WHERE tag = 'match_success'
  AND time BETWEEN '2024-10-01' AND '2024-10-31';
```

- **설명**: 매칭 성사 태그가 `"match_success"`인 로그만 필터링하여 매칭 성사 수를 계산합니다.

##### 3.4 **기간 내 게임 플레이 수 (정상적으로 끝난 게임 수)**
   - 특정 기간 동안 종료된 게임 수를 계산합니다.

```sql
SELECT COUNT(*) AS game_play_count
FROM play_logs
WHERE timestamp BETWEEN '2024-10-01' AND '2024-10-31';
```

- **설명**: `play_logs` 테이블에서 게임 종료 시간이 특정 기간 내에 있는 게임의 개수를 계산합니다.



<br>
<br>
<br>

 ---

<br>


fluentd 설정 파일 예시
```xml
<source>
	@type forward
	port 24224
</source>

<match app.log>
	@type mysql
	host 127.0.0.1
	port 3306
	database your_database
	username your_username
	password your_password
	table user_activity_logs
	column_mapping timestamp:timestamp, log_data:log
</match>
```


----

# Feedback

## ZLogger 활용

- `BaseController` & `LoggerHelper` 
   + 2개의 클래스가 동일한 목적 `ActionLog()` 을 수행하고 있음
   + 따라서 BaseController를 제거하고 동일하게 전역에서 LoggerHelper를 이용하도록 수정

## FluentD 활용

- Buffer 사용 권장
   + Buffer 플러그인은 Fluentd output plugin 에서 사용되는 pluggable한 기능
   + 로그 유실 방지를 위해 쓰이기 때문에 현재 사용되는 output plugin 에 적용해 보는 것 추천
   + Buffer 는 메모리 형태와 파일 형태 두가지로 제공되며, 메모리 버퍼는 fluentd 종료 시 유실 되기 때문에 파일 형태가 추천
   + 설정 예시 (참고자료 : https://docs.fluentd.org/buffer/file )
       ```xml
    <match pattern>
        ...
        <buffer>
        @type file
        path /var/log/fluent/buf
        </buffer>
    </match>
       ```


- Secondary 사용 권장
   + Secondary 플러그인은 전송에 실패한 로그를 지정한 경로에 저장함
   + 로그 유실 방지 및 전송 실패 원인 분석을 위해 사용 권장


- Fluentd 역할 분리 구성 권장
   + 서버와 데이터베이스가 주로 다른위치에 있기 때문에, 
     효율적으로 부하를 분산 시키기 위해 전송용 Fluentd(Forwarder), 가공 및 저장용 Fluentd(Aggregator)가 분리되어 구성되는 것이 권장
     + Forwarder : 전송용 Fluentd
     + Aggregator : 가공 및 저장용 Fluentd

- @include 사용 권장
   + 데이터베이스 저장시에, 테이블별 DB 설정이 반복되는 것을 방지하기 위해 conf 파일을 분리하여 @include 사용 권장



## Project & Structure

- UTF-8 Encoding 사용
