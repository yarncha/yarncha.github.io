---
layout: post
title: Opaque predicate
date: 2020-03-05
Author: yarncha
categories:
tags: [난독화, 태그수정가능]
comments: true
---

# 1. Opaque predicate란?

  한 방향으로 실행되는 조건문을 만들고, 실행되지 않는 부분에 쓰레기 코드를 삽입하는 대표적인 난독화 방법 중 하나이다. 이는 프로그램 일부를 최적화하는 것을 어렵게 하고, 제어 흐름을 복잡하게 만듦으로써 프로그램 분석을 방해할 수 있다.[^1][^2]     3가지 유형으로 구분할 수 있다.[^3]

###  1.1. Invariant Opaque Predicates
  어떤 값을 넣어도 같은 결과가 나오는 분기[^5]를 조건문에 작성하는 방법. 잘 알려진 명제를 활용하여 작성한다. 이는 fuzzing[^4]을 통해 어떤 결과가 나오지 않는지를 확인할 수 있다.

###  1.2. Contextual Opaque Predicates
  어떠한 전제 조건에서는 항상 같은 값을 낼 수 있는 분기를 조건문에 작성하는 방법. 전제 조건의 값만 알아도 어떤 조건을 타게 되는지 알 수 있다.    예를 들면, 다음과 같은 그래프에서는 x>1일 경우 항상 양수이므로, x>1을 전제 조건으로 주는 것과 같다.
  ![graph](<\images\2020-03-05-opaque-prediciate_01.png>)

###  1.3. Dynamic Opaque Predicate
  실행할 때마다 분기문을 타는 방향은 다르지만, 결과는 항상 같게 되는 방법이다. Correlated와 adjacent의 두 가지 성질을 가진다.

  [^4]: Fuzzing : 임의의 데이터를 input으로 넣어보면서 프로그램의 작동을 확인하는 방법.
  [^5]: x2는 항상 양수가 되는 것과 같은 수학적 명제.

# 2. 적용 예시

  ```
  int main(int a, int b)
  {
    int result = 0;
    result = a + b;
    return result;
  }
  ```

  위의 코드는 두 parameter를 받아서 더하는 간단한 코드이다. 이를 앞에서 서술한 3가지의 방법으로 난독화 해 보았다.

###  2.1. Invariant Opaque Predicates

항상 같은 값이 나오는 조건을 a^2>=0으로 두고, return을 제외한 stmt가 나올 때마다 난독화하였다.

  ```
  int main(int a, int b)
  {
    int result = 0;
    if (a * a >= 0) {
      result = a + b;
    } else {
      int temp = 0;
      result = temp;
    }
    return result;
  }
  ```

###  2.2. Contextual Opaque Predicates

para>0일 경우는 para^2>1이므로, stmt를 para^2>1을 분기로 가지는 조건문으로 묶고 para>0인 (조건을 만족하는) para를 명시해 두었다.

```
int main(int a, int b)
{
  int result = 0;
  int para = 2;
  if (para * para >= 0) {
    result = a + b;
  } else {
    int temp = 0;
    result = temp;
  }
  return result;
}
```

###  2.3. Dynamic Opaque Predicate

para는 랜덤 값이 들어가도록 해 둔뒤, 첫 번째 조건문에서는 para>0, 두 번째 조건문에서는 para&lt;=0을 조건으로 둔다. 첫 번째와 두번째는 결국 true->false 또는 false->true일 수 밖에 없는데, 이 두 경우의 output이 같을 수 있도록 한다. true에서 result--;를 했으면 false에서는 result++;를 넣어주는 방식이다. 이는 앞의 두 방식과는 다르게, 종료하기 return문 전에 연산을 수행할 수 있게 하였다.

```
int main(int a, int b)
{
  int result = 0;
  result = a + b;
  int para = rand() % 9;
  if (para * para >= 0) {
    result--;
  } else {
    result++;
  }
  if (para * para < 0) {
    result++;
  } else {
    result--;
  }
  return result;
}
```

전부 코드를 읽고 단순 변환하는 단계에서 바꾸어 보았는데, 직접 바꿔준다는 느낌이 많이 들어서 변환이 이루어지는 단계가 좀더 이르면 좋을 것 같다. 예를 들어 파일을 읽고 따로 구조만 파악하여 바꿔준다는 느낌으로?

  [^1]: Dongpeng Xu, Jiang Ming, Dinghao Wu. (n.d.). Generalized Dynamic Opaque Predicates: A New Control Flow Obfuscation Method(College of Information Sciences and Technology). The Pennsylvania State University, n.p..
  [^2]: Genevi\`eve Arboit. (n.d.). A Method for Watermarking Java Programs via Opaque Predicates(School of Computer Science, Crypto and Quantum Info Lab). McGill University, n.p..
  [^3]: Jiang Ming, Dongpeng Xu, Li Wang, Dinghao Wu. (n.d.). LOOP: Logic-Oriented Opaque Predicate Detection in Obfuscated Binary Code(College of Information Sciences and Technology). The Pennsylvania State University, n.p..
