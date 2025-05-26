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