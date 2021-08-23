## CH07. 예외처리

<img width="673" alt="스크린샷 2021-08-24 오전 12 16 34" src="https://user-images.githubusercontent.com/39195377/130473228-d9e36d1b-b0a6-4ec7-bdd6-5b2b3eed19bf.png">


### 🧚 Uncheck Exception을 사용하자.

- 자바 언어 명세가 요구하는 것은 아니지만, 업계에 널리 펴진 규약으로 Error 클래스를 상속해 하위 클래스를 만드는 일은 자제하자.
- 즉, 사용자가 직접 구현하는 unchecked throwable은 모두 Runtime Exception의 하위 클래스여야 한다.
- Exception, RuntimeException, Error를 상속하지 않는 throwable을 만들수도 있지만 이러한 throwable은 정상적인 사항보다 나을 게 하나도 없으면서 API 사용자를 헷갈리게 할 뿐이다.

### 🧚 Checked Exception이 나쁜 이유

1. 특정 메소드에 cheked Exception을 throw 하고 상위 메소드에서 그 exception을 catch한다면 모든 중간단계 메소드에 exception을 throws 해야 한다.

2. OCP(개방 폐쇄 원칙) 위배

   상위 레벨 메소드에서 하위 레벨 메소드의 디테일 (Exception)에 대해 알아야 하기 때문에 OCP원칙에 위배된다.

3. 필요한 경우 checked Exception을 사용하지만, 대부분의 경우 득보다는 실이 많다.

### 🧚 Exception 잘 쓰기

1. 예외에 메시지를 담아라. 

   🐯 오류가 발생한 원인과 위치를 찾기 쉽도록 예외를 던질때는 전후 상황을 충분히 덧붙인다.

   🐯 실패한 연산 이름과 유형 등 정보를 담아서 예외를 던진다

   🐯 로그는 찍을 뿐 할 수 있는 일이 없다.

  <img width="618" alt="스크린샷 2021-08-24 오전 12 16 58" src="https://user-images.githubusercontent.com/39195377/130473138-51367778-0d01-469e-83b2-7338dc9f442d.png">


   위와 같은 checked Exception의 catch단에서의 예외 처리는 로그를 찍는것 말고는 처리할 수 있는 방법이 없다. 위와 같은 상황에서는 아래 코드처럼 예외를 감싸는 클래스를 만들어서 처리하자.

  <img width="572" alt="스크린샷 2021-08-24 오전 12 17 07" src="https://user-images.githubusercontent.com/39195377/130473156-0e35176f-0079-4697-8dd1-73aeaa55f38b.png">

   - checked Exception들을 하나의 PortDeviceFaulure Exception으로 감싸서 던진다.

### 🧚 실무 예외 처리 패턴

1. getOrElse : 예외 대신 기본 값을 리턴한다.

   - null이 아닌 기본 값
   - 도메인에 맞는 기본 값을 가져온다.

   ```java
   UserLevel userLevel = null;
   try {
   	User user = userRepository.findByUserId(userId);
   	userLevel = user.getUserLevel();
   }catch (UserNotFoundException e) {
   	userLevel = UserLevel.BASIC;
   }
   
   // userLevel을 이용한 처리
   ```

   위 예제 case는 그리 좋은 코드가 아니다.  비즈니스 처리 로직에 try-catch문이 실행되기 때문에 코드의 논리적인 흐름이 끊기게 된다.

   ```java
   public class UserService {
   	private static final UserLevel USER_BASIC_LEVEL = UserLevel.BASIC;
   	
   	public UserLevel getuserLevelOrDefault(Long userId) {
   		try {
   				User user = userRepository.findByUserId(userId);
   				return usr.getuserLevel();
   		}catch (UserNotFoundException e) {
   				return USER_BASIC_LEVEL;
   		}
   }
   ```

   예외 처리를 데이터를 제공하는 쪽에서 처리해 호출부 코드가 심플해진다. 즉, 논리적인 흐름을 끊지 않도 ㅗ드를 읽어나갈 수 있으며, 도메인에 맞는 기본값을 도메인 서비스에서 관리한다.

2. getorElseThrow : null 대신 예외를 던진다.

   null을 리턴하게 되면 일명 null 체크 지옥에 빠진다.  해당 메소드를 호출한 쪽에서 null을 체크하는 코드가 추가되며, 이는 코드의 가독성을 떨어뜨린다.

   - 기본 값이 없다면, null 대신 예외를 던진다.

   ```java
   User user = userRepository.findByUserId(userId);
   if(user == null) {
   		throw new IllegalArgumentException("User is not fount. userId = " + userId)
   }
   return user;
   
   }
   ...
   ```

3. 파라미터의 null을 점검하라.

   위에서 null 체크 지옥이라는 말을 언급한 적이 있는데, 여기서는 예외다. null을 리턴하는 것도 나쁘지만 파라미터로 null을 전달하는것은 더 나쁘다.

   ```java
   ...
   if(p1 == null || p2 == null) {
   		throw InvalidArgumentException( ....);
   }
   ...
   ```

   위 예제처럼 파라미터로 들어오는 값들에 대한 null 체크는 반드시 해주자!
