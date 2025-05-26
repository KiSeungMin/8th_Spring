## - 시니어 미션

#### - Slice와 Page의 구조적 차이와 적용 시점 파악하기

**1. Page와 Slice가 각각 어떻게 출력값이 나오는 지 알아보기**

- **Page를 적용했을 때**

  ``` java
  {
    "isSuccess": true,
    "code": "COMMON200",
    "message": "성공입니다.",
    "result": {
      "reviewList": [
        {
          "ownerNickname": "string",
          "score": 3,
          "body": "고생",
          "createdAt": "2024-09-27"
        },
        {
          "ownerNickname": "string",
          "score": 4,
          "body": "많으셨습니다!",
          "createdAt": "2024-09-27"
        }
      ],
      "listSize": 2,
      "totalPage": 1,          
      "totalElements": 2,      
      "isFirst": true,
      "isLast": true,
      "pageNumber": 0,         
      "pageSize": 10,          
      "numberOfElements": 2    
    }
  }
  ```

  -> totalPage, totalElement, pageNumber, pageSize, numberOfElements 등의 정보를 통해 **전체 페이지 정보를 파악할 수 있다.**

- **Slice를 적용했을 때**

  ``` java
  {
    "isSuccess": true,
    "code": "COMMON200",
    "message": "성공입니다.",
    "result": {
      "reviewList": [
        {
          "ownerNickname": "string",
          "score": 3,
          "body": "고생",
          "createdAt": "2024-09-27"
        },
        {
          "ownerNickname": "string",
          "score": 4,
          "body": "많으셨습니다!",
          "createdAt": "2024-09-27"
        }
      ],
      "listSize": 2,
      "isFirst": true,
      "isLast": true,
      "hasNext": false
    }
  }
  ```

  -> **총 개수/총 페이지 정보는 빠지고, 대신 다음 페이지가 있는지 여부만 제공한다.**

**2. Page, Slice 각각 적용 시 장단점 파악하기**

- **Page를 적용했을 때 장점**

    - totalElements, totalPages, pageNumber, pageSize, numberOfElements 등 **페이징에 필요한 모든 정보를 반환한다.**

    - 클라이언트에서 총 몇 건, 몇 페이지인지 바로 확인할 수 있어서 **페이지 네비게이션 등의 구현이 쉬워진다.**

    - 사용자가 **임의의 페이지로 건너뛰기 하기가 쉬워진다.**

    - **전체 건수를 정확히 표시해야 하는 경우 필수적이다.**

- **Page를 적용했을 때 단점**

    - **count(*)를 위한 별도가 항상 실행되기 때문에** 데이터 규모가 크거나 복잡한 조인/조건이 많으면 응답 시간이 늘어난다.

    - 트래픽이 많거나 실시간성이 중요할 경우 지연이 발생한다.

- **Slice를 적용했을 때 장점**

    - **전체 count 쿼리가 실행되지 않아 조회 성능이 훨씬 빠르다.**

    - 첫 페이지나 반복 호출이 빈번할 때 사용하기 좋다.

    - hasNext() 정보만 제공하기 때문에 **더 불러오기, 무한 스크롤 구현이 쉬워진다.**

- **Slice를 적용했을 때 단점**

    - **전체 건수와 페이지를 알 수 없어 마지막 페이지라는 사실만 알 수 있다.**

    - 총 X건의 정보를 표기할 수 없다.

    - 사용자가 임의 접근하기가 어렵고, 이전/다음 접근만 할 수 있다.

**3. 언제 적용하면 좋을 지 파악하기**

- **Page를 적용하면 좋을 때**

    - 전체 건수, 전체 페이지, 현재 페이지 정보가 필요할 때

    - 임의의 페이지로 자유롭게 이동할 수 있어야 할 때

    - ex) 관리자 대시보드, 검색 결과 페이징

