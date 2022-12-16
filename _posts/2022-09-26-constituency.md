---
layout: post
title: Constituency Test
date: 2022-09-26 19:29:00 +0900
description: 문장 안에서 한 덩어리로 묶일 수 있는 것은 어디까지일지, 이를 판별하는 방법에는 무엇이 있는지 알아본다
tags: linguistics syntax
giscus_comments: true
---

**Constituent(구성요소)** 란 문장에서 하나의 덩어리(chunk)로 다뤄질 수 있는 부분들을 말한다. 나무그림을 그리게 되면, 구성요소는 하나의 node 아래로 묶이는 것들이 된다.

예를 들어서, "A child read a book"이라는 문장을 보자. 여기에서 "a child", "a book" 등은 constituent이지만 "child read"는 constituent가 아니다. 이런 경우에는 비교적 쉽게 constituency를 판별할 수 있으나, 일반적으로 어떤 문자열이 constituent인지, 어떤 문자열이 아닌지 판별하는 것은 까다로운 문제이다. 이를 위해 여러 테스트들이 고안되어 있다. 조심해야 할 점은 각 test는 충분조건이지 필요조건은 아니라는 점이다. 즉, test가 실패한다고 해도 해당 문자열이 constituent가 아니라고 보장할 수는 없다. 따라서 최대한 많은 test들을 모두 적용해보아야 한다.

# Substitution
문장의 한 부분을 한 단어(또는 두 단어)로 치환하여 문장이 의미를 보존하면서도 문법적인 문장을 유지하는지 말이 되는지를 판별하여 constituency를 판별할 수 있다. Substitution을 통한 constituency test 방법으로는 대표적으로 **pronominalization, there-substitution, one substitution, do so substitution**이 있다. 

## Pronominalization
말 그대로 문장의 일부분을 대명사(I, me, you, she, her, he, him, it, they, them, we, us)로 바꾸어 말이 되는지를 보는 방법이다. 구를 대명사로 치환하는만큼 **DP(Determiner Phrase)에 대해서만** 사용할 수 있다. 다음 문장을 보자. 
> This girl in the red coat will put a picture of Bill on your desk before tomorrow.

여기에서 This girl in the red coat를 she로 치환하면 다음 문장이 된다.
> **She** will put a picture of Bill on your desk before tomorrow.

치환된 문장이 문법적인 것으로 보아, "The girl in the red coat"는 원 문장의 constituent가 맞다는 것을 확인할 수 있다. 마찬가지로 다음과 같은 치환이 가능하다.

> * This girl in the red coat will put **it** (a picture of Bill) on your desk before tomorrow.
 * This girl in the red coat will put a picture of Bill **there** (on your desk) before tomorrow.
* This girl in the red coat will put a picture of **him** (Bill) on your desk before tomorrow.

이를 통해 각각 "a picture of Bill", "on your desk", "Bill"이 constituent라는 사실을 알 수 있다. 반면 This girl을 she로 치환한 다음 문장을 보자.
> ***She** in the red coat will put a picture of Bill on your desk before tomorrow.

*은 비문법적인 문장을 표시할 때 쓰는 기호이다. 위의 문장이 비문이 된다는 사실로부터, "This girl"은 문장의 constituent가 아니라는 사실을 알 수 있다.

## _There_-substitution
**PP(prepositional phrase, 전치사구)에 대해** 사용할 수 있는 치환방법이다. 위의 문장에서 예시를 들어보자면, 전치사 _on_이 이끄는 PP _on your desk_를  there로 치환할 수 있을 것이다.
> This girl in the red coat will put a picture of Bill **there** before tomorrow.

따라서 _on your desk_는 문장의 constituent이다. 한편,  _in the red coat_를 치환하면 다음과 같다.
> This girl **there** will put a picture of Bill on the desk before tomorrow.

위 문장을 보면 문법적으로 말이 되기는 하지만, 원래 문장의 의미를 보존하지는 않는다. 원래 "빨간 코트를 입은 소녀"라는 의미를 가지고 있던 것이 "그곳에 있는 소녀"라는 뜻으로 바뀌었기 때문이다. 사실 _in the red coat_는 실제로는 문장의 constituent가 맞지만, there-substitution을 통해서는 이런 결론을 내릴 수 없다. 즉, 다른 test를 적용해야 한다.

## _one_-substitution for NPs
**NP(noun phrase, 명사구)**에 대해서 적용할 수 있는 방법으로, _one_ 또는 _ones_로 치환하는 방법이다. 즉, pronominalization과 달리 **DP에 대해서는 적용할 수 없다.** 앞의 문장을 다시 가져오자.

> This girl in the red coat will put a picture of Bill on your desk before tomorrow.

_girl in the red coat_를 one으로 치환하면 다음과 같다.

> This **one** will put a picture of Bill on your desk before tomorrow.

또한, _coat_를 다음과 같이 one으로 치환하는 것도 가능하다.

> This girl in the red **one** will put a picture of Bill on your desk before tomorrow.

