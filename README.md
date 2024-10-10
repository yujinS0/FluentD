# FluentD
FluentD를 통한 Log 처리 학습 및 실습을 위한 레포지토리

## 0. Log Management System
일반적으로 로그 시스템은 로그를 수집하는 Log Collector,
로그를 저장하는 Log Storem
이를 시각화하는 로그 뷰어 Log Visualisation 로 구성된다.



## 1. FluentD란?
로그(데이터) 수집기로 Ruby로 만든 로그 수집용 오픈 소스 솔루션이다. (퍼포먼스가 중요한 부분은 C로)
즉, 다양한 시스템에서 들어오는 여러 로그들을 fluentd 하나로 수집하고 파싱하고 포워딩할 수 있다.

로그 수집 솔루션은 FluentD, Logstash가 주로 사용되고 사용되는 조합에 따라 ELK, EFK, FLG 스택 등으로 불린다.

- ELK 스택: Elasticsearch(), Logstash, Kibana
- EFK 스택: Elasticsearch, FluentD, Kibana
- FLG 스택: Loki, FluentD, Grafana

### FluentD와 Logstash
둘 다 엔터프라이즈급 로그 파이프라인 구성에 사용되는 로그 분석용 파서 역할
수집한 로그를 필터링, 라우팅이 가능하며 다양한 플러그인을 사용할 수 있다.


| FluentD | Logstash |
|:---:|:---:|
| ruby(+C) | jruby |
| 자제 데몬으로 수집 | 자체 수집 X |
| CNCF | Elastic |
| 마이크로 서비스 | 모노리틱 |


### FluentD의 구조


Log Source -> " Input (Parser) -> Engine -> (Filter) -> (Buffer) -> Output (Formatter) " -> Log Storage

중앙의 Engine 부를 제외하고 모두 Plugin 형태로 제공되어 사용자가 설정할 수 있다.
Input, Output 이 필수적인 기능, Parser, Filter, Buffer Formatter 등은 선택적 기능이다.

#### Input
실제로 데이터를 수집하는 부분이다. (다양한 로그 소스를 지원한다.)
거의 모든 경우를 커버하는 Input Plugin이 존재한다.
["제공되는 Input Plugins"](https://docs.fluentd.org/input) 
- ex. tail, forward, udp, tcp, unix, http, syslog, exec, sample, windows_eventlog ...

FluentD에서 내부적으로 사용하는 데이터 구조는 tag, time, record로 구성된다.

```
2012-02-04 01:33:51	// time
myapp.access {		// tag
	"user":"me",	// record
	"page":"1"
}
```

#### Pasrser (optional)
Input Plugin으로 수집한 로그 포맷이 FluentD에서 사용할 수 없는 포맷인 경우, 변환을 위해 사용하는 Plugin이다.
- 이때, FluentD의 디폴트 포맷은 JSON이다.
["제공되는 Parser Plugins"](https://docs.fluentd.org/parser)

#### Filter (optional)
Input -> Paser 를 걸쳐 전달된 데이터를 가공할 때 사용하는 Plugin이다.
아래 3가지 기능을 제공한다.

1. 필터링
	- 특정 데이터만 output으로 보내기 위해 필터링
2. 데이터 필드 추가
	- 데이터 조합하거나 가공해서 새로운 데이터 필드를 만들어 추가
	- 데이터를 전송한 서버명이나 주소 추가도 가능
3. 데이터 필드 삭제 또는 특정 필드 마스킹
	- 데이터의 특정 필드를 삭제하는 기능

#### Output
필터링되고 가공된 데이터를 로그 저장소로 보내는 Pulgin이다.
Output Plugin은 1개의 모드로만 동작한다.
["제공되는 Output Plugins"](https://docs.fluentd.org/output)
- ex. copy, file, null, roundrobin, stdout, exec_filter, forward, exec ...

#### Buffer(optional)
Input에서 들어온 데이터를 바로 output으로 보내지 않고, 중간에 선택적으로 Buffer를 둬서 Throttling(작업처리량줄이기)할 수 있다.
버퍼는 File과 Memory 두가지가 있다.


#### Formatter(optional)
Output Plugin을 통해 로그 저장소로 데이터를 보내기 전에, 로그 저장소에서 사용하는 데이터 포맷으로 변환이 필요한 경우, 데이터를 가공하는 Plugin이다.
["제공되는 Formatter Plugins"](https://docs.fluentd.org/formatter)


### Engine (FluentD Core)

앞서 FluentD의 구조는 Engine과 Plugin으로 구성된다고 했다.
이 중 Engine 부분은 Router 역할을 하는 부분으로 사용자가 제공하는 규칙에 따라 로그 데이터를 다르게 처리할 수 있는 기능을 제공한다.
조건에 따라 다른 필터를 적용하거나(Filter 기능), 다른 아웃풋으로 데이터를 전송하는 기능(Match 기능)을 제공한다.

Router는 Event Router, Label Directives 로 크게 두 부분으로 나눠집니다.

#### 1) Event Router
Event Router는 FluentD에서 가장 중요한 기능 중 하나로, 이벤트를 어떻게 처리할지를 결정하는 역할을 한다.

Event Router는 tag의 패턴 매칭을 통해 Filter와 Match를 선택해서 실행한다.
- Filter : 적용할 필터를 선택하는 기능, 복수의 필터를 적용해 파이프라인을 구성하는 것이 가능하다.
- Match : 파이프라인 끝에서 Output을 선택하는 기능, 파이프라인 당 1개의 Match만 사용할 수 있다.

#### 2) Label Directives
라벨 기능은 GOTO처럼 작동한다.
로그 프로세싱을 작은 단위로 쪼개서 처리할 수 있게 하고, 태그 매칭을 단순화하는데 도움을 준다.


## 2. FluentD 사용법

### 설치 방법
[fluent-package 설치 링크](https://docs.fluentd.org/installation/install-by-deb)

0) 설치 전 [Pre-installation Guide](https://docs.fluentd.org/installation/before-install) 확인
1) Install from Aot
DEB Package를 사용해서 설치를 진행한다.
`lsb_release -a` 명령어로 OS 버전을 확인하고, 해당 버전에 맞는 설치 명령어를 사용한다.


### Directives 리스트
기본적으로 숙지해야하는 Directives 목록입니다.