- **Slice를 적용하면 좋을 때**

    - 무한 스크롤, 더 보기 버튼으로 연속 데이터를 로딩할 때

    - 전체 카운트가 필요하지 않고, 즉각저인 응답 속도가 더 중요할 때

    - 대규모 데이터를 대상으로 빈번히 호출될 때

    - ex) 피드, 무한 스크롤, 더보기

## - 실습 기록

#### - 페이징 검증 어노테이션 생성

- 미션에 page 범위를 검증하는 어노테이션을 생성해야 했기 때문에 먼저 검증 어노테이션과 validator를 다음과 같이 추가해주었다.

``` java
@Documented
@Constraint(validatedBy = PageableIndexValidator.class)
@Target({ ElementType.PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidPageableIndex {
    String message() default "page 파라미터는 1 이상이어야 합니다.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

``` java
public class PageableIndexValidator implements ConstraintValidator<ValidPageableIndex, Pageable> {
    @Override
    public boolean isValid(Pageable pageable, ConstraintValidatorContext context) {
        if (pageable == null) {
            return true;
        }
        int pageNumber = pageable.getPageNumber();
        return pageNumber >= 0;
    }
}
```

#### 1. 내가 작성한 리뷰 목록 조회

- 먼저 다음과 같이 querydsl을 사용해 repository 로직을 구현해주었다. Repository에서 Pageable을 구현해서 반환하도록 코드를 작성했다.

``` java
    @Override
    public Page<Review> findByMemberReviews(Long memberId, Pageable pageable) {
        List<Review> content = jpaQueryFactory
                .selectFrom(review)
                .where(review.member.id.eq(memberId))
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .orderBy(review.createdAt.desc())
                .fetch();

        long total = jpaQueryFactory
                .selectFrom(review)
                .where(review.member.id.eq(memberId))
                .fetchCount();

        return new PageImpl<>(content, pageable, total);
    }
```

- 서비스 로직에서는 Repository 조회 결과를 Dto로 변환해 반환하도록 구현했다.

``` java
    @Override
    public Page<ReviewResponseDto.JoinResultDTO> findUserReviews(Long memberId, Pageable pageable) {
        Page<Review> reviews = reviewRepository.findByMemberReviews(memberId, pageable);
        return reviews.map(ReviewConverter::toJoinResultDTO);
    }
```

- 컨트롤러 로직에서는 유저의 리뷰를 조회해야 하기 때문에 엔드포인트로 /memberId를 설정해주었다. 또한, @PageableDefault 어노테이션을 사용해 한 번에 조회되는 리뷰의 개수를 기본 10개로 설정해주었다.


``` java
    @GetMapping("/{memberId}")
    public ApiResponse<Page<ReviewResponseDto.JoinResultDTO>> getUserReviews(
            @ValidPageableIndex
            @PageableDefault(size = 10, sort = "createdAt", direction = Sort.Direction.DESC)
            Pageable pageable,
            @PathVariable("memberId") Long memberId
    ) {
        return ApiResponse.onSuccess(reviewQueryService.findUserReviews(memberId, pageable));
    }
```

#### 2. 특정 가게의 미션 목록 조회

- 먼저 다음과 같이 repository에 페이징을 적용해 가게의 모든 미션 목록을 조회하는 메서드를 만들었다.

``` java
    @Override
    public Page<Mission> findMissionsByStore(
            Long storeId,
            Pageable pageable
    ) {
        List<Mission> content = jpaQueryFactory
                .selectFrom(mission)
                .join(mission.store, store).fetchJoin()
                .where(
                        store.id.eq(storeId)
                )
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .orderBy(mission.id.asc())
                .fetch();

        Long totalCount = jpaQueryFactory
                .select(mission.count())
                .from(mission)
                .join(mission.store, store)
                .where(
                        store.id.eq(storeId)
                )
                .fetchOne();

        long total = (totalCount != null ? totalCount : 0L);

        return new PageImpl<>(content, pageable, total);
    }
