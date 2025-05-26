## - 실습 기록

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
            @PageableDefault(size = 10, sort = "createdAt", direction = Sort.Direction.DESC)
            Pageable pageable,
            @PathVariable("memberId") Long memberId
    ) {
        return ApiResponse.onSuccess(reviewQueryService.findUserReviews(memberId, pageable));
    }
```