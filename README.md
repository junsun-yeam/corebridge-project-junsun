<h1 align="center">
    <img src="docs/assets/imgs/CoreBridge-icon.png" alt="CoreBridge 아이콘" width="30" height="30">
    CoreBridge
</h1>

<p align="center">
  <img src="docs/assets/imgs/CoreBridge-logo.png"  alt="CoreBridge 로고" height="500" />

<br /><br />

---

# 📑 목차 (Table of Contents)

- [프로젝트 기획과 설계](#프로젝트-기획과-설계)
    + [1. 시스템 아키텍처](#-1-시스템-아키텍처)
    + [2. ERD](#-2-erd)
    + [3. 핵심기술](#-2-erd)
  

- [나의 역할 소개](#-프로젝트-소개)
    * [1. 내가 맡은 주요기능](#1-개요)
    * [2. 주요 성능 개선](#2-핵심-기능)
    * [3. 한계](#3-한계)


---

# 프로젝트 기획과 설계

## 1. 시스템 아키텍처

![시스템아키텍쳐.png](https://github.com/user-attachments/assets/fa562ef8-c613-4f0e-8a2c-3ace9a98cf95)

## 2. ERD

![ERD.png](./docs/ERD.png)

## 3. 핵심 기술
Frontend
<p> <img src="https://img.shields.io/badge/Vue.js-4FC08D?style=flat&logo=vue.js&logoColor=white" /> <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white" /> <img src="https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black" /> <img src="https://img.shields.io/badge/TailwindCSS-06B6D4?style=flat&logo=tailwindcss&logoColor=white" /> </p>
Backend
<p> <img src="https://img.shields.io/badge/Java-007396?style=flat&logo=openjdk&logoColor=white" /> <img src="https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat&logo=springboot&logoColor=white" /> <img src="https://img.shields.io/badge/QueryDSL-0099CC?style=flat&logoColor=white" /> </p>
Database
<p> <img src="https://img.shields.io/badge/MariaDB-003545?style=flat&logo=mariadb&logoColor=white" /> </p>
Server / Infra
<p> <img src="https://img.shields.io/badge/Nginx-009639?style=flat&logo=nginx&logoColor=white" /> <img src="https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white" /> </p>
Development Tools
<p> <img src="https://img.shields.io/badge/IntelliJ-000000?style=flat&logo=intellijidea&logoColor=white" /> <img src="https://img.shields.io/badge/VSCode-007ACC?style=flat&logo=visualstudiocode&logoColor=white" /> 

# 나의 역할 소개

## 1. 내가 맡은 주요기능

### 채용공고 등록 기능
채용공고를 생성할 때, 등록 가능한 부서/기술스택/근무지를 사전 조회하여 데이터 정합성 보장하도록 설계

![채용공고 등록](docs/gif/공고등록.gif)

### 채용공고 전체조회 및 상세조회 기능
채용공고 조회시 각 채용공고별 필요한 데이터를 정제해서 조회토록 구현
![채용공고 조회](docs/gif/공고조회.gif)

### 채용공고 지원현황 조회
칸반보드를 활용한 공고별 지원자들의 단계 확인
![지원현황 조회](docs/gif/지원현황.gif)

### 채용공고 지원자수 시각화
공고별 지원자 수를 차트를 통한 시각화 구현
![시각화](docs/gif/시각화.gif)

### 지원자의 현재 지원현황 조회
지원자가 지원한 프로세스 조회 구현
![지원현황 확인](docs/gif/지원단계.gif)

### 지원자 채용공고 조회
지원자가 어떤 공고가 있는지 확인하는 기능 구현
![지원현황 확인](docs/gif/채용공고조회.gif)

## 2. DB 성능개선

본 프로젝트에서 실제 데이터 50만건 이상의 공고를 등록 후에 채용공고 조회 API의 성능 병목을 분석하고,
원인 파악 및 개선을 단계적으로 수행

---

### 문제요약

1. 연관관계 LAZY 로딩으로 인한 N+1 발생
- 채용공고 조회 시 부서, 채용단계, 지원자 수 등 관련된 테이블을 함께 조회하면서 예상치 못한 다중 N+1 쿼리가 발생
- 결과적으로 DB Connection Pool 점유율 상승 -> 응답 지연 -> 페이징 렌더링 지연으로 이어짐

2. 인덱스 미적용
- 인덱스가 적용되지 않아 모든 데이터를 확인하는 Full Scan 발생

### 원인 분석
- N+1 발생 JPA LAZY 기본값 + join fetch 미사용
- index 미적용 %keyword% 검색으로 인덱스 탐색 불가
- 불필요한 모든 필드 SELECT / JOIN  
- 50만 건 데이터 전체 Scan

### 개선전 성능
#### 실제 데이터 50만건 기반 성능 측정(Locust)
- 유저: 50명 
- 시간: 2분 
- 테스트대상(전체조회, 경력검색, 복합검색, 제목검색)

![개선전](docs/image/개선전%20성능.png) (실제 측정자료)

![개선전](docs/image/개선전%20성능표.png)


### 개선 방법

1. QueryDsl기반 단일 쿼리 최적화
- 필요한 컬럼만 조회하는 DTO Projection(QueryDsl) 적용
- 불필요한 엔티티 join 제거
- join fetch 적용해 N+1 제거

2. 인덱스 재설계 
- 제목
- 경력(경력, 신입, 무관)
- 공고생성일자

### 개선후 성능
![개선후](docs/image/개선후%20성능.png)

![img.png](docs/image/개선후%20성능%20표.png)

## 3. 한계

- 부분 검색(%keyword%)은 구조적으로 인덱스를 활용할 수 없음
- 제목 기반의 Full-Text 검색은 결국 DB로는 한계

추후 해결 방안
- Elasticsearch 도입 필요성 확인
  - 역색인 기반 구조로 부분 매칭에도 빠른 검색 가능
  - 한글 형태소 분석기(nori tokenizer)로 정확도 개선
  - 검색 스케일 아웃 구조로 대규모 트래픽 대비 가능