```

- service에서는 1번과 마찬가지로 결과값을 dto로 변환해 반환하도록 메서드를 구현했다.

``` java
    @Override
    public Page<MissionResponseDto.JoinResultDTO> findStoreMissions(Long storeId, Pageable pageable) {
        Page<Mission> storeMissions = storeRepository.findMissionsByStore(storeId, pageable);

        return storeMissions.map(MissionConverter::toJoinResultDTO);
    }
```

- 컨트롤러 로직에서는 가게의 모든 미션들을 조회해야 하기 때문에 엔드포인트로 /{storeId}/missions로 설정해주었다. 또한, @PageableDefault 어노테이션을 사용해 한 번에 조회되는 리뷰의 개수를 기본 10개로 설정해주었다. 이전에 생성한 페이징 검증 어노테이션도 적용해주었다.

``` java
    @GetMapping("/{storeId}/missions")
    public ApiResponse<Page<MissionResponseDto.JoinResultDTO>> getStoreMissions(
            @ValidPageableIndex
            @PageableDefault(size = 10, sort = "createdAt", direction = Sort.Direction.DESC)
            Pageable pageable,
            @PathVariable("storeId") Long storeId
    ) {
        return ApiResponse.onSuccess(storeQueryService.findStoreMissions(storeId, pageable));
    }
```

#### 3, 4. 내가 진행중, 진행완료한 미션 목록 조회

- 먼저 다음과 같이 repository에 페이징을 적용해 가게의 모든 미션 목록을 조회하는 메서드를 만들었다. 진행중, 진행 완료한 미션을 검색 조건에 따라 조회해야 하므로, **MissionStatus 값을 저장하고 있는 status 필드를 전달받아 status와 일치하는 값만 조회하도록 where 절을 설정했다.**

```
    @Override
    public Page<MemberMission> findMissionsByMember(
            Long memberId,
            MissionStatus status,
            Pageable pageable
    ) {
        List<MemberMission> content = jpaQueryFactory
                .selectFrom(memberMission)
                .join(memberMission.member).fetchJoin()
                .where(
                        memberMission.id.eq(memberId),
                        memberMission.status.eq(status)
                )
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .orderBy(memberMission.id.asc())
                .fetch();

        Long totalCount = jpaQueryFactory
                .select(memberMission.count())
                .from(memberMission)
                .join(memberMission.member)
                .where(
                        member.id.eq(memberId)
                )
                .fetchOne();

        long total = (totalCount != null ? totalCount : 0L);

        return new PageImpl<>(content, pageable, total);
    }
```

- service에서는 1번과 마찬가지로 결과값을 dto로 변환해 반환하도록 메서드를 구현했다.

``` java
    @Override
    public Page<MemberMissionResponseDto.JoinResultDTO> findMemberMissions(Long memberId, MissionStatus status, Pageable pageable) {
        Page<MemberMission> memberMissions = memberRepository.findMissionsByMember(memberId, status, pageable);

        return memberMissions.map(MemberMissionConverter::toJoinResultDTO);
    }
```

- 컨트롤러 로직에서는 가게의 모든 미션들을 조회해야 하기 때문에 엔드포인트로 /{memberId}/missions로 설정해주었다. 또한, @PageableDefault 어노테이션을 사용해 한 번에 조회되는 리뷰의 개수를 기본 10개로 설정해주었다. 이전에 생성한 페이징 검증 어노테이션도 적용해주었다. 또한, 3번과 4번 미션이 각각 검색 조건을 진행중, 진행 완료로 검색하기 때문에 mission의 상태를 query parameter로 전달할 수 있도록 **status 값을 @RequestParam으로 설정해주었다.**

``` java
    @GetMapping("/{memberId}/missions")
    public ApiResponse<Page<MemberMissionResponseDto.JoinResultDTO>> getMemberMissions(
            @ValidPageableIndex
            @PageableDefault(size = 10, sort = "createdAt", direction = Sort.Direction.DESC)
            Pageable pageable,
            @RequestParam MissionStatus status,
            @PathVariable("memberId") Long memberId
    ) {
        return ApiResponse.onSuccess(memberCommandService.findMemberMissions(memberId, status, pageable));
    }
```