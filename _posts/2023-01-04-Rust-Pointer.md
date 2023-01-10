---
layout: post
title: "[Rust] 참조와 포인터"
date: 2023-01-04 01:00:00 +0900
description: "Rust에서 포인터처럼 사용할 수 있는 참조자, 원시포인터, Box<T>, Rc<T>, 그리고 RefCell<T> 등에 대해 알아본다"
tags: rust
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

# `NonNull`

    > `*mut T` but non-zero and covariant

Rust 공식 문서의 설명이다. Null 값을 가지지 않도록 해주며, `Option<NonNull<T>>`와 같은 식으로 둘러싸서 필요할 경우에만 `None` 값을 가지도록 할 수도 있을 것이다. 공식 documnetation에서는 꼭 필요한 경우가 아니면 그냥 `*mut T`를 사용하라고 되어 있다.

# `Box<T>`
가장 대표적인 스마트포인터로, 힙에 있는 데이터를 가리키게 되어 있다. 데이터는 힙에 저장하면서, 그것을 가리키는 포인터는 스택에 저장해 두는 것이다. 크기를 컴파일 타임에 알 수 없는 자료형을 저장하고 싶을 때 사용하게 된다. 

대표적으로, 연결리스트처럼 재귀적인 자료형을 나타낼 때 `Box`를 사용하게 된다. 연결리스트의 노드를 생각해보면

```rust
struct Node {
    value: i32,
    next: Option<Node>
}
```
와 같은 형태가 될 것이다. 노드에 저장된 수와, 현재 노드가 가리키는 다음 노드를 저장해 두는 것이다. 이런 식의 구현은 자료형의 데이터 크기를 컴파일 타임에 알 수가 없기 떄문에 컴파일이 되지 않는다. 따라서 Box를 사용해서 다음과 같이 구현해야 한다.


```rust
struct Node {
    value: i32,
    next: Option<Box<Node>>
}
```

위와 같은 경우, 실제로 next에 저장되는 것은 Node 그 자체가 아니라 Node를 가리키고 있는 포인터이고, 포인터는 크기가 정해져 있기 때문에 문제 없이 컴파일이 가능하다. 

Box는 스마트 포인터의 종류라고는 하지만.. Rust가 가지는 소유권이라는 특성상 포인터처럼 자유자재로 쓸 수 있는 것은 아니고, "재귀적인 자료형같은걸 구현할 때 써야 하는 것" 정도의 느낌인 것 같다. 우선 한 객체에 하나의 Box만이 있을 수 있으니 Box를 다루는 것과 Box가 가리키는 객체를 다루는 것에 프로그래머의 입장에서는 체감되는 차이를 잘 모르겠다. 그래서 무언가를 가리킨다는 것이 전혀 연상되지 않는 "Box"라는 이름을 가진게 아닌가 한다. Heap의 데이터를 품고 있는 것이니까..?

# `Rc<T>`
Rc는 Reference Counter를 의마한다. 어떤 객체를 참조하고 있는 포인터가 몇 개인지를 세다가, 카운터가 0이 되면 그 객체를 메모리에서 삭제하고 메모리 관리를 해주는 것이다. 
<p align="center" style="color:gray">
<img src="https://rinthel.github.io/rust-lang-book-ko/img/trpl15-03.svg" width="80%"><br>
출처: <a href="https://rinthel.github.io/rust-lang-book-ko/ch15-04-rc.html"> Rust 공식 가이드북 </a>
</p>

위와 같은 데이터 구조를 생각해보자. 이는 Box로는 구현이 불가능하다. a가 가리키고 있는, 5를 저장하고 있는 노드의 소유권이 불명확하기 때문이다. Rust에서 소유권을 가진 것은 하나여야 한다. 이런 경우 `Box<T>` 대신 `Rc<T>`를 사용해서 해결할 수 있다.

```rust
use std::rc::Rc;

struct Node{
    value: i32,
    next: Option<Rc<Node>>
}
```

편의상 위의 그림에서 5를 가진 노드까지만 나타내고, 그 뒤의 10이나 Nil을 가진 Node는 무시하겠다.
```rust
let a = Rc::new(Node{value: 5, next: None});
let b = Node{value: 3, next: Some(Rc::clone(&a))};
let c = Node{value: 4, next: Some(Rc::clone(&a))};

println!("{}", a.value);
```

