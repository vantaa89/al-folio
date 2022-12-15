---
layout: post
title: 네트워크 유량 문제와 Ford-Fulkerson 알고리즘
date: 2022-09-15 00:00:00 +0900
description: 
categories: Algorithm
giscus_comments: true
---

**포드-풀커슨 알고리즘**은 **네트워크 유량(network flow)** 문제를 해결하기 위한 대표적인 알고리즘이다.  먼저, 네트워크 유량 문제가 무엇인지부터 살펴보자.

## 네트워크 유량 문제
네트워크 유량 문제는 최대 유량(maximum flow)이라고도 한다. 이는 "그래프의 소스(source)에서 싱크(sink)로 흐를 수 있는 최대의 유량을 찾는 문제"라고 할 수 있다. 여기에서 그래프의 간선은 물이 흐를 수 있는 배관과도 같다. 각각의 배관은 용량(capacity)이 있어서, 주어진 용량 이상으로는 물이 흐를 수 없도록 되어 있다. 
<p align="center" style="color:gray">
<img src="https://velog.velcdn.com/images/vantaa89/post/0d2911b3-12d6-4013-bdf8-a6a7eb9df439/image.jpeg" width="50%" height="50%" align="center"/>
<br>
  네트워크 유량 문제
</p>
간단한 예시로 위의 그림을 보자. 여기에서 s가 소스, t가 싱크이다. 동그라미는 정점, 화살표는 간선을 나타내며, 각 간선 옆에 붙어있는 수는 그 간선의 용량이다. 위 그림의 경우에 s에서  t로 흐를 수 있는 최대의 유량은 15이다. s에서 u로 10, s에서 v로 5가 흐르고, u에서 v로 5가 흐르며 u와 v가 각각 t로 5와 10씩을 전달해주면 되기 때문이다.

네트워크 유량 문제를 조금 더 깔끔하게 표현해보자.

- $$G=(V, E, c)$$: 주어진 그래프
    - $$V$$: 그래프에 속하는 정점들의 집합
    - $$E \subseteq V^2$$: 간선들의 집합
    - $$c: E\to \mathbb{R}$$: 각 간선의 용량
- $$f: E\to \mathbb{R}$$: 각 간선에 흐르는 유량(flow). 세 가지 성질을 만족해야 한다.
    - $$f(u, v) \le c(u, v)$$ (capacity constraint)
    - $$f(u, v) = -f(v, u)$$ (skew-symmtry)
    - $$\forall u \in V: u \ne s, t \Rightarrow \sum_{w\in V} f(u, w) = 0$$ (flow conservation)
    
$$f$$가 가져야 될 세 가지 성질을 살펴보자면, 먼저 유량은 주어진 간선(파이프)의 용량을 넘어서는 안된다. (capacity constraint) 또, $$u$$에서 $$v$$로 10만큼이 흐르는 것은 반대로 $$v$$에서 $$u$$로 -10만큼이 흐른다고도 표현할 수 있다. (skew-symmtry) 마지막으로, 소스와 싱크를 제외한 나머지 정점들에 대해서는 들어오는 유량과 빠져나가는 유량이 같아야 한다. (flow conservation) 이때 주의할 점으로는 $$E$$는 항상 양방향으로 되어 있어야 한다는 것이다. 즉, $$(u, v)\in E \Leftrightarrow (v, u) \in E$$이다.

다시 말해, **네트워크 유량 문제는 $$G= (V, E, c)$$가 주어질 때 $$f: E\to \mathbb{R}$$을 찾는 문제이다**. $$f$$를 안다면 소스에서 싱크로 흐르는 총 유량 또한 쉽게 구할 수가 있기 때문이다.

## 포드-풀커슨 알고리즘
포드-풀커슨 알고리즘은 greedy한 방법으로 네트워크 유량 문제를 해결하는 한 방법이다. 먼저, 주어진 그래프 $$G= (V, E, c)$$와 유량 $$f: E\to \mathbb{R}$$에 대해서 **residual network** $$G = (V, E, r)$$를 정의한다. 복잡하게 표현했지만, 각 간선의 용량에서 이미 흐르고 있는 양을 빼고 더 흐를 수 있는 양(잔여 용량)
$$
r(u, v) = c(u, v) - f(u, v)
$$ 
을 계산해 놓은 것에 불과하다. 예를 들어서 용량이 9인 간선에 4만큼의 유량이 흐르고 있다면, 이 간선의 잔여 용량(residual capacity)은 5라고 할 수 있을 것이다. 

