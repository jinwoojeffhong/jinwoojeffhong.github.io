---
layout: post
title: Dependency Parsing
date: 2019-04-07T00:00:00.001Z
categories: update
---
<img src="/images/dependency_parsing/coffee.jpg" class="fit image">


# 1. 일반적 문장분석방법론

 언어학에서 문장을 분석하는 방법으로 크게 (1)Constituency Parsing과 (2)Dependecy Parsing 두가지 방식이 존재한다. 두 방법들 모두 분석하고자 하는 문장을 문장을 구성하는 하위 부분(Part)들로 구분한다는 점에서는 동일하나 분할규칙**(Division Method)** 에서 차이를 보인다.

Constituency Parsing은 문장의 문법구조에 기반한 분석방식이다. Constituency Parser는 문장을 하위-구(Sub-Phrase)로 쪼개며, 구체적으로 명사구(Subject Noun Phrase)와 동사구(Predicate Verb Phrase)로 분리한다. 반면 Dependency Parser는 문장 내 두 단어의 의존관계에 집중한다. 두 방식모두 이러한 분할규칙에 따라 Parse Tree를 생성하는데 일단 Dependency Parser에 집중해보겠다. 두 단어 사이의 수식관계에 따라 수식단어를 Parent 노드로, 피수식단어를 Child 노드로 분리하고, 문장 내 동사(Verb)를  Tree의 제일 상단, 즉 Root로 설정한다.

또한 Constituency 내 단어 대응관계는 1대1 혹은 1대 多가 되지만, Dependency에서는 무조건 1대1이다.

두 방법을 일러스트로 표현하자면 다음과 같다.


### 1-1. Constituency Tree

                     Sentence
                         |
           +-------------+------------+
           |                          |
      Noun Phrase                Verb Phrase
           |                          |
         John                 +-------+--------+
                              |                |
                            Verb          Noun Phrase
                              |                |
                            sees              Bill



### 1-2. Dependency Tree

    							sees
                    |
            +--------------+
    subject |              | object
            |              |
          John            Bill



# 2. Dependency Parsing

문장(Sentence)를 입력받아 Dependency Tree를 출력한다. 즉 문장을 Tree로 매핑(Mapping)해주는 Parsing Model를 만드는 게 목적이다.

Parsing 모델을 만들기 위해서는 다음의 두 가지 문제를 해결해야 한다. 첫번째 문제는 Learning인데, Dependency Graph 처리된 문장 데이터셋으로 모델을 학습시켜, 새로운 문장이 들어왔을 때 올바른 Parsing을 하는 모델은 만들어야 한다. 두번째 문제는 Parsing이며 모델과 문장이 주어졌을 때 문장을 처리하는 최적 Dependency Graph를 도출해야 한다. 즉 Dependency Parsing을 잘하기 위해서는 모델(M)과 Dependency Graph(D)를 잘 만들어야 한다.



# 3. Transition-Based Dependency Parsing

Dependency Parsing(의존성 문장분석)에서도 여러 갈래가 존재하는데 그중 Transition-Based (전이기반)Dependency Parsing은 전이(Trasition)를 통해 문장을 Tree로 매핑하며 State라는 개념을 도입하여 State안에서 전이를 수행한다. 여기서 Learning은 과거 Transition 기록들에 기반해서 State 내 다음에 올 전이를 예측하는 문제이며, Parsing은 모델(M)이 주어졌을 때, 입력 문장의 최적의 Transition Sequence를 찾는 것이다.



# 4. Greedy Deterministic Transition-Based Parsing

이 방식은 2003년 Nivre라는 사람이 만들었는데, Transition-Based인 만큼 State Machine을 이용한다. State Machine은 State와 State들 사이의 Transition으로 이루어져 있으며, 모델은 initial State에서 Terminal State까지 도달하기 위한 일련의 Transition Sequence들을 만들어낸다.

이제 모델을 구성하는 State와 Transition에 대해 설명할 건데, 이해하기가 어렵다면 다음과 같은 비유를 들어보겠다. 우선 State를 구성하는 Stack, Buffer, Arc 3가지는 돼지고기, 버섯, 계란이라고 생각해보자. 그리고 Transition을 구성하는 SHIFT, LEFT-ARC, RIGHT-ARC 3가지를 레시피라고 생각하자. 어떤 레시피를 따르느냐에 따라 식재료를 손질하는 방식이 달라지듯, 어떤 Transition 타입을 따르느냐에 따라 Stack, Buffer, Arc를 이용하는 방식이 달라진다. 이제 State와 Transition을 살펴보자.

