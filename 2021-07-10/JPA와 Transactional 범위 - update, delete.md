JPA 를 이용해 서비스를 개발 중에 마주친 문제점과 해결방안에 대해서 공유합니다. 실제 개발 중에 겪었던 문제 상황을 간략한 도메인을 만들어서 재연해 보도록 하겠습니다. 사용한 기술 스택은 아래와 같습니다.

- Java 11
- Spring Boot 2.3.11.RELEASE
  - Spring Boot Starter Data JPA
- junit 5

## 문제 상황

발견한 문제 상황은 총 두 가지 입니다. 문제의 원인은 동일하기 때문에 하나의 주제로 정리를 하도록 하겠습니다.

첫 번째 문제는 `JpaRepository` 를 이용해 엔티티를 삭제할 때 의도했던 범위에 예외를 반환하지 않았던 문제였습니다. 

두 번째 문제는 서비스 계층에서 엔티티를 업데이트 했을 때 업데이트 한 정보를 제대로 반영하지 못한 문제입니다. 서비스 계층에서 엔티티를 업데이트 한 후  엔티티의 업데이트 정보를 외부 시스템에 이벤트에 담아 발행하는 구조가 있었습니다. 이때 외부 시스템은 엔티티가 업데이트 되었다는 이벤트를 수신하면 데이터베이스에 해당 엔티티를 조회해 정의한 동작을 수행하게 되는 구조입니다. 외부 시스템이 데이터베이스에 업데이트된 엔티티를 조회했을 때 업데이트된 데이터가 아닌 업데이트 이전의 엔티티 정보가 조회되는 문제가 있었습니다.

하나씩 간단한 예제 코드를 통해서 설명하겠습니다. 먼저 문제가 되었던 코드를 소개하고 수정한 후의 코드를 소개해 두 코드를 비교하여 어디에서 문제가 발생하였고 무엇이 문제였는지 확인해 보도록 하겠습니다.

### 공통 도메인

예시에서 사용할 아주 단순하게 재구성한 도메인 입니다. 서비스 계층에서 단순한 기능만 사용할 것이기 때문에 아무 비즈니스 로직을 포함하지 않도록 하겠습니다.

User 엔티티와 Post 엔티티가 있을 때 User 1 : N Post 의 구조를 갖습니다.

``` java
// entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Post> posts = new ArrayList<>();

    public User(String name) {
        this.name = name;
    }

}

@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    private User user;

    public Post(String title, User user) {
        this.title = title;
        this.user = user;
    }

}

// repository
public interface UserRepository extends JpaRepository<User, Long> {
}

public interface PostRepository extends JpaRepository<Post, Long> {
}
```

## 삭제 문제

먼저 삭제를 하는 로직은 단순합니다.

- 컨트롤러에서 요청으로 User 의 아이디 값을 넘겨줍니다.
- 해당 아이디에 유효한 엔티티가 있는지 확인합니다. 없다면 예외를 반환합니다.
- 엔티티 삭제를 시도합니다.

위 로직을 서비스 계층에 구현하면 아래와 같습니다. 그리고 아래 코드는 처음 문제가 발생했던 코드입니다.

``` java
@AllArgsConstructor
@Service
public class UserService {

    private final UserRepository userRepository;

    @Transactional
    public void deleteUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(IllegalArgumentException::new);
        try {
            userRepository.delete(user);
        } catch (DataIntegrityViolationException exception) {
            throw new IllegalArgumentException();
        }
    }

}
```

User 삭제를 시도할 때 try catch 구문을 이용한 이유는 만약 User 에 연관된 Post 가 있을 때 삭제가 되지 않기 때문에 해당 예외를 처리하기 위함입니다.

다음은 서비스 계층을 테스트 해보겠습니다. 테스트 코드를 작성할 때는 catch 구문에서 예상한 `DataIntegrityViolationException` 이 반환되어 `IllegalArgumentException` 이 throw 될 것이라 생각하였습니다. 따라서 작성한 테스트 코드는 아래와 같습니다.

``` java
@DisplayName("외래키 제약사항에 포함된 user 를 삭제 시 예외를 반환한다")
@Test
public void deleteUserWithForeignKeyConstraints() {
  // given
  User savedUser = userRepository.save(new User("user1"));
  Post savedPost = postRepository.save(new Post("post1", savedUser));
  Long userId = savedUser.getId();

  // then
  assertThatThrownBy(() -> userService.deleteUserById(userId))
    .isInstanceOf(IllegalArgumentException.class);
}
```

하지만, 위 테스트 코드는 실패했고 반환된 예외는 `DataIntegrityViolationException` 이었습니다. 