즉, _girl in the red coat_와 _one_은 문장의 constituent이다. 이번에는 다른 문장을 가져와보자.
> That tall student of linguistics from Brussels will read this new novel quickly.

이 문장으로부터 다음의 치환이 모두 가능하다.
>* That **one** (tall student of linguistics from Brussels) will read this new novel quickly.
* That tall student of linguistics from Brussels will read this **one** (new novel) quickly.

따라서 즉, 치환된 부분이 각각 문장의 constituent라고 할 수 있다.

## do (so) substitution for VPs
**do so substitution**은 VP에 대해 사용할 수 있는 test로, 앞선 방법들과는 달리 두 단어로 치환을 하게 된다. 예시로 두 명의 화자 A, B가 다음과 같은 대화를 나누고 있다고 가정하자.
> A: That girl in the red coat will not _put a picture of Bill on your desk_
B: Yes, but this girl in the red coat will **do so**.

B의 문장을 보면, _do so_가 _put a picture of Bill on your desk_를 대체하는 것을 알 수 있다. 따라서 _put a picture of Bill on your desk_는 문장의 constituent이다. 한편, B의 문장에서 do so가 없어도 문장의 의미에 문제가 없는 것을 알 수 있다. do so를 생략한다면 이는 후술할 **VP ellipsis**의 예시가 된다.

# Ellipsis/Deletion

>A: This girl in the red coat will put a picture of Bill on your desk before tomorrow.
B: Yes, but this girl in the red coat will __ before tomorrow.

위는 앞서 말한 **do so substitution**에서 _do so_를 생략한 문장이다. 즉, 빈칸에 원래 들어가야 할 _put a picture of Bill on your desk_가 생략되어 있는 것으로 볼 수 있다. 반면, 다음과 같은 생략은 불가능하다.
>* *This girl \_\_ coat will put a picture of Bill on your desk before \_\_.
* *__girl in the red coat will \_\_ a picture of Bill on your desk before tomorrow.

따라서 이들은 constituent가 아니다.

# Coordination
coordination은 앞서 다룬 substitution과는 달리 두 phrase(구)를 conjunction(and, or, nor 등)으로 연결하는 방법이다. 만약 두 phrase를 연결할 수 있다면 두 phrase는 각각이 문장의 constituent라고 할 수 있으며, 같은 syntactic category에 속한다고 할 수 있게 된다.

>* The girl in the rain coat will put a picture of Bill on your desk before tomorrow.

위의 문장에서, 다음과 같은 phrase를 삽입해도 문법적인 문장이 되는 것을 알 수 있다.
>* The girl in \[the rain coat\] **and \[the black shoes\]** will put a picture of Bill on your desk before tomorrow.

conjunction을 통해서 다른 phrase와 연결된다는 사실을 통해서 _the rain coat_가 문장의 constituent라는 점을 알 수 있다. 또, _the black shoes_는 DP이므로 _the rain coat_ 또한 DP이다. 

Coordination test는 substitution과 매우 유사하지만, 추후 X-bar theory를 배우면 알게 될 $\bar{T}$과 같은 구성요소에도 적용이 가능하다는 점이 있다. 예를 들어서, 다음과 같은 phrase에 대한 constituency test는 substitution을 통해서는 불가능하다.
> This girl in the red coat \[will have her breakfast\] **and \[will put a picture of Bill\]** on your desk.

여기에서 _will have her breakfast_는  modal(조동사) will이 이끄는 $\bar{T}$로, 한 단어로 치환하여 문법성을 보존할 수는 없다. 그러나 coordination test를 통해 이것이 constituent라는 사실을 알 수 있었다.

## Coordination test의 예외
다음 문장을 보자.
> They play unusual music, and I listen to unusual music.

이는 실제 발화에서 다음과 같이 축약될 수 있을 것이다.
>\[They play\], and \[I listen to\] unusual music 

위 문장만 보면 _they play_와 _i listen to_가 conjunction을 통해 연결되므로 각각이 문장의 constituent라고 생각할 수 있을 것이다. 그러나 이는 예외적인 경우로, **right node raising**이라는 현상에 해당된다. 이는 coordination test가 실패하는 대표적인 경우이다.

Coordination test의 또 다른 예외로는 **gapping**이 있다. 이는 Ellipsis와 Coordination의 조합으로 생각할 수 있다. 예를 들어서, 
> John will go to the movies and Sue __ to the theatre

라는 문장을 보면, 빈칸에 _will go_가 생략되어 있는 것으로 볼 수 있다. 앞서 살펴본 Ellipsis test에 의하면 생략이 가능한 phrase는 constituent여야 하지만, 실제로 _will go_는 constituent가 아니다. 즉 이는 예외적인 경우에 속한다. 

# Movement / Displacement
Movement는 문장에 특정 종류의 distortion을 줌으로써 constituent structure를 알아내는 것이다. 
## Topicalization
> The girl in the rain coat will put a picture of Bill on your desk \[before tomorrow\].