### 4-1. State

State는 Stack, Buffer, Arc Set 3가지로 구성되어 있으며, 각각 다음과 같이 표시한다.

$$$
c=(\sigma, \beta, A)
$$$

**1)Stack**: 단어를 쌓아놓는 함이며, Initial State에서는 Root만 존재하나 Terminal State에서는 모든 단어를 다 담고 있다.
**2)Buffer**: 역시 단어를 쌓아놓는 함이며, Buffer에서 단어를 꺼내 Stack에 쌓는다. Initial State에서는 Root를 제외한 모든 단어를 담고 있으며, Terminal State에서는 빈 함이 된다.(Empty)
**3)Arc Set:** 두 단어간 관계정보를 담은 Arc들의 집함이며, Arc는 단어 두개와 두 단어 간 관계정보를 표시하는 r로 구성되어있다. Arc의 노테이션은 A이다.

$$
A= (w_i, r, w_j)'s
$$

## 4-2. Terminal

Terminal 또한 3가지 Type이 있다.
1)SHIFT: Buffer의 첫번째 단어를 꺼내 Stack의 Top에 놓는 것
2)LEFT-ARC:  Arc Set에 A를 담는 것. A는 Stack Top에서 첫번째단어와 두번째 단어 간 관계정보이다. A를 Arc Set에 담은 후 Stack의 Top에서 첫번째 단어를 Stack에서 제외시킨다.
3)RIGHT-ARC: LEFT-ARC와 동일하게 수행하지만 여기서 첫번째단어와 두번째 단어가 다르다.

$$
LEFT-ARC: (w_i, r, w_j)
$$

$$
RIGHT-ARC: (w_j, r, w_i)
$$

- 여기서 w(i), w(j)는 각각 Stack의 Top에서 두번째 단어, 첫번째 단어를 의미한다. 즉 두 방식의 차이는 문장의 왼쪽부터 읽냐 오른쪽부터 읽냐의 차이라는 것을 알 수 있다.
- 여기서 흥미로운 점은 한국어 구문분석시 SHIFT 혹은 RIGHT-ARC만을 이용한다는 건데, 그 이유는 *[한국어가 대표적인 후위언어](http://bozon.nlp.wo.tc/?f=158c7e65826538)이기 때문이다. 다음 예시를 통해 이게 무슨 말인지 예시를 통해 쉽게 파악해보자.

> 철수가 밥을 먹었다.

> (철수←**가**) (밥←**을**) (먹-**었**-다.)

- 여기서 '가'는 '철수'라는 주격을, '을'은 '밥'이라는 목적격을 '었'은 동사가 과거 시제임을 알려준다. 이처럼 한국어는 영어같이 어순에 의해서가 아니라 기능형태소들이 명사의 구문적 기능, 동사의 시제, 어절 간 수식관계 같은 문법관계를 결정한다. 그리고 이러한 기능형태소들이 대부분 구의 오른쪽에 존재하기 때문에 오른쪽부터 읽는 것이라고 할 수 있다.
- Greedy-Determininstic의 의미는 문장을 볼때, 한번에 두 단어의 관계만을 고려한다는 의미인 것 같다.(확신하지 못함 ㅠ)

# 5. Neural Dependency Parsing

- Greedy Transition-Based Neural Dependency Parser는 기존의 전통적 방식인 Feature-Based Discriminative Dependency Parser에 비해 효율성과 성능 모두 높다. 그중에서도 이전 모델들과의 확연한 차이는 바로 Sparse feature representation보다 dense representation에 가깝다는 것이다.
- 여기서 소개할 Greedy Transition-Based Neural Dependency Parser는 Transition-Based라는 점에서 Arc 기반 전이 시스템이다. 또한 Greedy라는 점에서 한번에 하나의 Transition T(SHIFT, RIGHT, LEFT 3개 중 하나) 를 예측한다. 여기서 중요한 점은 Transition을 예측할 때 현재 구성(Current Configuration, c)에서 두 단어간 정보관계를 파악한 후 그에 맞는 행동절차(Transition)을 예측한다.
- 모델의 목표는 일련의 Transition Sequence를 예측하는 것이다.
