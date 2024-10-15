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
