> #### 키

-  **각 행을 고유하게 식별하기 위해 사용되는 속성 혹은 속성들의 조합**
- **다른 튜플들과 구별할 수 있는 유일한 기준이 되는 속성**
- 키는 데이터베이스에서 데이터 무결성과 검색 효율을 높여준다.
- ex) 기본 키, 후보 키, 대체 키, 복합 키, 외래 키

> ####  기본키

- 테이블에서 **가장 대표적인 식별자 역할**을 하는 키(Primary Key)
- **한 테이블에 오직 하나만 존재**한다.
- **NULL 값을 가질 수 없고 중복을 허용하지 않는다.**

> #### 외래키

- **다른 테이블의 기본 키를 참조**하는 키(Foreign Key)
- 테이블과 관계를 맺을 때 사용한다.
- 외래 키를 통해 참조 무결성을 유지할 수 있다.

> #### 복합 키

- **두 개 이상의 컬럼을 결합해 하나의 키로 사용**하는 것(Composite Key)

> #### ER 다이어그램

- 데이터베이스를 설계할 때 **개채와 개체들 간의 관계를 시각적으로 표현한 다이어그램**
- Entity Relationship Diagram의 약자


> ## 연관관계

- **데이터베이스에서 두 개 이상의 개체 간의 의미 있는 연결이나 관계**
- 관계에 따라 1:1, 1:N, N:1, N:N의 관계로 표현된다.
- 연관관계를 통해 현실 세계의 데이터 간의 상호 작용을 구체적으로 나타낼 수 있다.

> #### 정규화

- 데이터를 구조화하여 **중복을 최소화하고**, **데이터 무결성을 유지하는 것**
- 테이블을 어떻게 분해하는지에 따라 정규화 단계가 달라진다.
- **제1 정규화**
    - **테이블의 컬럼이 원자값을 갖도록 테이블을 분해하는 것**
    - **원자 값**
        - 한 필드에 저장되는 값이 더 이상 분해될 수 없는 최소 단위인 것
    - ex) 한 컬럼에 여러 값이 들어있는 경우를 분리한다.
- **제2 정규화**
    - 제1 정규화를 만족하며, **완전 함수 종속을 만족**하도록 테이블을 분해한 것
    - **완전 함수 종속**
        - 한 속성이 후보 키 전체에 의존하는 것
        - ex) 두 개의 속성 A, B가 합쳐져 후보 키가 되었을 때, 다른 속성 C가 A와 B 둘 다에 의존하지만, A나 B 중 하나에만 의존하지 않는 것
        - 만약 C가 후보 키의 일부에만 의존한다면, 이는 제2 정규형 위반의 원인이 된다
- **제3 정규화**
    - 제2 정규화를 진행한 테이블에 대해 **이행적 종속을 없애는 것**
    - **이행적 종속**
        - A->B, B->C가 성립할 때, A->C가 성립되는 것
- **BCNF 정규화**
    - 제3 정규화를 진행한 테이블에 대해 **모든 결정자가 후보키가 되도록** 테이블을 분해하는 것
    - 즉, 속성 A가 속성 B를 결정하고 있다면, A는 후보 키여야 한다.
    - 만약 후보 키가 아닌 속성이 결정자 역할을 한다면, BCNF 정규화를 위반한다.
    - 장점: 데이터 중복과 이상 현상을 철저히 제거할 수 있다.
    - 단점: 테이블 분해 후 조인이 복잡해질 수 있다.

> #### 반 정규화

- **일부 중복을 허용하여 쿼리 성능을 향상시키는 전략**
- 왜 필요할까?
    - -> **정규화된 구조는 여러 테이블 간의 조인 연산이 빈번하게 발생**해 읽 성능이 저하될 수 있어서
- 장점
    - 조회 성능이 향상된다.
- 단점
    - 중복 데이터가 존재하므로, 데이터 일관성 유지 및 갱신 작업이 복잡해질 수 있다.
- -> **읽기 작업이 빈번하고, 업데이트 빈도가 낮은 경우에 고려할 만 하다.**