```Output: 5```
 
Rc가 만들어내는 reference는 읽기만 가능하다. 즉, mut이 아니다. Rust는 mutable reference를 여러 개 만드는 것을 허용하지 않기 떄문이다.

# `RefCell<T>`
`RefCell<T>`는 `Rc<T>`의 이런 단점을 해결하기 위해 있는 것으로, 하나의 mutable reference를 만드는 역할을 한다. 위의 코드같은 경우, a가 가리키는 노드에 저장되어 있는 값 5를 수정하는 것은 불가능하다. a, b, c가 모두 immutable reference로 Node를 참조하고 있기 때문이다. 이것을 해결해주려면 Rc안에 RefCell을 살짝 끼워넣어주면 된다.

```rust
use std::rc::Rc;
use std::cell::RefCell;

struct Node{
    pub value: i32,
    next: Option<Rc<RefCell<Node>>>
}
```

`value` 앞에 `pub`을 추가한 것은 value를 외부에서 바꾸는 예시를 보여주기 위해서이다.

```rust
let a = Rc::new(RefCell::new(Node{value: 5, next: None}));
let b = Node{value: 3, next: Some(Rc::clone(&a))};
let c = Node{value: 4, next: Some(Rc::clone(&a))};
```

이제 `value`값을 다음과 같이 바꿀 수 있게 된다.

```rust
a.borrow_mut().value = 10;
println!("{}, {}", 
    a.borrow().value, 
    b.next.unwrap().borrow().value
);
```

``` Output: 10, 10```

여기서 `Rc<RefCell<Node>>` 자체는 immutable reference이지만, 그 안에 들어 있는 `RefCell<Node>`이 mutable이므로 `value`값을 바꾸는 접근을 할 수 있게 된다. 이렇게 `RefCell<T>`는 일반적으로 `Rc<T>`와 조합해서 많이 사용한다. 이것을 내부 가변성(interior mutability)이라고 한다.

위의 코드에서 `RefCell`이 가리키는 값에 접근할 때 `borrow()`와 `borrow_mut()`을 사용한 것을 볼 수 있을 것이다. 이들은 `RefCell`이 감싸고 있는 객체에 대한 (immutable) reference와 mutable reference를 만드는 역할을 한다. 이때 Rust의 기본 원칙인, 한 scope안에 동일 객체의 2개 이상 mutable reference가 있을 수 없다는 것에 따라 `borrow_mut()`은 한 개만 만들 수 있을 것이다.


# `Weak<T>`
Reference counter 방식의 스마트 포인터는 기본적으로 순환참조 시의 메모리 누수 문제가 있다. 만약 두 개의 객체가 서로를 참조한다면, 두 객체의 reference counter는 1로, 영원히 메모리에서 사라지지 않게 되기 때문이다. `Weak<T>`는 `Rc<T>`에서 이를 해결하기 위해 존재하는 타입이다. `Weak`라는 이름은 "약한 참조"에서 나온 것으로, 소유권을 가지지 않는 `Weak<T>`의 특징에서 비롯된 것이다. 

`Week<T>`는 `weak_count`를 1 증가시키는 대신, `strong_count`는 증가시키지 않는다. 메모리를 정리하는 것은 `strong_count`가 0이 될 때 정리하는 것으로, `week_count`와는 관련이 없다. 따라서 두 메모리가 서로를 참조할 때, 하나는 강한 참조(`Rc<T>`)로, 하나는 `Week<T>`로 참조하는 방식을 사용하면 순환참조를 막아줄 수 있다. 예를 들어서, 양방향 연결리스트(doubly linked list)를 다음과 같이 구현하는 것이 가능하다.

```rust
struct Node{
    data: isize,
    prev: Option<Weak<RefCell<Node>>>,
    next: Option<Rc<RefCell<Node>>>,
}
```

참조하는 값을 정해줄 때는, `Rc<T>`에서 `clone`을 사용한 것 대신에 `Rc::donwgrade()`를 사용해주면 약한 참조가 된다.

```rust
a.borrow_mut().next = Some(Rc::clone(&b));
b.borrow_mut().prev = Some(Rc::downgrade(&a));
```

