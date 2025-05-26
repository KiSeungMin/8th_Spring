## - 키워드 정리

#### - Spring Data JPA의 Paging

- **주요 인터페이스와 클래스**

    - **Pageable**

        - 페이지 번호와 페이지 크기, 정렬 정보를 담는 인터페이스

        - PageRequest를 구현체로 사용한다.

        - 주요 메서드, 필드

            - content: 현재 페이지에 포함된 엔티티 리스트

            - pageable: page, size, sort 정보를 포함한 객체

            - total: 전체 요소 수

            - getContent(): 조회된 엔티티 리스트 반환

            - getTotalElements(): total값 반환

            - getTotalPages(): totalElements/pageSize 반환

            - hasNext() ,hasPrevious(): 현재 페이지와 전체 페이지 수를 비교해 판단

    - **Page**

        - 조회된 데이터 리스트와, 전체 페이지 수, 전체 요소 수, 현재 페이지 등의 데이터를 제공한다.

    - **slice**

        - Page에서 몇몇 요소를 제거했다. 다음 페이지가 있는지 여부만 제공한다.

- **사용 방법**

    - **api 요청 예시**

        - 다음과 같이 status가 ACTIVE, username이 kim인 유저를 페이징으로 조회하는 요청을 보낸다고 할 때

      ``` java
      GET /api/users?status=ACTIVE&usernameLike=kim&page=1&size=15&
      ```

    - **요청 dto**

        - 검색 조건 dto는 다음과 같이 정의할 수 있다.

      ``` java
      @Getter @Setter
       public class UserSearchDto {
         private String status;  
         private String usernameLike;
       }
  
      ```

    - **컨트롤러 로직**

        - @PageableDefault : Pageable 객체의 기본값을 설정

      ``` java
       @GetMapping
      public Page<UserDto> searchUsers(
          UserSearchDto condition,                      
          @PageableDefault(size = 20, sort = "createdAt", direction = DESC)
          Pageable pageable
      ) {
          Page<User> page = userService.searchUsers(condition, pageable);
          return page.map(UserDto::fromEntity);
      }
      ```

    - **서비스 로직**

      ```
      public Page<User> searchUsers(UserSearchDto cond, Pageable pageable) {
          return userRepository.searchByCondition(cond, pageable);
      }
      ```

    - **Repository(QueryDSL)**

  ``` java
   @Override
    public Page<User> searchByCondition(UserSearchDto cond, Pageable pageable) {
        QUser u = QUser.user;

        // 1) 컨텐츠 조회
        List<User> content = queryFactory
            .selectFrom(u)
            .where(
                cond.getStatus() != null ? u.status.eq(cond.getStatus()) : null,
                cond.getUsernameLike() != null ? u.username.contains(cond.getUsernameLike()) : null,
            )
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .orderBy(QuerydslUtils.getOrderSpecifier(u, pageable.getSort()))
            .fetch();

        // 2) 전체 카운트 조회
        long total = queryFactory
            .select(u.id)
            .from(u)
            .where(
                cond.getStatus() != null ? u.status.eq(cond.getStatus()) : null,
                cond.getUsernameLike() != null ? u.username.contains(cond.getUsernameLike()) : null,
            )
            .fetchCount();

        return new PageImpl<>(content, pageable, total);
  }
  ```

    - **응답 JSON 형태**

  ``` java
  {
    "content":[
      { "id":12, "username":"kim1", "email":"...", "status":"ACTIVE"},
      …
    ],
    "pageable":{ … },
    "totalElements":123,
    "totalPages":9,
    "last":false,
    "number":1,
    "size":15,
    "sort":{…},
    "first":false,
    "numberOfElements":15
  }
  ```
