---
layout: post
title: "Modern C++ 문법 정리" 
date: 2023-01-14 01:00:00 +0900
description: C++11 이상에서 새로 도입된 대표적인 modern c++ 문법
tags: data-structure
giscus_comments: true
---

C++은 C++98 이후로 굉장히 오랫동안 정체되어 있다가, C++11 업데이트에서 많은 기능이 새로 도입되었다. 이 때문에 C++11 이후의 업데이트인 C++14, C++17, C++20 등을 **모던 C++**이라고 따로 지칭한다.
평소 알고리즘 문제를 풀 떄 정도는 modern C++ 기능들을 알 필요가 별로 없어서 따로 공부할 생각을 하지 않았지만 중요한 몇 개는 알고 있어야 할 것 같아서 정리해본다.

# `auto`와 `decltype`
## `auto`
변수를 선언할 때, 모던 C++에서는 특정 경우 자료형을 따로 지정하지 않아도 알아서 자료형을 추론한다. 단, 선언과 동시에 변수가 초기화되어야 한다. 그러지 않으면 자료형을 알 수가 없기 떄문이다.
```C++
auto var1 = 100;        // int
auto var2 = 100L;       // long
auto var3 = 100.0;      // double
auto var4 = "string";   // string
auto var5; //ERROR!
```

특히 iterator와 같은 것들을 귀찮게 적어줄 필요가 없어졌다.
```
vector<int> a;
// 기존
vector<int>::iterator it = a.begin();
// 모던 C++
auto it = a.begin();
```

## `decltype`
`decltype`은 함수처럼 사용되어서 감싸고 있는 표현의 타입을 알려준다.

```c++
auto a = 5;     // int
decltype(a) b = 3;      // int

int& r_a = a;
decltype(r_a) r_b = b;  // int&
```

값이 아닌 표현이 들어갈 경우에는 조금 더 복잡해지는데, 여기에 대해서는 이후에 추가하겠다.

# range-based loop
아마 파이썬에서 영향을 받아온 것 같다. 파이썬에서는
```python
array = [1, 2, 3, 4, 5]
for e in array:
    print(e)
```
처럼 iterable의 원소들을 하나씩 가져와서 매우 쉽게 사용할 수 있다. 기존 C++에서는 이를 위해서는 귀찮게 `iterator`를 사용해야 했다.
```c++
vector<int> v
for(vector<int>::iterator it = v.begin(); it < v.end(); ++it){
    cout << *it << endl;
}
```
하지만 Modern C++에서는 python과 같은 range-based loop를 사용할 수 있다.
```c++
vector<int> v = {1, 2, 3, 4, 5};
for(auto e: v){
    cout << e << endl;
}
```
값을 복사하는 대신 reference를 사용해 가져올 수도 있다.
```c++
vector<string> v = {"hello", "modern", "C++"};
for(auto& e: v){
    cout << e << endl;
}
```

C++17부터는 pair로 이루어진 list를 아래와 같이 풀어서 쓸 수도 있다. 점점 파이썬과 비슷해지는 것 같다.
```c++
vector<pair<int, string>> v = {make_pair(1, "hello"), make_pair(3, "modern"), make_pair(6, "C++")};
for(auto& [i, st]: v){
    cout << i << " " << st << endl;
}
```

# 람다 함수
파이썬에서처럼 람다 함수(익명 함수)를 사용할 수 있게 되었다. 예를 들어서, 파이썬에서는 다음과 같은 표현이 가능했다.

```python
f = lambda x: x + 2
print(f(3)) # 5
```

특히 이는 내장 함수의 파라미터로 다른 함수가 들어가는 경우에 유용하게 사용할 수 있었다. 예를 들어서 정렬을 할 때, 우리가 정한 함수의 기준으로 정렬을 하고 싶을 수 있다. 그럴 때 우리는 이렇게 써줄 수 있었다.

```python
sorted_list = sorted(some_list, key=lambda x: x%12)
```

이 예시에서 sorted라는 내장함수는 key라는 함수를 받아서, 그 함수가 작은 순서대로 정렬을 하게 된다. 여기에서 굳이 함수를 이름붙여주고 `def`를 사용해 정의하기보다 저렇게 inline으로 써주는 것이 훨씬 간단하다. 마찬가지로 C++에서도 람다 함수를 사용할 수 있다. 다만 문법은 조금 다르다.

