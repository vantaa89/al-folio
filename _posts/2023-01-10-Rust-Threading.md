---
layout: post
title: "[Rust] 동시성과 스레딩"
date: 2023-01-10 12:49:00 +0900
description: "Rust에서 제공하는 동시성 프로그래밍 기능을 살펴본다"
tags: rust
giscus_comments: true
---

# 스레드

스레딩은 프로그램의 계산 중 동시에, 독립적으로 수행할 수 있는 것들을 쪼갠 후 여러 개의 **스레드(thread)**가 동시에 수행하게 함으로서 성능을 향상시키는 방법이다. 스레딩은 성능 향상에 매우 효과적이지만, 프로그램을 복잡하게 만들고 이 때문에 다음과 같은 여러 문제점이 발생할 수 있다.

* race condition: 여러 스레드들이 일관성 없는 순서로 데이터나 리소스에 접근하게 되는 것
* deadlock: 두 스레드가 서로가 가진 리소스의 사용이 끝나기를 기다리면서 계속 멈춰 있는 것

러스트는 소유권과 타입 시스템을 통해서, 부정확한 동시성 코드는 컴파일 자체를 차단함으로써 동시성 프로그래밍을 쉽게 만들어준다.

# 참고문헌
<a href="https://rinthel.github.io/rust-lang-book-ko/ch16-00-concurrency.html"> _16. 겁없는 동시성_, The Rust Programming Language (한글 번역)</a>