위의 문장에서, _before tomorrow_를 앞으로 가져온 다음의 문장을 생각해보자.

> **\[Before tomorrow\]**, the girl in the rain coat will put a picture of Bill on your desk.

위의 문장은 "내일 안에 무슨 일이 있을까?" 하는 질문의 대답이 될 수 있을 것이다. _Before tomorrow_를 맨 앞으로 가져옴으로써 _Before tomorrow_는 대화의 **주제**이며, _the girl in the rain coat will put a picture of Bill on your desk_가 새로운 정보라는 사실을 암시하게 된다. 이 때문에 위와 같은 변형을 **topicalization**이라고 한다. 이는 **constituent에 대해서만 적용이 가능하다**.
위의 경우에서 topicalization은 PP에 적용되었으나, DP, PP, VP, NP 등 다양한 constituent에 대해 적용될 수 있다.
> DP: **The picture of Bill**, this girl in the red coat will put (the picture of Bill) on your desk before tomorrow.
PP: **On your desk**, this girl in the red coat will put a picture of Bill (on your desk) before tomorrow.
VP: **Put a picture of Bill on your desk**, this girl in the red coat will (put a picture of Bill on your desk) before tomorrow.

반면 다음과 같이 constituent가 아닌 경우에 대해서는 topicalization을 적용할 수 없다.
>\***Girl in the red coat**, this (girl in the red coat) will put a picture of Bill on your desk before tomorrow

## Clefting 
>It's X that Y; X: focused

누군가가 
> The girl in the rain coat will put a picture of Bill on your desk before tomorrow.

라는 문장을 말했다고 할 때, 다른 사람이 
>It was **before Tuesday** that the girl in the red coat will put a picture of Bill on your desk.

와 같이 정정해준 상황을 생각해보자. 이 경우, that 이후에 나오는 것은 오래된 정보(presupposition)이 되는 것이고, _before Tuesday_가 이 사람이 말하고 싶은 focus가 된다. 이와 같이 변형하는 것을 **clefting**이라고 한다. 마찬가지로 이는 constituent일 때에만 적용이 가능하며, 특히 focus가 **DP와 PP에 대해서만 적용이 가능**하다. 

## Pseudo-clefting 
>What X was Y; Y: focused

Pseudo-clefting은 **What X was Y**의 형태를 갖는 변형으로,  clefting과 마찬가지로 문장의 특정 constituent에 focus를 주는 기능을 한다. Clefting과 마찬가지로 focused element는 문장의 constituent여야만 하며, what이 아닌 who, where 등으로는 사용할 수 없다.
유사하나 **VP, AdjP, CP**에 대해서 적용이 가능하다.

> What the girl in the red coat will do is **put a picture of Bill on your desk tomorrow**.

위 문장에서 _put a picture of Bill on your desk tomorrow_는 VP로, 문장의 constituent가 된다.

다음 예시를 보면 clefting과 pseudo-clefting의 차이를 알 수 있다.
>Mary believes that John is incompetent.
*It is that John is incompetent that **Mary believes**. (clefting)
What Mary believes is that **John is incompetent**. (pseudo-clefting)

위에서 _Mary believes는 constituent_가 아니기 때문에 clefting이 불가능하다. 반면 _John is incompetent_가 constituent라는 것을 pseudo-clefting으로부터 알 수 있다.

## Wh-movement
> The girl in the rain coat will put a picture of Bill on your desk before tomorrow.

위 문장에서 _The girl in the rain coat_를 who로 치환하면 다음과 같이 의문문이 된다.
> **Who** will put a picture of Bill on your desk before tomorrow?

마찬가지로 _a picture of Bill_을 _what_으로 치환하고 순서를 바꿔서
>**What** will the girl in the rain coat will put on your desk before tomorrow?

위와 같이 Wh-로 치환 후 순서를 바꿔 의문문으로 만들 수 있으면 문장의 constituent이다. 이를  **Wh-movement**라고 한다.


## Heavy-NP shift

Heavy-NP shift는 특이하게도 constituent의 길이가 길 때만 일어나는 현상이다. 다음 문장을 보자.
> I sent it to you.

다음과 같이 it (또는 다른 NP)을 맨 뒤로 shift시켜보자.
>*I sent to you **it**.
*I sent to you **recipes**.
?I sent to you **the recipes from the paper**.

위와 같이 짧은 NP에 대해서는 NP-shift가 문법적으로 오류가 있는 문장을 만들어낸다. 하지만, 아래와 같이 긴 NP에 대해서는 (heavy-NP) 문법성에 문제가 생기지 않는다.
>  I sent to you **the recipes from the paper that I told you about yesterday**

이를 heavy-NP shift라고 한다. 앞서 소개한 문장의 변형방법들과 마찬가지로, heavy-NP shift 또한 constituent들에 대해서만 적용이 가능하다.

# 참고문헌

Koopman, H., Sportiche, D., & Stabler, E. (2013). An introduction to syntactic analysis and theory. John Wiley & Sons.