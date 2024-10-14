# 02. 로그 저장 및 패턴 매칭 ( Match Directive )

- 태그 패턴 매칭 학습
- 로그 데이터에 새로운 필드 추가 학습 ( Filter Plugin - record_transformer)
- 태그 패턴에 따라 아웃풋을 다르게 하는 로그 라우팅 학습
- 로그 데이터로 라우팅 하는 방법 학습

### 예제 실습
- [서버 타입에 따라 데이터 저장하는 예제](sample_match_servertype_fluentd.conf)
- [로그 타입(레벨)에 따라 데이터 저장하는 예제](sample_match_loglevel_fluentd.conf)


### 과제
- [Error 로그 별도 저장하기](task_log_error_fluentd.conf)


### DB 스키마

```sql
CREATE DATABASE log;

CREATE TABLE `log`.`game_server` (
  `serial` BIGINT NOT NULL AUTO_INCREMENT,
  `server_id` INT NOT NULL,
  `log_type` VARCHAR(45) NOT NULL,
  `created_at` DATETIME NOT NULL,
  `msg` VARCHAR(1024) NOT NULL,
  PRIMARY KEY (`serial`));

CREATE TABLE `log`.`gateway_server` (
  `serial` BIGINT NOT NULL AUTO_INCREMENT,
  `log_type` VARCHAR(45) NOT NULL,
  `created_at` DATETIME NOT NULL,
  `msg` VARCHAR(1024) NOT NULL,
  PRIMARY KEY (`serial`));
```


#### error_logs 테이블
```sql
CREATE TABLE `log`.`error_logs` (
  `serial` BIGINT NOT NULL AUTO_INCREMENT,
  `server_type` VARCHAR(45) NOT NULL,
  `server_id` INT NOT NULL,
  `created_at` DATETIME NOT NULL,
  `msg` VARCHAR(1024) NOT NULL,
  PRIMARY KEY (`serial`));
```