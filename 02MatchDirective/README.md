# 02. �α� ���� �� ���� ��Ī ( Match Directive )

- �±� ���� ��Ī �н�
- �α� �����Ϳ� ���ο� �ʵ� �߰� �н� ( Filter Plugin - record_transformer)
- �±� ���Ͽ� ���� �ƿ�ǲ�� �ٸ��� �ϴ� �α� ����� �н�
- �α� �����ͷ� ����� �ϴ� ��� �н�

### ���� �ǽ�
- [���� Ÿ�Կ� ���� ������ �����ϴ� ����](sample_match_servertype_fluentd.conf)
- [�α� Ÿ��(����)�� ���� ������ �����ϴ� ����](sample_match_loglevel_fluentd.conf)


### ����
- [Error �α� ���� �����ϱ�](task_log_error_fluentd.conf)


### DB ��Ű��

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


#### error_logs ���̺�
```sql
CREATE TABLE `log`.`error_logs` (
  `serial` BIGINT NOT NULL AUTO_INCREMENT,
  `server_type` VARCHAR(45) NOT NULL,
  `server_id` INT NOT NULL,
  `created_at` DATETIME NOT NULL,
  `msg` VARCHAR(1024) NOT NULL,
  PRIMARY KEY (`serial`));
```