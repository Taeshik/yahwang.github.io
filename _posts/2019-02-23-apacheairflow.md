---
layout: test_post
title: Airflow 기본 정보 (상시 업데이트)
date: 2019-02-23 10:00:00 pm
permalink: posts/airflow
description: Airflow에 대해 정리한 자료
categories: [Tech]
tags: [Engineering, Airflow]
---

- [옵션 설정](#옵션설정)
- [airflow 내부 DB](#airflow_DB)
- [시간정보](#시간정보)
- [Variables](#Variables)
- [JINJA 템플릿]([#JINJA템플릿])

### 옵션설정

참고 : [how to set config](https://airflow.readthedocs.io/en/stable/howto/set-config.html){:target="_blank"}

- 설정 우선순위
1. 환경변수
2. airflow.cfg 파일 내 설정
3. command in airflow.cfg
4. Airflow’s built in defaults

Docker로 설치할 경우, bash에서 printenv를 통해 환경변수 확인가능

- Executor의 종류
    - SequentialExector: 기본 설정, 한 서버에서 task를 순차처리할 수 있음
    - LocalExecutor : 한 서버에서 task들을 병렬처리할 수 있음
    - CeleryExecutor : task를 여러 서버(worker)에 할당하여 처리할 수 있음


- [Securing Connections](https://airflow.apache.org/howto/secure-connections.html){:target="_blank"}

airflow는 접속한 비밀번호를 메타데이터에서 그대로 저장하는데 보안을 위해 cryptography 라이브러리의 FERNET 방식을 사용자가 적용해야 한다. 

참고 : FERNET 방식은 encode와 decode가 같은 대칭키이다.


### airflow_DB

참고 : [
Initializing a Database Backend](https://airflow.readthedocs.io/en/stable/howto/initialize-database.html){:target="_blank"}

기본 DB는 SQLite3로 설정되어 있다. 

SQLite3는 동시 접근이 제한되어 DAG가 병렬처리되지 않고 순차처리(SequentialExecutor)가 되는 문제가 있다. 

airflow 자체에서도 MySQL이나 PostgresSQL로 사용할 것을 권장한다.

SequentialExecutor => Local(Celery)Executor로 변경하여 사용해야 한다.

Docker로 설치할 경우, docker-compose를 통해 PostgreSQL을 새로 구축해 연동하여 서비스를 생성한다.

### 시간정보

참고 : [Time Zone](https://airflow.readthedocs.io/en/stable/timezone.html?highlight=pendulum#){:target="_blank"}

airflow에서는 **UTC** 시간을 사용한다. time zone 설정을 지원하기는 하지만 실제 내부에서는 UTC로 다시 변환하여 처리한다. 

현재 UI에서는 UTC만 보이도록 설정되어 있다(?)...

타임존을 설정하기 위해 **pendulum** 라이브러리를 활용할 수 있다. 

``` python
import pendulum

local_tz = pendulum.timezone("Asia/Seoul")
default_args=dict(
    start_date=datetime(2016, 1, 1, tzinfo=local_tz)
...
```

### Variables

변수를 미리 사용자가 지정하여 DAG를 생성할 때 사용 가능하다.

![airflow_var]({{site.baseurl}}/assets/img/tech/airflow_var.jpg)


Variable은 메타DB에 저장되고 .get 함수를 사용할 때마다 매번 접속을 시도한다. 따라서, 변수를 따로따로 만드는 것은 비효율적이다.

JSON 파일에 수많은 변수들을 저장한 뒤에 사용하는 것이 효율적이다.

``` python
from airflow.models import Variable

test = Variable.get("test")
# JSON 파일에서 변수 사용하기
config = Variable.get("config파일명", deserialize_json=True)
var1 = config["var1"]
var2 = config["var2"]
```

### JINJA템플릿