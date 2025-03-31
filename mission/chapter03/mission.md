## - 미션

#### - 홈 화면 엔드포인트 설계

<img src = "https://velog.velcdn.com/images/sm011212/post/02ea3fdb-1197-4d76-9342-89448e3f4b97/image.png" width = "500">

- 조회 요청이기에 **GET** 메서드를 사용했다.

- JWT 토큰을 사용한다고 가정해 Request Header에 access token 정보를 작성했다.

- userId는 jwt 토큰에 담겨있을 것 같아 엔드포인트에 userId를 명시하지 않았고, 메인 화면의 의미를 담아 /api/main으로 설정해주었다.

- 메인 화면에서 지역 정보를 필요로 하기에 query string으로 region 값을 넣어주었고, 커서 기반 페이징 적용을 위해 cursor 값과 limit 값을 추가로 요청 받도록 설정했다.

- GET 요청이기 때문에 Request Body에는 값을 입력하지 않았다.

#### - 마이 페이지 리뷰 작성

<img src = "https://velog.velcdn.com/images/sm011212/post/3b1bf940-cbd1-4344-aa4a-814c1727e802/image.png" width = "500">

- 등록 요청이기에 **POST** 메서드를 사용했다.

- JWT 토큰을 사용한다고 가정해 Request Header에 access token 정보를 작성했다.

- 특정 store에 review를 작성하는 것이기 때문에 엔드포인트에 /store/{storeId}를 작성해 storeId를 전달하도록 했고, review를 작성하는 것이기 때문에 /review를 추가로 적어주었다.

- rating 값은 소수값이라고 가정했고, image는 여러 개의 값을 받을 수 있을 것이라 생각해 url string이 담긴 리스트로 설계했다.

#### - 미션 목록 조회

<img src = "https://velog.velcdn.com/images/sm011212/post/7f768827-cd39-4e56-80ec-f99b8c14cdbd/image.png" width = "500">

- 조회 요청이기에 **GET** 메서드를 사용했다.

- JWT 토큰을 사용한다고 가정해 Request Header에 access token 정보를 작성했다.

- userId는 jwt 토큰에 담겨있을 것 같아 엔드포인트에 userId를 명시하지 않았고, 특정 유저의 미션이므로 user-missions 라는 엔드포인트를 구분했다. (그냥 mission으로 하면 공통 미션인지 아닌지 헷갈리니까)

- 커서 기반 페이징 적용을 위해 query string으로 cursor 값과 limit 값을 추가로 요청 받도록 설정했다.

- GET 요청이기 때문에 Request Body에는 값을 입력하지 않았다.

#### - 미션 성공 누르기

<img src = "https://velog.velcdn.com/images/sm011212/post/7817b09c-e782-418d-bdea-46e98549ecc6/image.png" width = "500">

- 미션 정보의 상태를 변화시키기 때문에 **PATCH** 메서드를 사용했다.

- JWT 토큰을 사용한다고 가정해 Request Header에 access token 정보를 작성했다.

- userId는 jwt 토큰에 담겨있을 것 같아 엔드포인트에 userId를 명시하지 않았고, 특정 유저의 미션이라는 것을 명시하기 위해 user-missions 엔드포인트로 설정했다. 해당 미션의 정보를 알아야 하기 때문에 {missionId}를 path variable로 받도록 했다.

- 미션의 상태를 변화시킬 수 있도록 request body에 변화시킬 상태값을 보내도록 설계했다. (성공 전용 엔드포인트를 만드는 것보다 상태를 변화시키는 엔드포인트를 만드는게 낫다고 생각했다.)

#### - 회원 가입 하기

<img src = "https://velog.velcdn.com/images/sm011212/post/0bfc757f-dea8-4261-9192-5887ed9a6bc5/image.png" width = "500">

<img src = "https://velog.velcdn.com/images/sm011212/post/d2bbd339-fa6e-45a3-9ab8-edd6fa41de10/image.png" width = "500">

- 새롭게 유저를 등록하므로 **POST** 메서드를 사용했다.

- 아직 유저가 등록되지 않았으므로 access token 정보는 작성하지 않았다.

- 유저 등록에 필요한 정보들을 Request Body로 작성했다.

- 주소 정보, 알림 여부, 권한 정보는 별도의 dto로 받을 수도 있다고 생각해 map 객체로 받도록 설계했다.