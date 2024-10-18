# FluentD
FluentD�� ���� Log ó�� �н� �� �ǽ��� ���� �������丮�Դϴ�.

- FluentD �н� : [Study ���丮](./Study/)
- Integrating Fluentd �ǽ� ����

<br> 

## Integrating Fluentd �ǽ� ����

(Docker Container Logs ����� Ȱ���Ͽ�,)

���� �������� ��µ� �α׸� Fluentd�� �����ͺ��̽��� �����մϴ�.

- �̶� ���� ������ [���� ���� ����](https://github.com/yujinS0/Omok-Game) �������丮�� ����մϴ�.


## Requirements

### 0. ZLogger�� ���� JSON ���� �ܼ� ���

ZLogger�� Ȱ���Ͽ� �ֿܼ� JSON �������� �αװ� ��� �ǵ��� �մϴ�.

### 1. Flunted�� ����Ͽ� �α� ����

�����̳�ȭ�� ������ �ܼ� ��¹��� fluented�� ���� �ǰ��մϴ�.

����� ũ�� �ΰ���(a, b)�� �ֽ��ϴ�.

#### a. local ȯ�濡�� fluent-package(���� td-agent)�� Ȱ��

- local�̳� VM ȯ�濡�� fluentd�� ��ġ�Ͽ� Ư�� ����� �α׸� �����ϴ� ���
- ��Ŀ�� �⺻������ �����̳� �α׸� `/var/lib/docker/containers/<containers-id>/<containers-id>-json.log`�� ����
	
#### b. �����̳�ȭ�� fluent�� docker container logs ����

- docker logging driver�� ���� �����̳�ȭ�� fluentd�� �α׸� �����ϴ� ���
- fluentd ���� ���Ͽ��� fluentd-input �÷������� ����� �α׸� ���� �� ����
	```xml
	<source>
	  @type forward  # forward input �÷����� ���
	  port 24224     # Docker logging driver�� �����ϴ� ��Ʈ�� ���� (fluentd �����̳� ���� �� ������ ��Ʈ)
	  bind 0.0.0.0   # ��� IP�κ��� �α� ����
	</source>

	<match docker.*>  # ��Ŀ���� ���۵� �α� ó��
	  @type stdout    # �α׸� stdout���� ����� ���� �ְ�, �ʿ��� ��� �����ͺ��̽��� ���� ����
	</match>
	```

### 2. Fluentd�� ���� MySQL �����ͺ��̽��� ����

���� Fluentd ���� �˸��� MySQL ������ ���̽��� ���̺�� ���� �ǵ��� �մϴ�.

���⼭ �ٽ��� ZLogger�� ���� ���� �����õ� �α׸� �ּ��� ���μ�������,

�˸��� fluentd ������ ���� MySQL DB�� �αװ� ����Ǵ°��Դϴ�.

<br>

#### �α׷� �Ʒ��� �� ��� ���� �ؾ� �մϴ�

- [ ] Daily Active Users
- [ ] User-Specific Statistics
  - [ ] �α��� Ƚ��
  - [ ] ���� �÷��� Ƚ��
- [ ] Time-Based Statistics
  - [ ] �Ⱓ �� �α��� �α�
  - [ ] �Ⱓ �� ��Ī ��û ��
  - [ ] �Ⱓ �� ��Ī ���� �� (��Ī ��û �� ����� ���� ����)
  - [ ] �Ⱓ �� ���� �÷��� �� (���������� ���� game ��)

  
  ---

 
 ### �α� �� ���� 
 
1. �α��� ���� �� (Daily Active Users)
 
 - �α��� ���� ���δ� Daily Active Users�� �α��� Ƚ�� ��迡 ����
 - �ʵ�
    + action : login_success
    + playerId
 - ����
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
  
2. ��Ī ��û ���� �� (Time-Based Statistics �Ⱓ �� ��Ī ��û ��)

 - �ʵ�
    + action : match_request
    + playerId
 - ����
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

 3. ��Ī ���� �� (Time-Based Statistics �Ⱓ �� ��Ī ���� ��)
 
 - �ʵ�
    + action : match_success
    + playerId
 
 - ����
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
 
 4. ���� ���� �� (Time-Based Statistics �Ⱓ �� ���� �÷��� ��)

 - �ʵ�
    + action : game_end
    + winnerId
    + loserId

 - ����
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

 ### �α� ���� ���̺� ���� ����

 ```sql
 create database game_logs;

 ```

 #### �α��� �α� ���̺� login_logs

  ```sql
    CREATE TABLE `game_logs`.`login_logs`(
        id BIGINT AUTO_INCREMENT PRIMARY KEY,
        playerId VARCHAR(255) NOT NULL,    -- �÷��̾� ID
        timestamp DATETIME NOT NULL        -- �α��� �߻� �ð�
    );

 ```
 - �α��� ���� ������ ����ϴ� ���̺�. 
 - Daily Active Users �� �α��� Ƚ�� ��꿡 ���˴ϴ�.


 
 #### ��Ī �α� ���̺� matching_logs

  ```sql
    CREATE TABLE `game_logs`.`matching_logs`(
        id BIGINT AUTO_INCREMENT PRIMARY KEY,
        playerId VARCHAR(255) NOT NULL,    -- ��Ī�� ��û�� �÷��̾� ID
        time DATETIME NOT NULL,            -- ��Ī ��û �Ǵ� ���� �ð�
        tag VARCHAR(50) NOT NULL           -- 'match_request' �Ǵ� 'match_success' ������ ���� �±�
    );

 ```
 - ��Ī ��û�� ���� ������ ����ϴ� ���̺�. 
 - ��Ī ��û ���� ��Ī ���� �� ��迡 ���˴ϴ�.



 #### ���� �α� ���̺� play_logs

  ```sql
	CREATE TABLE `game_logs`.`play_logs`(
		id BIGINT AUTO_INCREMENT PRIMARY KEY,
		winnerId VARCHAR(255) NOT NULL,    -- ���� ID
		loserId VARCHAR(255) NOT NULL,     -- ���� ID
		timestamp DATETIME NOT NULL        -- ���� ���� �ð�
	);

 ```

 - ���� ����� ����ϴ� ���̺�.
 - ���� �÷��� �� �� ���� ���� ��迡 ���˴ϴ�.

---

### ��� ���� ����

#### 1. **Daily Active Users (DAU)**
   - Ư�� ��¥�� �α����� ���� �÷��̾� ���� ����մϴ�.

```sql
SELECT COUNT(DISTINCT playerId) AS daily_active_users
FROM login_logs
WHERE DATE(timestamp) = CURDATE();
```

- **����**: `DISTINCT`�� ����Ͽ� �ߺ� �α����� ������ ������ �÷��̾� ���� ���մϴ�. ��¥�� `CURDATE()`�� ������ ��¥�� ���͸��մϴ�.



#### 2. **User-Specific Statistics**

##### 2.1 **�α��� Ƚ��**
   - �� �÷��̾��� �α��� Ƚ���� ����մϴ�.

```sql
SELECT playerId, COUNT(*) AS login_count
FROM login_logs
GROUP BY playerId;
```

- **����**: �� �÷��̾��� `playerId`�� �������� �׷�ȭ�� ��, �α����� Ƚ���� ����մϴ�.

##### 2.2 **���� �÷��� Ƚ��**
   - �� �÷��̾��� ���� �÷��� Ƚ���� ����մϴ�.

```sql
SELECT playerId, COUNT(*) AS play_count
FROM (
    SELECT winnerId AS playerId FROM play_logs
    UNION ALL
    SELECT loserId AS playerId FROM play_logs
) AS combined_players
GROUP BY playerId;
```

- **����**: `winnerId`�� `loserId` ��θ� �ϳ��� `playerId`�� ��ģ ��, �� �÷��̾��� ���� ���� Ƚ���� ����մϴ�. **UNION ALL**�� ����Ͽ� ���ڿ� ���ڸ� ��� ������ �� �׷�ȭ�մϴ�.


#### 3. **Time-Based Statistics**

##### 3.1 **�Ⱓ �� �α��� �α�**
   - Ư�� �Ⱓ ���� �α����� ���� �÷��̾� ���� ����մϴ�.

```sql
SELECT COUNT(DISTINCT playerId) AS unique_logins
FROM login_logs
WHERE timestamp BETWEEN '2024-10-01' AND '2024-10-31';
```

- **����**: `DISTINCT`�� ����Ͽ� �ߺ� �α����� ������ ������ �÷��̾� ���� ���մϴ�. Ư�� �Ⱓ ���� �߻��� �α����� ���͸��մϴ�.

##### 3.2 **�Ⱓ �� ��Ī ��û ��**
   - Ư�� �Ⱓ ���� �߻��� ��Ī ��û ���� ����մϴ�.

```sql
SELECT COUNT(*) AS match_request_count
FROM matching_logs
WHERE tag = 'match_request'
  AND time BETWEEN '2024-10-01' AND '2024-10-31';
```

- **����**: ��Ī ��û �±װ� `"match_request"`�� �α׸� ���͸��Ͽ� ��Ī ��û ���� ����մϴ�.

##### 3.3 **�Ⱓ �� ��Ī ���� �� (��Ī ��û �� ����� ���� ����)**
   - Ư�� �Ⱓ ���� ��Ī ��û �� ����� ��Ī ���� ����մϴ�.

```sql
SELECT COUNT(*) AS match_success_count
FROM matching_logs
WHERE tag = 'match_success'
  AND time BETWEEN '2024-10-01' AND '2024-10-31';
```

- **����**: ��Ī ���� �±װ� `"match_success"`�� �α׸� ���͸��Ͽ� ��Ī ���� ���� ����մϴ�.

##### 3.4 **�Ⱓ �� ���� �÷��� �� (���������� ���� ���� ��)**
   - Ư�� �Ⱓ ���� ����� ���� ���� ����մϴ�.

```sql
SELECT COUNT(*) AS game_play_count
FROM play_logs
WHERE timestamp BETWEEN '2024-10-01' AND '2024-10-31';
```

- **����**: `play_logs` ���̺��� ���� ���� �ð��� Ư�� �Ⱓ ���� �ִ� ������ ������ ����մϴ�.



<br>
<br>
<br>

 ---

<br>



fluentd ���� ���� ����
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