# 정리

<!-- 
|                   | 여러 개 생성 가능     | 수정 가능(mutable) |
| ----------------- | :---------------: | :--------------: |
| `Box<T>`          | X                 | X |
|`Rc<T>`            | O                 | X |
|`RefCell<T>`       | X                 | O |
|`Rc<RefCell<T>>`   | O                 | O (내부 가변성) | -->
<center>
<style type="text/css">
.tg  {border-collapse:collapse;border-color:#ccc;border-spacing:0;}
.tg td{background-color:#fff;border-bottom-width:1px;border-color:#ccc;border-style:solid;border-top-width:1px;
  border-width:0px;color:#333;font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 14px;
  word-break:normal;}
.tg th{background-color:#f0f0f0;border-bottom-width:1px;border-color:#ccc;border-style:solid;border-top-width:1px;
  border-width:0px;color:#333;font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;
  padding:10px 14px;word-break:normal;}
.tg .tg-9wq8{border-color:inherit;text-align:center;vertical-align:middle}
.tg .tg-l0uw{background-color:#efefef;border-color:inherit;font-family:"Courier New", Courier, monospace !important;text-align:left;
  vertical-align:middle}
.tg .tg-fsme{background-color:#efefef;border-color:inherit;text-align:center;vertical-align:middle}
.tg .tg-qoqj{background-color:#f9f9f9;border-color:inherit;font-family:"Courier New", Courier, monospace !important;text-align:left;
  vertical-align:middle}
.tg .tg-kyy7{background-color:#f9f9f9;border-color:inherit;text-align:center;vertical-align:middle}
.tg .tg-t7wa{border-color:inherit;font-family:"Courier New", Courier, monospace !important;text-align:left;vertical-align:middle}
</style>
<table class="tg">
<thead>
  <tr>
    <th class="tg-l0uw"></th>
    <th class="tg-fsme">여러 개 생성 가능</th>
    <th class="tg-fsme">수정 가능</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td class="tg-qoqj">Box&lt;T&gt;</td>
    <td class="tg-kyy7">X</td>
    <td class="tg-kyy7">X</td>
  </tr>
  <tr>
    <td class="tg-t7wa">Rc&lt;T&gt;</td>
    <td class="tg-9wq8"><span style="font-weight:400;font-style:normal;text-decoration:none;color:black">O</span></td>
    <td class="tg-9wq8">X</td>
  </tr>
  <tr>
    <td class="tg-qoqj">RefCell&lt;T&gt;</td>
    <td class="tg-kyy7"><span style="font-weight:400;font-style:normal;text-decoration:none;color:black">X</span></td>
    <td class="tg-kyy7"><span style="font-weight:400;font-style:normal;text-decoration:none;color:black">O</span></td>
  </tr>
  <tr>
    <td class="tg-t7wa">Rc&lt;RefCell&lt;T&gt;&gt;</td>
    <td class="tg-9wq8"><span style="font-weight:400;font-style:normal;text-decoration:none;color:black">O</span></td>
    <td class="tg-9wq8">O (내부 가변성)</td>
  </tr>
</tbody>
</table>
</center>
* `Weak<T>`: `Rc<T>`의 순환참조 문제를 해결하기 위해 사용






# 참고문헌
<a href="https://rinthel.github.io/rust-lang-book-ko/ch04-02-references-and-borrowing.html"> Rust 공식 가이드북 (한글 번역) - 참조자와 빌림</a><br>
<a href="https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/raw-pointers.html">Rust 공식 가이드북 (영문, 1판) - Raw Pointers </a><br>
<a href="https://doc.rust-lang.org/std/ptr/struct.NonNull.html#impl-Pointer-for-NonNull%3CT%3E"> Rust Documentation - `Struct std::ptr::NonNull` </a><br>
<a href="https://applied-math-coding.medium.com/an-introduction-into-rust-part-12-box-t-rc-t-and-refcell-t-fae061d2d7fb"> Medium - An Introduction into Rust. Part 12: Box\<T\>, Rc\<T\> and RefCell\<T\> </a><br>
쿠지라 히코우즈쿠에. (2023). _만들면서 배우는 러스트 프로그래밍_. (양현, 역). 파주: 위키북스. (원서출판 2022).