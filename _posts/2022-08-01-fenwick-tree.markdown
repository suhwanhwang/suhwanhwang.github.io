---
layout: post
title: "Fenwick Tree"
date: 2022-08-01 08:00:00 +0900
categories: algorithm
---

알고리즘 문제를 풀다보면 구간의 합을 구하는 문제가 종종 보인다.
이때 간단하게 생각할 수 있는 방법은 누적합을 이용해서 계산하는 방법이다.
예를 들어 `int[] array = {0, 1, 2, 3, 4, 5};` 가 있다고 하면,

누적합은 `sum = {0, 1, 3, 6, 10, 15}`
Binary indexed tree라고도 불리우는 Fenwick tree는 구간의 합을 빠르게 구할 수 있는 자료구조다.
