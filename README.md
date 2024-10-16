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
  - [ ] �Ⱓ �� ��Ī ���� ��
  - [ ] �Ⱓ �� ���� �÷��� ��

  
  ---

 
 ### �α� ���� (����: ���� ������)

 ���� �α� 

  ```json
  {
	  "timestamp": "2024-10-16T12:34:56Z",
	  "userId": "user_12345",
	  "event": "login",
	  "eventDetails": {
		"gameId": "game_67890",
		"matchId": "match_12345",
		"result": "win"
	  },
	  "sessionId": "session_98765",
	  "ipAddress": "192.168.1.1",
	  "duration": 120,
	  "meta": {
		"appVersion": "1.0.0",
		"deviceModel": "iPhone12"
	  }
	}

```

���̺� ����


```sql

CREATE TABLE user_activity_logs (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    timestamp DATETIME NOT NULL,         -- �α� �߻� �ð�
    userId VARCHAR(255) NOT NULL,        -- ���� ID
    event VARCHAR(50) NOT NULL,          -- �̺�Ʈ ���� (login, play_game, request_match, complete_match ��)
    gameId VARCHAR(255) NULL,            -- ���� ID (���� ���� �̺�Ʈ���� ���)
    matchId VARCHAR(255) NULL,           -- ��Ī ID (��Ī ���� �̺�Ʈ���� ���)
    result VARCHAR(50) NULL,             -- ��� (���� �Ϸ� �� win/lose ��)
    duration INT NULL,                   -- ���� �ð� �Ǵ� ���� �ð�(�� ����)
    ipAddress VARCHAR(45) NULL           -- ����� IP �ּ� (���� ����)
);

```


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


zlogger C# �ڵ� ����

```csharp
public class LoggerService
{
    private readonly ILogger _logger;

    public LoggerService(ILogger<LoggerService> logger)
    {
        _logger = logger;
    }

    public void LogLogin(string userId, string platform, string ipAddress)
    {
        var logData = new
        {
            timestamp = DateTime.UtcNow,
            userId = userId,
            event = "login",
            platform = platform,
            ipAddress = ipAddress
        };

        _logger.ZLogInformationWithPayload(logData, "User {userId} logged in");
    }

    public void LogGamePlay(string userId, string gameId, int duration)
    {
        var logData = new
        {
            timestamp = DateTime.UtcNow,
            userId = userId,
            event = "play_game",
            gameId = gameId,
            duration = duration
        };

        _logger.ZLogInformationWithPayload(logData, "User {userId} played game {gameId}");
    }
}

```