```c++
auto f = [] (int a, int b) {  cout << a+b << endl; };
f(3, 4);    // 7
```

각 부분의 의미를 살펴보면 이렇다.

* []: 캡처. 안에 들어가는 이름의 외부변수를 복사하거나 참조해온다.
* (): 매개변수
* {}: 함수의 동작

또한, `f`의 타입이 auto로 자동결정되는 것을 알 수 있는데 사실 `f`의 타입은 함수 포인터로, `void (*f)(int a, int b)`와 같은 형식이 된다. 이를 굳이 적어주기는 번거로우니 `auto`를 사용하면 된다.

매개변수와 함수의 동작은 기존 함수와 다를 것이 없다. C++의 람다 함수에서 처음 나오는 개념은 캡처라는 것이다. 이는 함수 외부에서 정의된 상수나 변수를 람다 함수 안에서 사용해주기 위해서 적어주는 것이다. 예를 들어서 다음과 같이 사용할 수 있다.

```c++
int main(){
    int val1 = 3;
    int val2 = 5;

    auto f = [val1, val2] (int a, int b){
        cout << val1 << " " << val2 << endl;
        cout << a << " "  << b << endl;
    }
    return 0;
}
```

`val1`과 `val2`를 복사하지 않고 reference를 통해 가져오는 것도 가능하다. 
 
```c++
auto f = [&val1, &val2] (int a, int b){
    cout << val1 << " " << val2 << endl;
    cout << a << " "  << b << endl;
}
```

만약 모든 변수를 가져오고 싶다면 다음과 같이 하면 된다.
```c++
auto f = [=] (int a, int b){                // 모든 변수를 복사해서 가져온다
    cout << val1 << " " << val2 << endl;
    cout << a << " "  << b << endl;
}

auto f = [&] (int a, int b){                // 모든 변수를 reference로 가져온다
    cout << val1 << " " << val2 << endl;
    cout << a << " "  << b << endl;
}
```

# 스마트 포인터
C++11에서는 `std::auto_ptr`이 처음 도입되었지만, 이는 더 이상 사용되지 않는다. 대신 `std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`가 도입되었다. 이는 러스트에서 사용하는 소유권의 개념과도 매우 비슷하다.

## `std::unique_ptr`

`unique_ptr`은 기존 포인터와 달리, 가리키는 대상을 자신이 "소유"하는 것으로 볼 수 있다. 

```c++
auto a = make_unique<int> (5);
// 또는
auto a = unique_ptr<int>{new int {5}};

cout << *a << endl; // 5
```
위와 같이 정의해주면 `a`는 이름에 맞게 5를 가리키는 "유일한 포인터"가 된다. 즉 `a`를 통하지 않고는 5에 접근할 수가 없다. 즉 아래와 같이 `a`를 복사하는 것이 불가능하다.

```c++
auto b = a;     // compile error
```

왜냐하면 `a`만이 5를 가리키는 유일한 포인터여야 하기 때문이다. 대신 소유권을 이전할 수는 있는데, 그러면 `a`로는 더 이상 5에 접근할 수 없게 된다.
```c++
auto b = move(a);
```

## `std::shared_ptr`

Shared pointer의 경우 reference count 방식으로 메모리를 관리한다. 이름에서 알 수 있듯이, `unique_ptr`와는 달리 여러 포인터가 한 객체를 가리키는 것이 가능하다. 여기에서 reference count라는 말은, 특정 객체를 가리키는 포인터의 개수를 세고 있다가 0이 되는 순간 메모리를 해제해준다는 뜻이다.

```c++
auto a = make_shared<int>(5);
auto b = a;
cout << *a << " " << *b << endl;        // 5 5
```

한편, reference counter 방식의 포인터는 <a href = "/blog/2023/Rust-Pointer/">이 링크</a>에서 설명해두었듯이 두 포인터가 서로를 참조하는 경우 영원히 메모리에서 삭제되지 않게 된다. 이를 방지하기 위해서 `std::weak_ptr`를 함께 사용해준다.