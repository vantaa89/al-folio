---
layout: post
title: "[Rust] 참조자와 포인터"
date: 2023-01-01 01:00:00 +0900
description: Rust에서 포인터처럼 사용할 수 있는 참조자, 원시(raw) 포인터, NonNull, 그리고 Box에 대해 알아본다
tags: data-structure rust
giscus_comments: true
---

# 참조자(reference)
C/C++의 참조자와 유사한 개념이다. Rust는 데이터를 가진 변수보다 참조자가 더 오래 존재하지 않도록 체크해줘서 dangling이 일어나지 않도록 방지한다.

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```
위의 코드에서, 만약 참조자를 사용하지 않았더라면 `calculate_length(s1)`을 부르는 순간 소유권이 해당 함수로 넘어가버릴 것이다. 이를 방지해주기 위해서 C와 비슷하게 &를 사용해서 넘겨줄 수 있다.

다만, Rust에서의 다른 자료형과 마찬가지로 참조자 또한 기본적으로 불변이다. 따라서 아래와 같이 파라미터를 바꾸는 코드는 에러를 발생시킨다.

```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

이를 해결해주려면 
1. 먼저, `s`를 `mut`로 바꿔야 한다.
```rust
let mut s = String::from("hello");
```
2. 참조자 또한 가변 참조자(mutable reference)로 바꿔야 한다.
```rust
change(&mut s);
```
가변 참조자의 경우, 한 가지 제한사항을 가진다.
>특정한 스코프 내에 특정한 데이터 조각에 대한 가변 참조자를 딱 하나만 만들 수 있다

예를 들어서 다음과 같은 코드는 불가능하다.
```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;
```

이는 데이터 레이스라는 것이 일어나는 것을 컴파일 타임에 방지하기 위한 것이라고 한다. 해결하려면 단지 둘 중 하나를 중괄호로 감싸서 스코프를 만들어주기만 하면 된다.

# 원시 포인터(Raw Pointers)
C/C++의 포인터와 유사하다. Rust가 해주는 컴파일 타임에서의 dangling 방지를 사용할 수 없다. 따라서 사용이 안전하지 않고, 권장하지 않는듯 하다. Raw pointer의 특징은 다음과 같다.
* 유효한 메모리 주소를 가리킨다는 것이 보장되지 않는다. NULL이 아닌지 또한 확인하지 않는다. 이는 `Box`나 `&`와의 가장 큰 차이점이다.
* 메모리 관리를 직접 해주어야 한다. Box와 달리 쓰레기 수집 기능이 없기 때문이다.
* 소유권을 이동시키지 않는다. 따라서 "use-after-free" (free된 후에 참조하는 것)을 방지할 수 없다.
* `&`와 달리 라이프타임 관리 기능이 없다. 즉 danling pointer를 방지할 수 없다.

Rust에서 원시 포인터(raw pointer)라고 불리는 것은 두 가지다.
* `*const T`
* `*mut T`
첫번째는 immutable한 변수에 대한 것, 두 번쨰는 mutable한 변수에 대한 것이라고 보면 될 것이다.
```rust
let x = 5;
let raw = &x as *const i32;

let mut y = 10;
let raw_mut = &mut y as *mut i32;
```
원시 포인터는 위와 같이 간단하게 만들 수 있다. 하지만 이렇게 만든 포인터를 dereference할 때는 가리키는 주소가 안전하지 않을 수 있다는 데에 대한 책임을 져야 한다.
```rust
println!("raw points at {}", *raw);             // compile error
println!("raw points at {}", unsafe {*raw});    // OK
```

# NonNull
> `*mut T` but non-zero and covariant


# 참고문헌
<a href="https://rinthel.github.io/rust-lang-book-ko/ch04-02-references-and-borrowing.html"> Rust 공식 가이드북 (한글 번역) - 참조자와 빌림</a>
<a href="https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/raw-pointers.html">Rust 공식 가이드북 (영문, 1판) - Raw Pointers </a>
<a href="https://doc.rust-lang.org/std/ptr/struct.NonNull.html#impl-Pointer-for-NonNull%3CT%3E"> Rust Documentation - `Struct std::ptr::NonNull` </a>