---
title: ETL & Data Pipelines with Shell, Airflow and Kafka 3주차
date: 2023-01-27 00:00:00 +0900
categories: [Education, Coursera - Data Engineering]
tags: [AI, Deep learning, Machine learning, Data Engineering, Airflow, Kafka, ETL, ELT]
description: Coursera - ETL and Data Pipelines with Shell, Airflow and Kafka (IBM) 강의 요약
toc: true
toc_sticky: true
toc_label: 목차
math: true
mermaid: true

---

이번 글에서는 [Coursera의 ETL and Data Pipelines with Shell, Airflow and Kafka (IBM)](https://www.coursera.org/learn/etl-and-data-pipelines-shell-airflow-kafka/home/info) 3주차 강의를 정리합니다. <br/>
이 강좌는 ETL 및 ELT 데이터 파이프라인에 대해 학습하며, Airflow와 Kafka 등을 이용해 이를 배우게 됩니다. <br/>
3주차에는 `Airflow를 이용해 데이터 파이프라인을 구축하는 방법`에 대해 공부합니다.

---

## 1. 데이터 학습 목표
- Apache Airflow의 주요 기능 및 원칙 나열
- 작업 및 종속성의 DAG로 워크플로 설명
- 워크플로를 코드로 정의할 때의 장점 소개
- 특정 DAG를 시각화 하는 다양한 방법 요약
- DAG 정의 파일의 주요 구성 요소 설명 및 연산자를 인스턴스화 하여 태스크 생성
- 로깅 기능을 사용해 작업 상태 모니터링 및 DAG 실행 문제 진단

## 2. Apache Airflow
- Apache Airflow는 훌륭한 오픈소스 워크플로 오케스트레이션 도구
- Batch Data Pipeline과 같은 워크플로를 만들고 실행할 수 있는 플랫폼
- Airflow에서는 워크플로가 DAG (Directed Acyclic Graph)로 표시
- Airflow는 Kafka, Storm, Spark 같은 도구들과는 달리, 데이터 스트리밍 솔루션이 아닌, 워크플로 관리자
- 기본 구성 요소
    - Airflow는 예약된 워크플로의 트리거를 처리하는 내장형 `Scheduler`가 존재
    - Scheduler는 예약된 각 워크플로에서 개별 작업을 `Executor`에게 전달
    - `Executor`는 이러한 작업을 `Worker`에 할당하여 실행 및 처리. 그리고 다음 작업 실행
    - `Web Server`는 Airflow의 대화식 사용자 인터페이스 제공 (여기에서 DAG를 검사하고, 트리거 및 디버그 가능)
    - `DAG Directory`는 Scheduler, Executor, Workers에서 액세스 할 수 있는 모든 DAG 파일이 포함
- DAG는 태스크 (수행할 작업들)들 사이의 종속성과 실행 순서를 지정

<figure style="text-align: center;">
    <img src="https://airflow.apache.org/docs/apache-airflow/stable/_images/arch-diag-basic.png" width="700" height="700">
    <figcaption align="center">https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html</figcaption>
</figure> 
<br/>

- Airflow의 주요 기능과 장점
    - 표준 Python을 사용하여 워크플로를 만들고, 이를 통해 데이터 파이프라인 구축 시, 유연성을 유지할 수 있음
    - UI가 유용하여 모니터링, 스케줄링, 워크플로 관리 등을 Web app을 통해 할 수 있음
    - IBM Cloudant와 같이 다른 많은 Plug-and-play 서비스들과 통합 가능
    - Python에 대한 지식이 있다면, 누구든지 워크플로를 배포할 수 있음
    - 오픈소스이기 때문에 무엇인가를 공유하고 싶다면, PR을 만들 수 있음
- Airflow의 네 가지 주요 원칙
    - 확장성: 모듈식 아키텍쳐를 가지고 있으며, 메시지 큐를 사용하여 임의의 Workers의 수를 관리
    - 동적: Airflow 파이프라인은 Python으로 정의되고, 동적 파이프라인 생성을 허용하여 동시 작업 가능
    - 확장 가능성: 자신의 환경에 맞게 연산자를 쉽게 정의하고, 라이브러리를 확장할 수 있음
    - 린 (Lean): Airflow는 매개변수화가 Jinja 템플릿 엔진으로 내장되어 린하고 명시적

## 3. DAG (Directed Acyclic Graph)
- DAG는 방향성 비순환 그래프라고 하는 특별한 종류의 그래프
    - Graph: Nodes and Edges
    - Directed Graph: Each edge has a direction
    - Acyclic: No loops (cycles)
- DAG는 Airflow에서 워크플로 또는 파이프라인을 나타내는데 사용 (Python 코드로 정의된 워크플로)
    - 파이프라인에서 수행하는 각 작업은 DAG에서 노드로 표시
    - Edge는 두 작업이 실행되어야 하는 순서를 정의
    - 따라서 DAG는 Airflow에서 실행해야 하는 작업을 정의하고, 어떤 순서로 실행해야 할지를 위해 사용
    - 이 DAG의 구조는 Python 스크립트로 정의되므로, 작업과 해당 종속성 역시 코드로 정의 (스케줄링도 마찬가지)
- Task는 Python으로 작성되고, Task는 Operators를 구현
- Operators는 DAG의 각 작업이 수행하는 작업을 정의하는데 사용
- DAG는 다음의 논리 블록으로 구성된 Python 스크립트 (아래 그림과 같음)
    - 라이브러리 불러오기, DAG Arguments, DAG 정의, 태스크 정의, 태스크 파이프라인

<figure style="text-align: center;">
    <img src="https://storage.googleapis.com/analyticsmayhem-blog-files/dbt-airflow/sample%20dag%20definition.png" width="700" height="700">
    <figcaption align="center">https://analyticsmayhem.com/dbt/schedule-dbt-models-with-apache-airflow/</figcaption>
</figure> 
<br/>

- Apache Airflow Scheduler를 사용해서 작업자 배열에 워크플로를 배포할 수 있음 (DAG에서 정한 기준에 따라 작동)
    - Airflow Scheduler 인스턴스를 시작하면, 코드에서 지정한 'Start Date' 기준으로 실행
    - 그 후에 Scheduler는 후속 DAG를 지정한 일정 간격에 따라서 실행
- 이와 같이 코드로 워크플로를 정의한다는 것은 유지보수, 버전관리, 협업, 테스트의 측면에서 장점을 갖음
- `task2 >> task3` 코드는 task2가 실행된 후, task3가 실행된다는 의미

## 4. Airflow의 UI

- Airflow 사용자 인터페이스의 랜딩 페이지는 아래와 같고, 기본값은 DAG에 대한 데이터가 포함된 테이블 형태
    - 각 행에는 다음과 같은 환경의 DAG에 대한 대화형 정보가 표시 (DAG 이름, 스케줄, Owner, 최근 태스크 등)

<figure style="text-align: center;">
    <img src="https://airflow.apache.org/docs/apache-airflow/1.10.6/_images/dags.png" width="700" height="700">
    <figcaption align="center">https://airflow.apache.org/docs/apache-airflow/1.10.6/ui.html</figcaption>
</figure> 
<br/>

- DAG 이름을 클릭하게 되면, 'Tree View' 형태로 결과를 아래와 같이 확인할 수 있음
    - 각 실행에 대한 작업의 상태를 타임라인 형태로 보여주고, 기본 날짜와 실행 횟수를 선택할 수도 있음

<figure style="text-align: center;">
    <img src="https://airflow.apache.org/docs/apache-airflow/1.10.6/_images/tree.png" width="700" height="700">
    <figcaption align="center">https://airflow.apache.org/docs/apache-airflow/1.10.6/ui.html</figcaption>
</figure> 
<br/>

- 또한 'Graph View' 형태로 결과를 아레와 같이 확인할 수 있음
    - DAG의 작업과 종속성을 확인할 수 있고, 각 작업은 연산자 유형에 따라 색으로 구분

<figure style="text-align: center;">
    <img src="https://airflow.apache.org/docs/apache-airflow/1.10.6/_images/graph.png" width="700" height="700">
    <figcaption align="center">https://airflow.apache.org/docs/apache-airflow/1.10.6/ui.html</figcaption>
</figure> 
<br/>

## 5. Airflow Monitoring and Logging
- 개발자가 작업 상태를 모니터링 하고, 문제를 진단하며 디버깅하기 위해서는 로깅 기능이 필요
    - Airflow는 Default로 로컬 파일 시스템에 로그 파일들이 저장되어 빠르게 확인 가능
    - Airflow의 production deployment인 경우, 원격 접속을 위해 클라우드 상에 로그 파일을 보낼 수 있음
    - 로그 파일을 검색 엔진과 대시보드로 보내서 검색 및 분석할 수 있고, 이때는 Elastic Search나 Splunk를 권장
    - 로그 파일의 Default 위치: `logs/dag_id/task_id/execution_date/try_number.log`
    - Airflow의 Web Server를 통해 UI 형태로도 로그 결과를 확인할 수 있음
- 구성 요소의 상태를 확인하고, 모니터링 하기 위한 Metrics
    - Counters: 성공 또는 실패한 작업 수와 같이 항상 증가하는 Metrics
    - Gauges: 현재 실행 중인 태스크의 수와 같이 변동 (Fluctuate)할 수 있는 Metrics
    - Timers: 태스크가 성공 또는 실패까지 걸리는 시간과 같이 Time duration과 연관된 Metrics
- Production 환경에서 Airflow의 Metrics는 수집되고, 전송되며 분석됨
    - `StatsD`는 Airflow에서 데이터를 수집하여, Metrics 모니터링 시스템으로 전송할 수 있는 네트워크 데몬
    - 그리고 `Prometheus`에 전달되어 Metrics를 모니터링하고 분석되며, 대시보드에서 시각화를 할 수도 있음