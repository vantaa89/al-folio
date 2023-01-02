---
layout: post
title: "[Rust] 연결 리스트 구현하기"
date: 2023-01-02 01:00:00 +0900
description: Rust로 연결 리스트 구현하기
tags: data-structure rust
giscus_comments: true
---

Rust는 소유권과 같은 핵심적인 개념들 때문에 연결리스트(linked list)와 같은 자료형을 구현하기가 까다로운 편이다. 심지어는 <a href="https://rust-unofficial.github.io/too-many-lists/index.html"> Learn Rust With Entirely Too Many Linked Lists </a>같은 책도 있을 정도이다.

여기에서는 rust에서 제공하는 포인터 타입 중 하나인 `Box`를 이용해 간단한 기능을 갖춘 연결리스트를 구현하였다.

# Node
연결리스트의 핵심은 Node이다. Node는 데이터를 가지고, 또 다음 노드에 대한 포인터를 가져야 한다.

먼저 Node를 선언하자.

```rust
pub struct Node<T: Copy> {
    pub data: Option<T>,
    pub next: Option<Box<Node<T>>>
}
```

data의 경우 `T`가 아닌 `Option<T>`로 만들어서 `None` 값 또한 가질 수 있도록 해주었다. 우리는 `head`를 빈 노드로 처리해 줄 것이기 때문에 이때 특히 유용하게 쓸 것이다. next의 경우에도, 연결 리스트의 끝 노드는 다음 노드를 갖지 않으므로 `None`으로 처리해주기 위해 `Option`을 사용하였다. 

새로운 노드의 초기화는 다음과 같이 구현하면 된다.
```rust 
impl<T: Copy> Node<T> {
    pub fn new(data: T) -> Node<T> {
        Node::<T> {data: Some(data), next: None}
    }
    // 뒤에 나올 add(), append(), remove() 등이 모두 여기 들어간다
}
```


## `add()`
현재의 노드에서 i개 후의 노드 뒤에 주어진 데이터를 가진 노드를 추가하는 메서드이다. 뒤에서 나오겠지만 `LinkedList`의 `add`함수를 위해서 구현해놓은 것이다. Rust에서 포인터를 가지고 iterate하는 것은 너무 어려워서 우선은 재귀적으로 구현했다.

```rust
pub fn add(&mut self, i: usize, data: T) {
    if i == 0 {
            let new_node = Node {data: Some(data), next: self.next.take()};
            self.next = Some(Box::new(new_node));
    } else {
        self.next.as_mut().expect("Index out of range").add(i-1, data);
    }
}
```

`take()`는 `Option`의 메서드인데, 원본에는 None을 가져다놓고 원본의 소유권을 가져오는 역할을 한다. 

## `append()`
맨 마지막 노드 뒤에 주어진 데이터를 가진 새로운 노드를 추가하는 메서드이다. `self.next`에 저장된 것이 `None`이 나올 때까지 반복하면 된다. 역시 재귀적으로 정의하였다.
```rust
pub fn append(&mut self, data: T){
    match self.next {
        None => {
            let new_node = Node {data: Some(data), next: None};
            self.next = Some(Box::new(new_node))
        }
        Some(ref mut next) => {
            next.append(data);
        }
    }
}
```
`new_node`를 정의한 후 그걸로 `Box`를 하나 만들어서 `self.next`에 넣어주면 된다.

## `remove()`
현재 노드에서부터 `i`번째 노드를 제거하는 메서드이다. 마찬가지로 재귀적으로 구현한다.
```rust
pub fn remove(&mut self, i: usize) -> Option<T> {
    if i == 0 {
        let next = self.next.take().expect("Index out of range");
        let ret = next.data;
        self.next = next.next; // link to up next
        ret
    } else{
        self.next.as_mut().unwrap().remove(i-1)
    }
}
```


## `get()`
역시 재귀적으로 구현된, 현재 노드에서부터 `i`개 뒤의 노드에 저장된 값을 가져오는 메서드이다. 가능한 범위를 벗어나면 `expect`가 에러를 발생시킨다. 여기서 `self.next`는 `Option<Box<Node<T>>>` 타입으로 되어 있는데, 특이하게 `as_ref()`를 먼저 해도 Option 안에 있는 `Box<Node<T>>`가 `Box<&Node<T>>`로 바뀐다. 그 다음에 `expect()`로 `Option`을 벗겨주면 되는 것 같다. 최종적으로 `next`는 `Box<&Node<T>>` 타입이 된다. 여기에 dereference를 한번 해주면(`*next`) `&Node<T>`가, 한번 더 해 주면(`**next`) `Node<T>`가 될 것이다.

그런데, rust는 c++이었으면 `(*next).data`을 써주거나 `next->data`와 같이 써줘야 했을 것들을 자동으로 해준다. 따라서 아래 코드에서 `next.data`는 실제로는 `(**next.data)`를, `next.get(i-1)`은 원래 call-by-reference이니 `(*next).data`를 의미한다.

```rust 
pub fn get(& self, i: usize) -> Option<T> {
    let next = self.next.as_ref().expect("Index out of range");
    match i {
        0 => next.data,
        _ => next.get(i - 1)
    }
}
```

# LinkedList
이제 연결리스트를 구성하는 노드를 구현하였으니, `LinkedList`에 필요한 메서드들을 적절하게 구현해주면 된다. 이미 위에서 대부분의 할 일을 다 해두었기 때문에 실제로 구현할 것은 많지 않다.

```rust
pub struct LinkedList<T: Copy> {
    head: Box<Node<T>>,
    size: usize
}
```
연결리스트는 노드 중 맨 앞에 오는 것, 즉 `head`의 주소만 저장해두면 된다.

```rust
impl<T: Copy> LinkedList<T> {
    pub fn new() -> LinkedList<T> {
        LinkedList::<T> { head: Box::new(Node{data:None, next: None}), size: 0 }
    }
    // other methods...
}
```
맨 앞의 노드(`head`)는 더미 노드이다. 연결리스트가 비어있더라도 항상 데이터를 가지지 않는 하나의 노드는 맨 앞에 오게 되어 있다.

예외처리만 따로 해주고, `Node`의 각 메서드를 그대로 사용해서 다음과 같이 구현하면 된다.

```rust
impl<T: Copy> LinkedList<T> {
    pub fn add(&mut self, i: usize, data: T) -> bool {
        if i > self.size {
            false   // Index out of range
        } else {
            self.head.add(i, data);
            self.size = self.size + 1;
            true
        }
    }
    pub fn append(&mut self, data: T){
        self.head.append(data);
        self.size = self.size + 1;
    }
    pub fn remove(&mut self, i: usize) -> Option<T> {
        if i >= self.size {
            None    // Index out of range
        } else {
            self.size = self.size - 1;
            self.head.remove(i)
        }
    }
    pub fn get(&self, i: usize) -> Option<T> {
        self.head.get(i)
    }
    pub fn len(&self) -> usize {
        self.size
    }
    pub fn is_empty(&self) -> bool {
        self.size == 0
    }
}
```