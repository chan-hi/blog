---
title: "Chapter 2: 주방 규모를 정하다 — 서버 타입"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, server, micro, standard, high-memory, high-cpu, cpu-intensive]
description: "NCP NCE 자격증 대비 학습 자료 - 서버 타입별 CPU:Memory 비율과 용도 완벽 정리"
---

> **학습 목표**: 각 서버 타입별 CPU:Memory 비율과 용도를 완벽히 매칭한다.

---

## 이야기: 메뉴판처럼 늘어선 서버 타입들

G3 서버를 만들기로 한 민준. 이번엔 서버 **타입**을 골라야 했다.

```
[ 서버 타입 선택 ]
□ Micro
□ Standard
□ High Memory
□ High CPU
□ CPU-Intensive
□ ...
```

> "다 뭐가 다른 거야? 그냥 제일 좋은 거 쓰면 안 돼?"

지수가 웃으며 말했다.

> "용도에 맞게 써야 돈 낭비가 없어. 각 타입마다 **CPU와 Memory의 비율**이 달라.
> 어떤 작업을 하느냐에 따라 뭐가 더 많이 필요한지 달라지거든."

---

## 🔑 핵심 개념: 서버 타입과 CPU:Memory 비율

NCP 서버 타입은 **vCPU : Memory(GB)** 비율로 구분된다.

| 타입 | **vCPU : Memory** | 주요 용도 | 특이사항 |
|------|------------------|----------|---------|
| **Micro** | **1 : 1** | 테스트, 개인 프로젝트 | **1년간 무료** 제공 (이후 과금) |
| **Standard** | **1 : 4** | 일반 웹서버, DB | 가장 기본적인 범용 타입 |
| **High Memory** | **1 : 8** | 고성능 DB, 인메모리 캐시 | 메모리가 CPU 대비 8배 |
| **High CPU** | **1 : 2** | 과학적 모델링, 컴퓨팅 집약 | CPU 비율이 높음 |
| **CPU-Intensive** | **1 : 2** | 딥러닝, 고성능 웹서버 | 많은 연산 처리 특화 |

> ⚠️ **시험 포인트**  
> **High CPU**와 **CPU-Intensive** 둘 다 1:2 비율이지만 용도가 다르다.  
> High CPU → 과학적 모델링 / CPU-Intensive → 딥러닝, 고성능 웹서버

---

## 타입별 선택 가이드

```
어떤 작업을 하나요?

무료로 써보고 싶다
  └→ Micro (1:1) — 1년 무료, 이후 과금

일반적인 웹서버, 중소형 DB
  └→ Standard (1:4) — 가장 많이 씀

수백만 건 쿼리를 빠르게 처리하는 DB
Redis, Elasticsearch 같은 인메모리
  └→ High Memory (1:8)

기상 예측, 유전자 분석, 물리 시뮬레이션
  └→ High CPU (1:2)

딥러닝 전처리, 트래픽 폭발적인 웹서버
  └→ CPU-Intensive (1:2)
```

---

## 이야기: 민준의 선택

민준은 고민 끝에 결정했다.

```
[클라우드빵집 서버 구성]

웹 애플리케이션 서버: Standard (1:4)
  → 일반적인 HTTP 요청 처리

주문 데이터베이스 서버: High Memory (1:8)
  → 수많은 주문 쿼리, JOIN 연산 처리

AI 추천 시스템 (나중에): CPU-Intensive (1:2)
  → 딥러닝 추론 처리
```

> "처음엔 Micro로 테스트하고, 실서비스는 Standard로 가야겠다.  
> DB는 High Memory가 맞는 것 같고."

---

## 🧠 암기법

```
Micro  → 1:1  (혼자 일하는 작은 가게)
Standard → 1:4 (점원 1명이 테이블 4개 담당 — 기본 세팅)
High Memory → 1:8 (기억력 좋은 바리스타 — 모든 주문 외움)
High CPU / CPU-Intensive → 1:2 (빠른 손을 가진 요리사)
```

---

## 📝 시험 대비 체크리스트

- [ ] Micro: **1:1** 비율, **1년 무료** (이후 과금)
- [ ] Standard: **1:4** 비율, 일반 웹/DB
- [ ] High Memory: **1:8** 비율, 고성능 DB/인메모리
- [ ] High CPU: **1:2** 비율, 과학적 모델링/컴퓨팅 집약
- [ ] CPU-Intensive: **1:2** 비율, 딥러닝/고성능 웹서버
- [ ] High CPU와 CPU-Intensive는 비율은 같지만 용도가 다름

---

## 🧠 암기 핵심 문장

> "**메모리가 중요하면 High Memory(1:8), CPU가 중요하면 1:2, 기본은 Standard(1:4).**"

---

*다음 챕터: `03_특수서버.md` → 베어메탈과 GPU — 더 강한 주방*