포드-풀커슨 알고리즘을 통해 네트워크 유량 문제를 해결하는 과정을 직접 살펴보자.
<p align="center" style="color:gray">
<img src="https://velog.velcdn.com/images/vantaa89/post/8c6fcd2a-b354-4f40-a61d-ba149a898626/image.png" width="50%" height="50%"/>
<br>
주어진 그래프
</p>
위의 그래프를 생각해보자. 간선에 0/5와 같이 적혀있는 것은 용량이 5인 간선에 0만큼의 유량이 흐르고 있음을 의미한다.

먼저, BFS나 DFS를 사용해서 소스 s에서 싱크 t로 가는 경로를 아무거나 하나 찾는다. 그리고 **경로의 최대 유량**을 구한다.
<p align= "center" style="color:gray">
<img src="https://velog.velcdn.com/images/vantaa89/post/1c10da90-fc85-46c3-915e-c75851cc4948/image.png" width="50%" height="50%"/>
<br>
위의 경우 경로의 최대 유량은 5가 된다.
</p>

찾은 경로를 구성하는 각 간선에 "경로의 최대유량"만큼씩을 흘려보내준다.

<p align= "center" style="color:gray">
<img src="https://velog.velcdn.com/images/vantaa89/post/f146c567-f59c-4797-afb2-ab20f6478b75/image.png" width="50%" height="50%"/>
</p>

이제 다시 처음으로 돌아와 DFS나 BFS를 사용하여 s에서 t로 가는 경로를 하나 찾는다. 단, 잔여 용량이 0인 간선은 제외한다. 즉, 위 그림에서 S->A로 가는 간선은 사용할 수 없다.

<p align= "center" style="color:gray">
<img src="https://velog.velcdn.com/images/vantaa89/post/dc4d2c12-fcad-4ad7-af11-9547fb374c82/image.png" width="50%" height="50%"/>
<br>
  이 경우 간선의 최대 용량은 2이다.
</p>

마찬가지로 간선의 최대용량만큼을 각 간선에 흘려보내준다.
<p align= "center" style="color:gray">
<img src="https://velog.velcdn.com/images/vantaa89/post/3f26fcdc-2e90-4250-9052-4f9b9871d47c/image.png" width="50%" height="50%"/>
<br>
  이 경우 간선의 최대 용량은 2이다.
</p>

그래프를 보면 더 이상 S에서 T로 가는 경로를 찾을 수가 없다. 따라서 알고리즘을 종료하며, 이 그래프의 네트워크 유량은 7이 되는 것을 알 수 있다.

### 슈도코드
```
func NetworkFlow((V, E, c), (s, t)) // G = (V, E, c), (source, sink)
	answer := 0
	foreach (u, v) ∈ E
    	r(u, v) := f(u, v) // residual flow
    end 
    while exists path p from s to t
    	maxPathFlow := ∞
    	foreach (u, v) in p:
        	if r(u, v) < maxPathFlow
            	maxPathFlow := r(u, v)
            end
        end
        foreach (u, v) in p:
            r(u, v) := r(u, v) - maxPathFlow
            r(v, u) := r(v, u) + maxPathFlow
        end
    end
end
```

### 구현
네트워크 유량 알고리즘을 이용한 대표적인 문제인 [백준 알고리즘 17412번: 도시 왕복하기](https://www.acmicpc.net/problem/17412)의 풀이이다. 언어로 C++17을 사용하였다.

$$i$$번 도시와 $$j$$번 도시를 연결하는 길이 있으면 $$c(i, j)=1$$로, 그렇지 않으면 $$c(i, j) = 0$$으로 설정한 후 1번 도시에서 2번 도시로 가는 네트워크 유량을 구하면 된다. `while(true)` 안에서 BFS로 1번 도시에서 2번 도시로 가는 경로를 구하고, 경로를 찾을 수 없으면 루프에서 빠져나오도록 한다. BFS 과정에서 이전 노드를 `parent`에 저장해놓아 경로를 백트레이싱 할 수 있게 한다. 그 후 경로의 최대 유량을 구하고 현재의 유량을 업데이트한다.

<p align="center" style="color:gray">
<img src="https://velog.velcdn.com/images/vantaa89/post/c6778a42-a9ff-41b7-8585-3b957cfa04f0/image.png" width="70%" height="70%"/>
