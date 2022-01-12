# springboot-aws-study
『스프링 부트와  AWS로 혼자 구현하는 웹 서비스』 공부내용 정리
<br>
![image](http://image.kyobobook.co.kr/images/book/large/602/l9788965402602.jpg)

# 1. JPA

### SQL을 직접 사용할 때의 문제점
- 관계형 데이터베이스는 SQL만 인식할 수 있기 때문에 각 테이블마다 기본적인 CRUD SQL을 매번 생성해야 한다.
  - 코드가 SQL 중심이 되고, SQL 의존적인 개발을 피할 수 없다.
- 관계형 데이터베이스는 어떻게 데이터를 저장할지에 초점이 맞춰진 기술이지만, 객체지향 프로그래밍 언어는 메시지를 기반으로 기능과 속성을 한 곳에서 관리하는 기술
  - 듀 언어의 패러다임이 서로 다르기 때문에, 객체를 데이터베이스에 저장할 때 패러다임의 불일치 발생

<br>

### JPA
- 자바 표준 ORM로, 서로 다른 객체지향 프로그래밍 언어와 관계형 데이터베이스 중간에서 패러다임 일치를 시켜주기 위한 기술
- 개발자는 객체지향적으로 프로그래밍을 하고, JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행
  - 더는 SQL에 종속적인 개발을 하지 않아도 되므로, 생산성 향상과 유지보수가 쉬워진다.
  
> MyBatis는 SQL Mapper로, ORM은 객체를 매핑하지만 SQL Mapper는 쿼리를 매핑한다.

<br>

### Spring Data JPA
- JPA는 인터페이스로서 자파 표준 명세서, 따라서 JPA를 사용하려면 구현체가 필요하다. (Hibernate, Eclipse, Link 등)
- 스프링에서 JPA를 사용할 때는 구현체들을 좀 더 쉽게 사용하고자 추상화시킨 Spring Data JPA라는 모듈을 사용한다.<br>
`JPA <- Hibernate <- Spring Data JPA`

#### 장점
1. 구현체 교체의 용이성: Spring Data JPA 내부에서 구현체 매핑을 지원해주기 때문에, Hibernate 외에 다른 구현체로 쉽게 교체할 수 있다.

2. 저장소 교체의 용이성: Spring Data의 하위 프로젝트들은 기본적인 CRUD 인터페이스가 같기 때문에, 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체할 수 있다.
(데이터베이스 교체를 위해 개발자는 의존성만 교체하면 된다.)
 
> Spring Data의 하위 프로젝트: Spring Data JPA, Spring Data Redis, Spring Data MongoDB 등

<br>

### 영속성 컨텍스트
- 엔티티를 영구 저장하는 환경으로, JPA의 핵심 내용은 엔티티가 영속성 컨텍스트에 포함되어 있냐 아니냐로 갈린다.
- JPA의 엔티티 매니저가 활성화된 상태로 트랜잭션 안에서 데이터베이스로부터 데이터를 가져오면 이 데이터는 영속 상태가 된다.
- 영속 상태에서 해당 데이터의 값을 변경하면 트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영한다. (더)
  - 즉, Entity의 값만 변경하면 별도로 Update 쿼리를 날릴 필요가 없다. 

<br>

### 정리: JPA를 사용하면!
- CRUD 쿼리를 직접 작성할 필요가 없다.
- 부모 자식 관계 표현, 1:N 관계 표현, 상태와 행위를 한 곳에서 관리하는 등 객체지향 프로그래밍을 쉽게 할 수 있다.

<br>

# 2. 스프링 웹 계층
![image](https://user-images.githubusercontent.com/60869749/149092708-15e106af-484d-45e7-b517-1d517bbf5401.png)

#### Web Layer
- 흔히 사용하는 컨트롤러와 JSP 등의 뷰 템플릿 영역
- 이외에도 필터, 인터셉터, 컨트롤러 어드바이스 등 외부 요청과 응답에 대한 전반적인 영억

#### Service Layer
- 서비스 영역으로, 컨트롤러와 DAO의 중간 영역에서 사용
- @Transactional이 사용되어야 하는 영역

#### Repository Layer
- 데이터베이스와 같이 데이터 저장소에 접근하는 영역
- DAO

#### Dtos
- 계층 간에 데이터 교환을 위한 객체인 DTO를 위한 영역
- 뷰 템플릿 엔진에서 사용하거나, Repository Layer에서 결과로 넘겨준 객체 등

#### Domain Model
- 도메인이라 불리는 개발 대상을 모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화 시킨 것
- 엔티티 클래스, VO

<br>

#### 도메인 주도 설계
- 도메인 모델에서 비즈니스 로직을 처리하는 방식
- 서비스 메소드는 트랜잭션과 도메인 간의 순서만 보장한다.

> 트랜잭션 스크립트: 비즈니스 로직을 서비스 영역에서 처리하는 방식으로, 모든 로직이 서비스 클래스 내부에서 처리

<br>

# 3. 도메인
- 도메인이란 게시글, 댓글, 회숸, 결제 등 소프트웨어에 대한 요구사항 혹은 문제 영역
- MyBatis와 같은 쿼리 매퍼에서 xml에 쿼리를 담고 클래스는 오로지 쿼리의 결과만 담던 일들이 모두 도메인 클래스에서 해결

### Entity 클래스
- Entity 클래스는 실제 DB의 테이블과 매칭된다.
- JPA를 사용하면 DB 데이터에 작업할 경우 실제 쿼리를 날리기보다는, Entity 클래스의 수정을 통해 작업
- Entity의 PK는 Long 타입의 Auto_increment를 사용하는 것을 추천
- Setter 메소드를 사용하면 해당 클래스의 인스턴스 값들이 언제 어디서 변해야 하는지 코드상으로 명확하게 구분할 수 없으므로, Setter를 만들지 않는다.
  - 대신 해당 필드의 값 변경이 필요하면 명확히 그 목적과 의도를 나타낼 수 있는 메소드 추가


### 어떻게 값을 채우지??
- 기본적인 구조는 생성자를 통해 최종값을 채운 후 DB에 삽입한다.
- 값 변경이 필요한 경우 해당 이벤트에 맞는 public 메소드를 호출하여 변경한다.

> @Builder: 생성자의 경우 지금 채워야 할 필드가 무엇인지 명확히 지정할 수 없지만, 빌더를 사용하면 어느 필드에 어떤 값을 채워야 하는지 명확하게 인지할 수 있다.

<br>

### Repository
- MyBatis에서 DAO라고 불리는 DB Layer 접근자
- 인터페이스 생성 후, JpaRepository<Entity 클래스, PK 타입>를 상속하면 기본적인 CRUD 메소드가 자동으로 생성된다.
- Entity 클래스와 기본 Entity Repository는 함께 위치해야 한다.
  - 둘은 아주 밀접한 관계이고, Entity 클래스는 기본 Reposiotry 없이는 제대로 역할을 할 수 없기 때문
  - 도메인 별로 프로젝트를 분리해야 하는 경우, Entity 클래스와 기본 Repository는 함께 움직여야 하므로, 도메인 패키지에서 함게 관리

<br>

# 4. API
### Service
- 트랜잭션, 도메인 간 순서 보장의 역할
- Bean을 주입받을 때 생성자로 주입받는 방식 권장
  - @RequiredArgsConstructor: final이 선언된 모든 필드를 인자값으로 하는 생성자 생성
  - 해당 클래스의 의존성 관계가 변경될 때마다 생성자 코드를 수정하는 번거로움 해결

<br>

### DTO
- Entity 클래스는 데이터베이스와 맞닿은 핵심 클래스로, Request/Response 클래스로 사용하면 안 된다.
- Entity 클래스를 기준으로 테이블이 생성되고 스키마가 변경되는데, 화면 변경을 위해 테이블과 연결된 Entity 클래스를 변경하는 것은 좋지 않다.
- 따라서, View를 위한 Request/Response 용 DTO를 따로 만든다.

> View Layer와 DB Layer의 역할 분리를 철저하게 하는게 좋다.<br>
> 실제로 컨트롤러에서 결과값으로 여러 테이블을 조인해서 줘야하는 경우가 빈번하므로, Entity 클래스만으로 표현하기 어려운 경우가 많다.<br>
> 꼭 Entity 클래스와 컨트롤러에서 쓸 DTO는 분리해서 사용하자!

<br>

# 5. 테스트
- @ WebMvcTest의 JPA의 기능이 작동하지 않고, 컨트롤러와 컨트롤러 어드바이스 등 외부 연동과 관련된 부분만 활성화
- JPA 기능까지 한 번에 테스트할 때는 @SpringBootTest오 TestRestTemplate를 사용한다.

<br>

# 6. JPA Auditing
- 보통 엔티티에는 해당 데이터의 생성시간과 수정시간을 포함한다.
- BaseTimeEntity 클래스는 모든 Entity의 상위 클래스가 되어 Entity들의 createdDate, modifiedDate를 자동으로 관리하는 역할

### 어노테이션
- @MappedSuperclass: JPA Entity 클래스들이 BaseTimeEntity를 상속할 경우 필드들도 칼럼으로 인식하도록 한다.
- @EntityListeners(AuditingEntityListener.class): BaseTimeEntity 클래스에 Auditing 기능을 포함시킨다.
- @createdDate, @ModifiedDate: Entity가 생성되어 저장될 때, 조회한 Entity의 값을 변경할 때 시간 자동 저장
- @EnableJpaAuditing: JPA Auditing 어노테이션들을 모두 
