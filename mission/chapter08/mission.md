## - 시니어 미션

#### - @DynamicInsert와 @DynamicUpdate가 어떻게 작동되는지 파악하기

- **기본적인 JPA의 동작 방식**

    - 엔티티의 모든 컬럼을 INSERT, UPDATE SQL에 포함시킨다.

    - 즉, 변경되지 않은 필드라도 모두 set 절에 포함시킨다.

- **@DynamicInsert의 동작 방식**

    - 엔티티 저장 시 null인 필드나 기본값이 지정된 필드를 SQL에서 제외한다.

- **@DynamicUpdate의 동작 방식**

    - 엔티티 수정 시 실제 변경된 필드만 UPDATE SQL에 포함한다.

#### - 기존 방식과 @DynamicInsert, @DynamicUpdate를 적용했을 때 장단점 파악하기

- **@DynamicInsert, @DynamicUpdate를 적용했을 때의 장점**

    - insert의 경우는 바인딩 파라미터 수가 감소하고, update의 경우는 변경 필드만 쿼리에 포함하기 때문에 SQL의 쿼리가 감소한다.

    - @DynamicInsert를 활용해 DB의 기본값을 활용할 수 있다.

    - Update 시 변경되지 않은 컬럼에 대한 불필요한 로깅, 트리거가 방지되어 락 충돌이 최소화된다.

- **@DynamicInsert, @DynamicUpdate를 적용했을 때의 단점**

    - 매번 다른 컬럼의 조합으로 이루어진 SQL이 생성되어 SQL 캐시 히트율이 저하된다.

    - Hibernate가 엔티티 상태를 dirty checking으로 비교해서 SQL을 생성하기 때문에 CPU 사용량이 증가한다.

    - 복잡한 매핑에서는 어떤 컬럼이 빠졌는지 파악이 어려워 디버깅이 번거로워질 수 있다.

#### - @DynamicInsert, @DynamicUpdate를 언제 적용하면 좋을지 파악하기

- **@DynamicInsert**

    - 에니티에 DB 기본값으로 정의된 컬럼이 많을 때

    - 일부 필드만 채우고 나머지는 DB가 자동으로 값을 넣도록 할 때

    - 생성 일시, UUID 자동 생성 등의 기본값 컬럼이 많을 때

- **@DynamicUpdate**

    - 엔티티의 필드가 많고, 일부 필드만 자주 수정될 때

    - 낙관적 락 버전 컬럼을 쓸 경우, 변경 없는 컬럼이 업데이트 되면서 충돌 위험이 높아질 때

- **적용하지 않는게 좋은 상황**

    - 컬럼이 많지 않은 엔티티

    - 수정 시 거의 모든 필드를 업데이트 하는 로직

    - PreparedStatement 캐시 히트율이 중요한 대량 처리

        - PreparedStatement 캐시란, Hibernate 등의 ORM이 이미 준비해 둔 SQL문을 다시 생성하지 않고 재사용하는 것을 말한다.

<br>