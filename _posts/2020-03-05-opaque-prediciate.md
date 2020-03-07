---
layout: post
title: Opaque predicate
date: 2020-03-05
Author: yarncha
categories:
tags: [난독화, 태그수정가능]
comments: true
---

1. Opaque predicate란?
----------
한 방향으로 실행되는 조건문을 만들고, 실행되지 않는 부분에 쓰레기 코드를 삽입하는 대표적인 난독화 방법 중 하나이다. 이는 프로그램 일부를 최적화하는 것을 어렵게 하고, 제어 흐름을 복잡하게 만듦으로써 프로그램 분석을 방해할 수 있다.[^1][^2]

3가지 유형으로 구분할 수 있다.[^3]
  1. Invariant Opaque Predicates
  어떤 값을 넣어도 같은 결과가 나오는 분기[^5]를 조건문에 작성하는 방법. 잘 알려진 명제를 활용하여 작성한다. 이는 fuzzing[^4]을 통해 어떤 결과가 나오지 않는지를 확인할 수 있다.

[^4]: Fuzzing : 임의의 데이터를 input으로 넣어보면서 프로그램의 작동을 확인하는 방법.

[^5]: x2는 항상 양수가 되는 것과 같은 수학적 명제

  2. Contextual Opaque Predicates
  어떠한 전제 조건에서는 항상 같은 값을 낼 수 있는 분기를 조건문에 작성하는 방법. 전제 조건의 값만 알아도 어떤 조건을 타게 되는지 알 수 있다.
  예를 들면, 다음과 같은 그래프에서는 x>1일 경우 항상 양수이므로, x>1을 전제 조건으로 주는 것과 같다.
  ![]()

  3. Dynamic Opaque Predicate
  실행할 때마다 분기문을 타는 방향은 다르지만, 결과는 항상 같게 되는 방법이다. Correlated와 adjacent의 두 가지 성질을 가진다.


2. 적용 예시
----------
```
int main(int a, int b)
{
  int result = 0;
  result = a + b;
  return result;
}
```
위의 코드는 두 parameter를 받아서 더하는 간단한 코드이다. 이를 앞에서 서술한 3가지의 방법으로 난독화 해 보았다.




  [^1]: Dongpeng Xu, Jiang Ming, Dinghao Wu, "Generalized Dynamic Opaque Predicates: A New Control Flow Obfuscation Method" The Pennsylvania State University
  [^2]: Geneviev` e Arboit, "A Method for Watermarking Java Programs via Opaque Predicates", McGill University