![image](https://user-images.githubusercontent.com/39546083/125152869-b991ab00-e18a-11eb-9e15-af45fe93f131.png)

해당 예외는 스프링에서 제공하는 예외인데 데이터베이스의 다양한 제약조건을 어겼을 때 반환되는 예외입니다. 하지만 서비스 계층에서 해당 예외가 발생했을 때 try catch 로 처리한 후 IllegalArgumentException 을 반환하도록 했는데 왜 해당 예외가 그대로 반환이 되었을까요?

### 원인은 @Transactional 과 영속성 컨텍스트의 관계

문제가 되었던 코드를 보면 서비스 계층의 deleteUserById 메소드에 `@Transactional` 어노테이션이 붙어있습니다. 따라서 해당 메소드가 끝날 때 까지 영속성 컨텍스트가 유지되고 있습니다. 레포지토리 계층은 JpaRepository 의 구현체를 이용하고 있고 해당 구현체의 `delete~` 메소드들에는 메소드 레벨에 @Transactional 어노테이션이 붙어있습니다. 제가 기본적으로 사용한 구현체는 `SimpleJpaReposiotry` 이고 이 클래스의 내부의 delete(T entity) 메소드를 확인해보면 아래와 같습니다.

![image](https://user-images.githubusercontent.com/39546083/125152999-c95dbf00-e18b-11eb-9573-19ef009b5a72.png)

@Transactional 이 메소드에 설정되어있고 Propagation(트랜잭션의 전파 수준) 이 기본값으로 설정되어 있습니다. 기본값은 `Propagation.REQUIRED` 이고 이 설정 값은 상위 트랜잭션이 실행 중이라면 해당 트랜잭션 내부에서 메소드를 실행하는 것입니다. 그렇기 때문에 delete 쿼리가 레포지토리 계층에서 시작되어 종료 되는 것이 아닌 서비스 계층의 메소드가 종료될 때 delete 쿼리가 실행되는 것입니다. 따라서 서비스 계층 내부의 try catch 에서는 예외를 반환하는 것이 아닌 서비스 계층의 메소드를 호출한 쪽에서 예외가 반환되게 되겠죠!

위와 같은 문제가 발생한 이유는 JPA 에서 삭제 쿼리는 먼저 **영속성 컨텍스트의 쓰기 지연 저장소에 저장**되었다가 실행되기 때문입니다. 쓰기 지연 저장소의 쿼리가 실행되는 시점은 트랜잭션이 커밋되는 시점입니다. 지금 코드는 레포지토리 계층에서 삭제 메소드가 실행되었을 때가 아닌 서비스 계층의 삭제 메소드가 실행되었을 때 트랜잭션이 커밋됩니다. 그래서 서비스 계층의 메소드가 종료되었을 때 `DataIntegrityViolationException` 이 던져지는 것입니다.

### 코드 수정

문제점을 알았으니 수정하기 위해서는 트랜잭션의 커밋 시점을 레포지토리 계층의 메소드가 실행 시점으로 변경해주면 됩니다. 아주 간단하게 서비스 계층의 메소드에 @Transactional 어노테이션을 제거함으로써 처음 의도대로 서비스 계층의 메소드 내부에서 예외를 try catch 로 처리할 수 있습니다.

``` java
@AllArgsConstructor
@Service
public class UserService {

    private final UserRepository userRepository;

    // @Transactional -> 트랜잭션을 없앱니다.
    public void deleteUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(IllegalArgumentException::new);
        try {
            userRepository.delete(user);
        } catch (DataIntegrityViolationException exception) {
            throw new IllegalArgumentException();
        }
    }

}
```

위와 같이 수정한 후 테스트 코드를 실행하면 테스트가 통과하게 됩니다.

## 업데이트 문제

외부로 이벤트를 발행했을 때 엔티티가 업데이트 되지 않았던 문제 역시 동일한 경우였습니다. 

(내용 추가 필요함)

## 결론

위의 두 문제에서 알아본 내용을 정리하겠습니다.

- JPA 를 이용할 때 작성한 쿼리가 실행되는 시점을 제대로 파악하고 있어야 합니다.
  - 엔티티를 저장할 때 작성한 쿼리가 실행되는 시점(@ID 의 생성 전략이 `GenerationType.IDENTITY` 인 경우에 해당)과 엔티티를 수정 / 삭제 하는 쿼리가 실행되는 시점의 차이를 인지하고 있어야 합니다.
- @Transactional 이 관리하는 트랜잭션의 범위를 인지해야 합니다.
  - 트랜잭션의 시작과 커밋 시점에 따라 JPA 메소드의 동작을 예상할 수 있어야 합니다.

또 다른 주의점으로는 테스트 코드를 작성할 때 입니다. 만약 User 엔티티를 삭제하는 문제점을 개선한 코드를 테스트할 때 테스트 코드에 @Transactional 어노테이션을 붙이면 어떻게 동작할까요? 이 경우에 기존 문제점과 마찬가지로 테스트 코드 메소드(상위 메소드)에서 트랜잭션이 시작되기 때문에 의도한 대로 예외를 처리할 수 없습니다.

``` java
@Transactional
@DisplayName("외래키 제약사항에 포함된 user 를 삭제 시 예외를 반환한다")
@Test
public void deleteUserWithForeignKeyConstraints() {
  // given
  User savedUser = userRepository.save(new User("user1"));
  Post savedPost = postRepository.save(new Post("post1", savedUser));
  Long userId = savedUser.getId();

  // then
  assertThatThrownBy(() -> userService.deleteUserById(userId))
    .isInstanceOf(IllegalArgumentException.class);
}
```

따라서 저는 서비스 계층을 테스트할 때 테스트 코드에는 @Transactional 어노테이션을 붙이는 것을 피하고 있습니다. 위와 같이 의도와 다른 동작을 테스트 할 수 있기 때문입니다.