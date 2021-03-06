스프링 시큐리티와 OAuth2.0으로 로그인
=======================
# 1. 스프링 시큐리티와 스프링 시큐리티 Oauth2 클라이언트
왜 많은 서비스에서 소셜 로그인을 사용할까? 이동욱 저자님의 생각으로는 **배보다 배꼽이 커지는 경우**라 하셨다.      
   
직접 구현한다면 다음을 전부 구현해야한다.        
* 로그인 시 보안 
* 비밀번호 찾기
* 비밀번호 변경
* 회원정보 변경
* 회원가입시 이메일 혹은 전화번호 인증
    
그렇기에 OAuth2 로그인 구현을하면 앞선 목록을 소셜 기업에 맡기고 서비스에만 집중할 수 있다.       
  
## 1.1. 스프링부트 1.5 vs 스프링 부트 2.0  
스프링 부트 1.5에서의 OAuth2 연동 방법이 2.0에서는 크게 변경되었다.  
하지만 ```spring-security-oauth2-autoconfigure``` 라이브러리 덕분에 **설정 방법의 큰 차이를 없게 할 수 있다**   
```
spring-security-oauth2-autoconfigure
```
해당 라이브러리를 사용할 경우 스프링 부트2에서도 기존 설정을 그대로 사용할 수 있다.   
아무래도 새로운 방식보다는 기존에 안전하게 작동하는 방법이 확실하므로 많은 개발자가 해당 방식을 이용했다.  
    
**하지만**    
우리는 스프링 부트2 방식인 ```Spring Security Oauth2 Client``` 라이브러리를 사용할 것이고    
이유는 아래와 같다.  

* 스프링 팀에서 기존 1.5에서 사용되던 spring-security-oauth 프로젝트는 유지 상태로 경정했으며   
더는 신규 기능은 추가하지 않고 버그 수정 정도의 기능만 추가될 예정이다.   
즉, 신규 기능은 oauth2 라이브러리에서만 지원하겠다고 선언한 것이다.  
* 스프링 부트용 라이브러리(starter)가 출시 되었다.  
* 기존에 사용되던 방식은 확장 포인트가 적절하게 오픈되어 있지 않아 직접 상속하거나 오버라이딩 해야 하고 
신규 라이브러리의 경우 확장 포인트를 고려해서 설계된 상태이다.    
   
그렇기에 이제 새롭게 배우는 학생 입장에서는 스프링 부트2 방식으로 배우는 것이 좀 더 나을 것이다.   
      
**추가 이야깃거리**   
스프링 부트2 방식의 자료를 찾고 싶은 경우 인터넷 자료들 사이에서 다음 2가지만 확인하면 된다.  
1. spring-security-oauth2-autoconfigure 라이브러리를 사용했는지  
2. application.properties 혹은 application.yml 정보가 다음 사진과 같이 차이가 있는지  
   
[사진]   
   
스프링 부트 1.5 방식에서는 url 주소를 모두 명시해야 하지만,     
스프링 부트 2.0 방식에서는 client 인증 정보만 입력하면 된다.      
     
1.5 버전에서 직접 입력했던 값들은 2.0 버전으로 오면서 모두 **enum**으로 대체되었다.     
**CommonOAuth2Provider**라는 enum이 새롭게 추가되어 구글, 깃허브, 페이스북, 옥타의 기본 설정값은 모두 여기서 제공한다.    
이외에 다른 소셜 로그인을 추가한다면 직접 다 추가해 주어야 한다.  

## 1.2. 구글 서비스 등록 

## 1.3. application-oauth 등록  
```application.properties```가 있는 src/main/resources/디렉토리에       
```application-oauth.properties``` 파일을 생성한다.           
그리고 해당 파일에 클라이언트ID와 클라이언트 보안 비밀 코드를 다음과 같이 등록한다.       
    
**application-oauth.properties**    
```
spring.security.oauth2.client.registration.google.client-id=692886957287-663ep6r6ds8ee0oukr9f5mrqof57k6bj.apps.googleusercontent.com
spring.security.oauth2.client.registration.google.client-secret=pVQcqYjwp_7fYyYdOJfkd5rk
spring.security.oauth2.client.registration.google.scope=profile,email
```    
**관점 포인트**   
```
scope=profile,email
```
* 많은 예제에서는 이 scope를 별도로 등록하지 않고 있다.  
* 기본값이 openid, profile, email이기 때문이다.
* 강제로 profile, email을 등록한 이유는 openid가 scope에 있으면 Open Id Provider로 인식하기 때문이다.  
* 이렇게 되면 Open Id Provider 인 서비스(구글)와 그렇지 않은 서비스(네이버/카카오 등)로 나눠서 각각 Oauth2Service를 만들어야 한다.
* 그렇기에 하나의 OauthService로 사용하기 위해 일부러 scope 에서 openid를 빼고 등록해주자.  
  
스프링 부트에서는 properties의 이름을 application-xxx.properties로 생성하면   
xxx라는 이름의 profile이 생성되어 이를 통해 관리를 할 수 있다.  
즉, profile=xxx 라는 식으로 호출하면 해당 properties의 설정들을 가져올 수 있다.   
호출하는 방식은 여러 방식이 있지만 스프링 부트의 기본 설정 파일인 application.properties에서 
appication-oauth.properties를 포함하도록 구성해보자  

**appication.properties**
```
spring.profiles.include=oauth
```

## 1.4. .gitignore 등록  
우리의 프로젝트는 깃허브와 연동하여 사용하다 보니 application-oauth.properties 파일이 깃허브에 올라갈 수 있다.   
보안을 위해 해당 파일이 올라가는 것을 금지해주자    
```
application-oauth.properties
```
추가한 뒤 커밋했을 때 커밋 파일 목록에 **application-oauth.properties**가 나오지 않으면 성공이다.   
만약 .gitignore에 추가했음에도 여전히 커밋 목록에 노출된다면 이는 Git의 캐시문제이다.  

***
# 2. 구글 로그인 연동하기 
## 2.1. User Entity 설정하기

구글의 로그인 인증정보를 발급 받았으니 프로젝트 구현을 진행해보자    
우선 사용자 정보를 담당할 도메인인 ```User```클래스를 생성해주자.    

**User**
```java
package com.jojoldu.book.springboot.domain.user;

import com.jojoldu.book.springboot.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column
    private String picture;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(String name, String email, String picture, Role role){
        this.name = name;
        this.email = email;
        this.picture = picture;
        this.role = role;
    }

    public User update(String name, String picture){
        this.name = name;
        this.picture = picture;

        return this;
    }

    public String getRoleKey(){
        return this.role.getKey();
    }
}

```

**소스코드 해석**
```java
@Enumerated(EnumType.STRING)

* JPA로 데이터베이스로 저장할 때 Enum 값을 어떤 형태로 저장할지를 결정합니다.    
* 기본적으로 int로 된 숫자가 저장됩니다.   
* 숫자로 저장되면 데이터베이스로 확인할 때 그 값이 무슨 코드를 의미하는지 알 수가 없습니다.   
* 그래서 문자열 (EnumType.STRING)로 저장될 수 있도록 선언합니다.  
```


**Role**
```java
package com.jojoldu.book.springboot.domain.user;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum Role {
    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;
}

```
**스프링 시큐리티에서는 권한 코드에 항상 ROLE_이 앞에 있어야 합니다.**      
그래서 코드별 키 값을 ```ROLE_GEUST```, ```ROLE_USER```등으로 지정합니다.     
   

**UserRepository**
```java
package com.jojoldu.book.springboot.domain.user;

import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

```
**소스코드 해석**
```java
findByEmail(String email);   

* 소셜 로그인으로 반환되는 값중 email을 통해 이미 생성된 사용자인지 처음 가입하는 사용자인지 판단하기 위한 메소드
* PK 를 사용한 것이 아니라 Unique 를 사용한 것을 알 수 있다.   
```

## 2.2. 스프링 시큐리티 설정   
먼저 ```build.gradle```에 스프링 시큐리티 관련 의존성 하나를 추가해주자      
    
**build.gradle**    
```gradle  
complie('org.springframework.boot::spring-boot-starter-oauth2-client')
```    

**소스코드 해석**
```gradle
spring-boot-starter-oauth2-client

* 소셜 로그인 등 클라이언트 입장에서 소셜 기능 구현시 필요한 의존성입니다.  
* spring-boot-starter-oauth2-client 와 spring-boot-starter-oauth2-jose를 기본으로 관리해줍니다.
```

``` build.gradle```설정이 끝났으면 OAuth 라이브러리를 이용한 소셜 로그인 코드를 작성해보자      
```config.auth``` 패키지를 생성합니다.         
앞으로 **시큐리티 관련 클래스는 모두 이곳에 담는다**고 보면 될 것 같습니다.     
    
그리고 SecurityConfig 클래스를 생성해줍시다.   

**SecurityConfig**
```java
package com.jojoldu.book.springboot.config.auth;

import com.jojoldu.book.springboot.domain.user.Role;
import lombok.RequiredArgsConstructor;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception{
        http
                .csrf().disable().headers().frameOptions().disable().and()
                .authorizeRequests()
                .antMatchers("/","/css/**","/images/**","/js/**","/h2-console/**", "/profile").permitAll()
                .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                .anyRequest().authenticated()
                .and()
                .logout()
                .logoutSuccessUrl("/")
                .and()
                .oauth2Login()
                .userInfoEndpoint()
                .userService(customOAuth2UserService);

    }
}
```
**소스코드 해석**
```java
@EnableWebSecurity   

* Spring Security 설정들을 활성화시켜줍니다.     
_____________________________________________________________
http.csrf().disable().headers().frameOptions().disable()    

* h2-console 화면을 사용하기 위해 해당 옵션들을 disable합니다.   
_____________________________________________________________
.authorizeRequests()  

* URL 별 권한 관리를 설정하는 옵션의 시작점입니다.   
* authorizeRequests() 가 선언되어야만 andMatchers() 옵션을 사용할 수 있습니다.   
_____________________________________________________________
.andMatchers("url", "url2")    

* 권한 관리 대상을 지정하는 옵션입니다.   
* URL, HTTP 메소드별로 관리가 가능합니다.   
* "/" 등 지정된 URL들은 permitAll() 옵션을 통해 전체 열람 권한을 주었습니다.   
* "api/v1/**" 주소를 가진 API는 USER 권한을 가진 사람만 가능하도록 했습니다.  
_____________________________________________________________
.anyRequest()

* 설정된 값 이외 나머지 URL 들을 나타냅니다.  
* 여기서는 .authenticated()을 추가하여 나머지 URL 들은 모두 인증된 사용자들에게만 허용합니다.   
* 인증된 사용자 즉, 로그인 한 사용자들을 이야기합니다.   
_____________________________________________________________
.logout().logoutSuccessUrl("/")     
* 로그아웃 기능에 대한 여러 설정의 진입점입니다.   
* 로그아웃 성공시 / 주소로 이동합니다.  
_____________________________________________________________
.oauth2Login()   

* OAuth2 로그인 기능에 대한 여러 설정의 진입점입니다.   
_____________________________________________________________
.userInfoEndpoint()   

* OAuth2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당합니다.   
_____________________________________________________________
.userService(customOAuth2UserService)    

* 소셜 로그인 성공시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록합니다.       
* 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능을 명시할 수 있습니다.  
* **OAuth2UserService 인터페이스의 추상메서드인 loadUser를 사용한다.**     
   
```
설정 코드 작성이 끝났다면 ```CustomOAuth2UserService```클래스를 생성하자   
이 클래스는 구글 로그인 이후 가져온 사용자 정보들을 기반으로    
가입 및 정보수정, 세션 저장등의 기능을 지원해준다.       
      
**CustomOAuth2UserService**    
```java
package com.jojoldu.book.springboot.config.auth;

import com.jojoldu.book.springboot.config.auth.dto.OAuthAttributes;
import com.jojoldu.book.springboot.config.auth.dto.SessionUser;
import com.jojoldu.book.springboot.domain.user.User;
import com.jojoldu.book.springboot.domain.user.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.client.userinfo.DefaultOAuth2UserService;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserRequest;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserService;
import org.springframework.security.oauth2.core.OAuth2AuthenticationException;
import org.springframework.security.oauth2.core.user.DefaultOAuth2User;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Service;

import javax.servlet.http.HttpSession;
import java.util.Collections;

@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails()
                .getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());
    }

    private User saveOrUpdate(OAuthAttributes attributes){
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                .orElse(attributes.toEntity());

        return userRepository.save(user);
    }
}

```
**소스코드 해석**
```java
String registrationId = userRequest.getClientRegistration().getRegistrationId();

* 현재 로그인 진행중인 서비스를 구분하는 코드   
* 구글로 로그인, 네이버로 로그인하는지 구분하기 위해 사용되는 코드이다.   
_________________________________________________________________________________________
String userNameAttributeName =   
userRequest.getClientRegistration().getProviderDetails().getUserInfoEndpoint().getUserNameAttributeName();

* OAuth2 로그인 진행시 키가 되는 필드값을 이야기합니다. PK 같은 역할   
* 구글의 경우 기본적으로 코드를 지원하지만 ("sub") , 네이버 카카오등은 지원하지 않습니다.   
* 이후 네이버 로그인과 구글 로그인을 동시 지원할 때 사용할 것입니다.  
_________________________________________________________________________________________
OAuthAttributes attributes = 
OAuthAttributes.of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

* oAuth2User 는 OAuth2UserService 로 만들어진 OAuth2User 객체를 참조하는 변수이다.  
* OAuthAttributes attributes는 OAuth2UserService 를 통해 가져온 OAuth2User 클래스의 attribute를 담을 클래스입니다. 
* 이후 네이버 등 다른 소셜 로그인도 이 클래스를 사용합니다.   
* 이것은 우리가 직접 정의해주는 클래스로 밑에서 클래스를 작성할 것입니다.   
________________________________________________________________________________________  
httpSession.setAttribute("user", new SessionUser(user));

* 세션에 자용자 정보를 저장하기 위한 DTO 클래스  
* User 클래스를 사용하면 안되기에 SessionUser를 만들었다.   
* 왜 User 클래스를 사용하면 안되나요?**
만약 User 클래스를 그대로 사용했다면 다음과 같은 에러가 발생합니다.   

Failed to convert from type [java.lang.Object] 
to type [byte[]] for value 'com.jojoIdu.book.springboot.domain.user.User@4a43d6'    
이는 User 클래스에 직렬화를 구현하지 않았다는 의미의 에러입니다.     
그렇다면 User 클래스에 직렬화 코드를 넣으면 될까요? 그렇기에는 생각할 것이 많습니다.   

바로 User 클래스가  데이터베이스와 직접 연결되는 엔티티이기 때문입니다       
엔티티 클래스에는 언제 다른 엔티티와의 관계가 형성될지 모릅니다.       
   
예를 들면 @OneToMany , @ManyToMany등 자식 엔티티를 갖고 있다면      
직렬화 대상에 자식들까지 포함되니 성능 이슈, 부수 효과가 발생할 확률이 높습니다.      
      
그래서 직렬화 기능을 가진 DTO를 하나 추가로 만드는 것이 이후 운영 및 유지보수 때 많은 도움이 됩니다.  
```
구글 사용자 정보가 업데이트 되었을 때를 대비하여 update 기능도 같이 구현되었습니다.      
사용자의 이름이나, 프로필 사진이 변경되면 User 엔티티에도 반영이 됩니다.     
정확히 말하면 기존 Email 이 있다면 최신 정보로 받아오고         
Email 이 없다면 지금 정보로 Entity를 만들어라 하고 있습니다.    
   
CustomOAuth2UserService 클래스까지 생성되었다면 OAuthAttributes 클래스를 생성합니다.      
필자의 경우 OAuthAttributes는 DTO로 보기 때문에 config.auth.dto 패키지를 만들어 해당 패키지에 생성했습니다.   

**OAuthAttributes**     
```java   
package com.jojoldu.book.springboot.config.auth.dto;

import com.jojoldu.book.springboot.domain.user.Role;
import com.jojoldu.book.springboot.domain.user.User;
import lombok.Builder;
import lombok.Getter;

import java.util.Map;

@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture){
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes){
        System.out.println("registration="+registrationId);
        return ofGoogle(userNameAttributeName, attributes);
    }
    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes){
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity(){
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST)
                .build();
    }
}

```   
**소스코드 해석**
```java
public static OAuthAttributes of(){}   

* OAuth2User에서 반환하는 사용자 정보는 Map 자료구조 형태이기에 값 하나하나를 변환해야한다.   
__________________________________________________________________________________________
toEntity()    

* User 엔티티를 생성합니다.   
* OAuthAttributes 에서 엔티티를 생성하는 시점은처음 가입할 때입니다.   
* 가입할 때의 기본 권한을 GUEST로 주기 위해서 role 빌더값에는 Role.GUEST를 사용합니다.   
```   
OAuthAttributes 클래스 생성이 끝났으면 같은 패키지에 SessionUser 클래스를 생성합니다.     
   
**SessionUser**
```java
package com.jojoldu.book.springboot.config.auth.dto;

import com.jojoldu.book.springboot.domain.user.User;
import lombok.Getter;

import java.io.Serializable;

@Getter
public class SessionUser implements Serializable {

    private String name;
    private String email;
    private String picture;

    public SessionUser(User user){
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}

```
SessionUser에는 인증된 사용자 정보만 필요합니다.         
그 외에 필요한 정보들은 없으니 name, email, picture 만 필드로 선언합니다.          

## 2.3. 로그인 테스트   
스프링 시큐리티가 잘 적용되었는지 확인하기 위해 화면에 로그인 버튼을 추가하자   
```index.mustache```에 로그인 버튼과 로그인 성공 시 사용자 이름을 보여주도록 하자   

**index.mustache**    
```mustache  
{{>layout/header}}
    <h1>스프링 부트로 시작하는 웹 서비스 ver.2</h1>
    <div class="col-md-12">
        <!-- 로그인 기능 영역 -->
        <div class="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
                {{#userName}}
                    Looged in as : <span id="user">{{userName}}</span>
                    <a href="/logout" class="btn btn-info active" role="button">Logout</a>
                {{/userName}}
                {{^userName}}
                    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">
                        Google Login
                    </a>
                {{/userName}}
            </div>
        </div>
    </div>
    <br>
    <!-- 목록 출력 영역 -->
    <table class="table table-horizontal table-bordered">
        <thead class="thead-string">
            <tr>
                <th>게시글번호</th>
                <th>제목</th>
                <th>작성자</th>
                <th>최종수정일</th>
            </tr>
        </thead>
        <tbody id="tbody">
            {{#posts}}
                <tr>
                    <td>{{id}}</td>
                    <td><a href="/posts/update/{{id}}">{{title}}</a></td>
                    <td>{{author}}</td>
                    <td>{{modifiedDate}}</td>
                </tr>
            {{/posts}}
        </tbody>
    </table>
{{>layout/footer}}
```   
**소스코드 해석**
```mustache
{{#userName}}   

* 머스테치는 다른 언어와 같은 if문을 제공하지 않습니다.     
* true/false 여부만 판단할 뿐입니다.      
* 그래서 머스터치에서는 항상 최종값을 넘겨줘야 합니다.      
* 여기서도 역시 userName 이 있다면 userName을 노출시키도록 구성했습니다.    
______________________________________________________________________________
a href="/logout"    

* 스프링 시큐리티에서 기본적으로 제공하는 로그아웃 URL 입니다.   
* 즉, 개발자가 별도로 저 URL에 해당하는 컨트롤러를 만들 필요가 없습니다.   
* SecurityCofing 클래스에서 URL을 변경할 순 있지만 기본 URL을 사용해도 충분하니 여기서는 그대로 사용합니다.   
______________________________________________________________________________
{{^userName}}

* 머스테치 에서 해당 값이 존재하지 않는 경우에는 ^ 를 사용합니다.   
* 여기서는 userName이 없다면 로그인 버튼을 노출시키도록 구성했습니다.  
______________________________________________________________________________
a href = "/oauth2/authorization/google"     

* 스프링 시큐리티에서 기본적으로 제공하는 로그인 URL 입니다.   
* 로그아웃 URL과 마찬가지로 개발자가 별도의 컨트롤러를 생성할 필요가 없습니다.  
* 후에 네이버 로그인은 따로 설정을 해주어야 할 것입니다.  
```
```index.mustache```에서 userName을 사용할 수 있게 IndexController 에서 UserName을 model에 저장하자  

**IndexController**
```java
package com.jojoldu.book.springboot.web;

import com.jojoldu.book.springboot.config.auth.LoginUser;
import com.jojoldu.book.springboot.config.auth.dto.SessionUser;
import com.jojoldu.book.springboot.service.posts.PostsService;
import com.jojoldu.book.springboot.web.dto.PostsResponseDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;
    private final HttpSession httpSession;
    
    @GetMapping("/")
    public String index(Model model){
        
        model.addAttribute("posts", postsService.findAllDesc());
        Session User user= (SessionUser) httpSession.getAttribute("user");
        if(user != null){
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }

    @GetMapping("/posts/save")
    public String postsSave(){
        return "posts-save";
    }

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model){

        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post",dto);
        return "posts-update";
    }
}

```
**소스코드 해석**
```java
(SessionUser) httpSession.getAttribute("user");
    
* 앞서 작성된 CustomOAuth2UserService에서 로그인 성공 시 세션에 SessionUser를 지정하도록 구성했습니다.          
* 즉, 로그인 성공시 httpSession.getAttribute("user")에서 값을 가져올 수 있습니다.     
______________________________________________________________________________
if(user != null)   

* 세션에 저장된 값이 있을 때만 model에 userName 으로 등록합니다.      
* 세션에 저장된 값이 없으면 model엔 아무런 값이 없는 상태이니 로그인 버튼이 보이게 된다.   
```

***
# 3. 어노테이션 기반으로 개선하기     
일반적인 프로그래밍에서 개선이 필요한 나쁜 코드의 대표적인 예로 **같은 코드가 반복되는 부분**이 있습니다.   
같은 코드를 계속해서 복사 & 붙여넣기로 만든다면 이후에 수정이 필요할 때 모든 부분을 일일이 수정해줘야 합니다.  
이렇게 될 경우 유지보수성이 떨어질 수 밖에 없으며, 혹시나 수정이 반영되지 않은 부분이 생길 수 있습니다.   
   
```java
Session user = (SessionUser) httpSession.getAttribute("user");
```
위 코드는 그 대표적인 예로 값이 바뀔 경우 모든 소스를 바꿔줘야합니다.      
그렇기에 위 코드를 **메소드의 인자로 세션값을 바로 받을 수 있도록 변경해보겠습니다.**   
   
우선 ```config.auth``` 패키지에 ```@LoginUser``` 어노테이션을 생성합니다.    

**@LoginUser**
```java
package com.jojoldu.book.springboot.config.auth;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}

```

**소스코드 해석**
```java
@Target(ElementType.PARAMETER)    
  
* 이 어노테이션이 생성될 수 있는 위치를 지정합니다.         
* PARAMETER 로 지정했으니 메소드의 파라미터로 선언된 객체에서만 사용할 수 있습니다.    
* 이 외에도 클래스 선언문에 쓸 수 있는 TYPE등이 있습니다.      
______________________________________________________________________________
@Retention(RetentionPolicy.RUNTIME)     

* 어노테이션의 범위(?)라고 할 수 있겠습니다.     
* 어떤 시점까지 어노테이션이 영향을 미치는지 결정합니다.    
______________________________________________________________________________
@interface 

* 이 파일을 어노테이션 클래스로 지정합니다.       
* LoginUser 라는 이름을 가진 어노테이션이 생성되었다고 보면 됩니다.     
```
   
그리고 같은 위치에 ```LoginUserArgumentResolver``` 를 생성합니다.   
```LoginUserArgumentResolver``` 는 ```HandlerMethodArgumentResolver``` 인터페이스를 구현한 클래스입니다.   
   
```HandlerMethodArgumentResolver```는 한 가지 기능을 제공합니다.    
바로 조건에 맞는 경우의 메소드가 있다면     
```HandlerMethodArgumentResolver```의 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있습니다.        
   
**LoginUserArgumentResolver**   
```java
package com.jojoldu.book.springboot.config.auth;

import com.jojoldu.book.springboot.config.auth.dto.SessionUser;
import lombok.RequiredArgsConstructor;
import org.springframework.core.MethodParameter;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;

import javax.servlet.http.HttpSession;

@RequiredArgsConstructor
@Component
public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {

    private final HttpSession httpSession;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        boolean isLoginUserAnnotation = parameter.getParameterAnnotation(LoginUser.class) != null;
        boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());
        return isLoginUserAnnotation && isUserClass;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        return httpSession.getAttribute("user");
    }
}

```

**소스코드 해석**
```java
@Override
public boolean supportsParameter(MethodParameter parameter){ 사용자 정의}

* 컨트롤러 메서드의 특정 파라미터를 지원하는지 판단합니다.  
* 여기서는 파라미터에 @LoginUser 어노테이션이 붙어있고, 파라미터 클래스 타입이 SessionUser.class인 경우 true를 반환합니다.  
* 즉, @LoginUser SessionUser 이면 합격  
______________________________________________________________________________
@Override
public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
   NativeWebRequest webRequest, WebDataBinderFactory binderFactroy) throws Exception {사용자 정의}

* 파라미터에 전달할 객체를 생성합니다.   
* 여기서는 세션에서 객체를 가져와서 넣습니다.  
* 즉 소셜로그인 하면서 httpSession에 user로 값이 저장되었는데 그것을 꺼내서 
@LoginUser SessionUser user 에 넣어주겠다는 의미입니다.   
```
이제 이렇게 생성된 ```LoginUserArgumentResolver```를 스프링에서 인식될 수 있도록      
```WebMvcConfiguration```에 추가하도록 하자.         
config 패키지에 ```WebConfig``` 클래스를 생성           

**WebConfig**
```java
package com.jojoldu.book.springboot.config;

import com.jojoldu.book.springboot.config.auth.LoginUserArgumentResolver;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.util.List;

@RequiredArgsConstructor
@Configuration
public class WebConfig implements WebMvcConfigurer {
    private final LoginUserArgumentResolver loginUserArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers){
        argumentResolvers.add(loginUserArgumentResolver);
    }
}

``` 
```HandlerMethodArgumentResovler```는 항상              
```WebMvcConfigure``` 의 ```addArgumentResolvers()```를 통해 추가해야 합니다.            
다른 ```Handler-MethodArgumentResovler```가 필요한다면 같은 방식으로 추가해주면 됩니다.         
   
모든 설정이 끝났으니 처음 언급한 대로     
```IndexController``` 의 코드에서 반복되는 부분들을 모두 ```@LoginUser```로 개선하겠습니다.        
     
**IndexController**    
```java
package com.jojoldu.book.springboot.web;

import com.jojoldu.book.springboot.config.auth.LoginUser;
import com.jojoldu.book.springboot.config.auth.dto.SessionUser;
import com.jojoldu.book.springboot.service.posts.PostsService;
import com.jojoldu.book.springboot.web.dto.PostsResponseDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@RequiredArgsConstructor
@Controller
public class IndexController {

    private final PostsService postsService;

    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user){
        model.addAttribute("posts", postsService.findAllDesc());

        if(user != null){
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }

    @GetMapping("/posts/save")
    public String postsSave(){
        return "posts-save";
    }

    @GetMapping("/posts/update/{id}")
    public String postsUpdate(@PathVariable Long id, Model model){

        PostsResponseDto dto = postsService.findById(id);
        model.addAttribute("post",dto);
        return "posts-update";
    }
}

```

**소스코드 해석**
```java
@LoginUser SessionUser user   
  
* 기존에(User) httpSession.getAttribute("user")로 가져오던 세션 정보 값이 개선되었습니다.     
* 이제는 어느 컨트롤러든지 @LoginUser만 사용함녀 세션 정보를 가져올 수 있게 되었습니다.   
```   


***
# 4. 세션 저장소로 데이터베이스 사용하기   
지금 우리가 만든 서비스는 애플리케이션을 재실행하면 로그인이 풀립니다.        
이는 **세션이 내장 톰캣의 메모리에 저장되기 때문입니다.**    
   
기본적으로 세션은 실행되는 **WAS의 메모리에서 저장되고 호출**됩니다.      
메모리에 저장되다 보니 **내장 톰캣처럼 애플리케이션 실행 시 실행되는 구조에선 항상 초기화 되는 것**입니다.      
즉, **배포할 때마다 톰캣이 재시작**되는 것입니다.     
   
이외에도 한 가지 문제가 더 있습니다.   
2대 이상의 서버에서 서비스하고 있다면 **톰캣마다 세션 동기화** 설정을 해야만 합니다.    
그래서 실제 현업에서는 세션 저장소에 대해 다음의 3가지중 한 가지를 선택합니다.   

1. 톰캣 세션을 사용한다.  
   * 일반적으로 별다른 설정을 하지 않을 때 기본적으로 선택되는 방식입니다.       
   * 이렇게 될 경우 톰캣 (WAS)에 세션이 저장되기 때문에     
   2대 이상의 WAS가 구동되는 환경에서는 톰캣들 간의 세션 공유를 위한 추가 설정이 필요합니다.     
   
2. MySQL과 같은 데이터베이스를 세션 저장소로 사용합니다.
   * 여러 WAS 간의 공용 세션을 사용할 수 있는 가장 쉬운 방법입니다.      
   * 많은 설정이 필요 없지만, 결국 로그인 요청마다 DB IO가 발생하여 성능상 이슈가 발생할 수 있습니다.   
   * 보통 로그인 요청이 많이 없는 백오피스, 사내 시스템 용도에서 사용합니다.  
     
3. Redis, Memcached 와 같은 메모리 DB를 세션 저장소로 사용한다.     
   * B2C 서비스에서 가장 많이 사용하는 방식입니다.    
   * 실제 서비스로 사용하기 위해서는 Embedded Redis와 같은 방식이 아닌 외부 메모리 서버가 필요합니다.  
   
여기서 2번째 방식인 **데이터베이스를 세션 저장소로 사용하는 방식**을 선택하여 진행하겠습니다.  
선택한 이유는 **설정이 간단**하고 사용자가 많은 서비스가 아니며 비용 절감을 위해서입니다.   
   
이후 AWS에서 이 서비스를 배포하고 운영할 때를 생각하면 레디스와 같은 메모리 DB를 사용하기는 부담스럽습니다.   
왜냐하면, 레디스와 같은 서비스(엘라스틱 캐시)에 별도로 사용료를 지불해야 하기 때문입니다.   
    
사용자가 없는 현재 단계에서는 데이터베이스로 모든 기능을 처리하는게 부담이 적습니다.   
만약 본인이 운영 중인 서비스가 커진다면 한번 고려해 보고, 이 과정에서는 데이터베이스를 사용하겠습니다.  

## 4.1. spring-session-jdbc 등록  
```spring-session-jdbc```역시 현재 상태에선 바로 사용할 수 없습니다.    
spring web, spring jpa를 사용했던 것과 마찬가지로 의존성이 추가되어 있어야 사용할 수 있습니다.  
   
```Build.gradle``` 에 다음과 같이 의존성을 등록합니다.  

**build.gradle**
```gradle
compile('org.springframework.session:spring-session-jdbc')    
```    
그리고 ```application.properties```에 세션 저장소를 jdbc로 선택하도록 코드를 추가합니다.     
설정은 다음 코드가 전부입니다. 이 외에 설정할 것이 없습니다.      

**application.properties**
```
spring.session.store-type-jdbc   
```
모두 변경하였으니 다시 애플리케이션을 실행해서 로그인을 테스트한 뒤, h2-console로 접속합니다.   
h2-console을 보면 세션을 위한 테이블 2개(SPRING_SESSION, SPRING_SESSION_ATTRIBUTES)가 생성된 것을 볼 수 있습니다.    
**JPA로 인해 세션 테이블이 자동 생성**되었기 때문에 별도로 해야 할 일은 없습니다.  
방금 로그인했기 때문에 한 개의 세션이 등록돼 있는 것을 볼 수 있습니다.   
   
이렇게 세션 저장소를 데이터베이스로 교체했습니다.   
물론 지금은 기존과 동일하게 **스프링을 재시작하면 세션이 풀립니다.**     
이유는 H2 기반으로 스프링이 재 실행될 때 **H2도 재시작되기 때문입니다.**    
이후 AWS로 배포하게 되면 AWS의 데이터베이스 서비스인 RDS를 사용하게 되니     
이때부터는 세션이 풀리지 않습니다.   


***
# 5. 네이버로 로그인    
마지막으로 네이버로 로그인하는 기능을 추가해보겠습니다.     
  
## 5.1. 네이버 API 등록  
   
해당 키값들을 ```application-oauth.properties``` 에 등록해줍시다.       
네이버에서는 스프링 시큐리티를 공식 지원하지 않기 때문에     
그동안 Commmon-OAuth2Provider에서 해주던 값들도 전부 수동으로 입력해줘야 합니다.      

**application-oauth.properties**
```properties
spring.security.oauth2.client.registration.google.client-id=692886957287-663ep6r6ds8ee0oukr9f5mrqof57k6bj.apps.googleusercontent.com
spring.security.oauth2.client.registration.google.client-secret=pVQcqYjwp_7fYyYdOJfkd5rk
spring.security.oauth2.client.registration.google.scope=profile,email

#registration
spring.security.oauth2.client.registration.naver.client-id=qJONuabRwLzw44TBZavZ
spring.security.oauth2.client.registration.naver.client-secret=2fGz3R2u8B
spring.security.oauth2.client.registration.naver.redirect-uri={baseUrl}/{action}/oauth2/code/{registrationId}
spring.security.oauth2.client.registration.naver.authorization_grant_type=authorization_code
spring.security.oauth2.client.registration.naver.scope=name,email,profile_image
spring.security.oauth2.client.registration.naver.client-name=Naver

# provider
spring.security.oauth2.client.provider.naver.authorization_uri=https://nid.naver.com/oauth2.0/authorize
spring.security.oauth2.client.provider.naver.token_uri=https://nid.naver.com/oauth2.0/token
spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
spring.security.oauth2.client.provider.naver.user_name_attribute=response

```
**소스코드 해석**
```properties
user_name_attribute=response

* 기준이 되는 user_name 의 이름을 네이버에서는 response로 해야합니다.         
* 이유는 네이버의 회원 조회 시 반환되는 JSON 형태 때문입니다.          
```
   
네이버는 reponse라는 JSON 객체의 최상위 필드로 한번 더 묶여서 반환되었기 때문이다.      
즉, ```JSON{response:{email : ~ , nickName : ~, profile_image : ~}}``` 이런형태로 반환된다.   

** 네이버 오픈 API의 로그인 회원 결과**
```
```
스프링 시큐리티에선 **하위 필드를 명시할 수 없습니다.**   
최상위 필드들만 ```user_name```으로 지정이 가능합니다.    
즉, 반환된 JSON 값의 첫번째 key(필드)값만 사용이 가능하다는 뜻 (객체 필드의 키는 사용 불가능)    
   
네이버의 응답값 최상위 필드는 **resultCode, message, response** 입니다.    
이러한 이유로 스프링 시큐리티에서 인식 가능한 필드는 저 3개 중에 골라야 합니다.   
   
본문에서 담고 있는 response를 user_name으로 지정하고 이후       
**자바 코드로 response의 id를 user_name으로 지정하겠습니다.**     

## 5.2. 스프링 시큐리티 설정 등록  
우리는 구글 로그인을 등록하면서 대부분 코드를 확장성 있게 작성했습니다.   
그렇기에 네이버도 쉽게 등록할 수 있습니다.   
    
 ```OAuthAttributes.java```에 다음과 같이       
 **네이버인지 판단하는 코드와 네이버 생성자만 추가해줍니다.**         

**OAuthAttributes.java**
```java
package com.jojoldu.book.springboot.config.auth.dto;

import com.jojoldu.book.springboot.domain.user.Role;
import com.jojoldu.book.springboot.domain.user.User;
import lombok.Builder;
import lombok.Getter;

import java.util.Map;

@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture){
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes){
        System.out.println("registration="+registrationId);
        if("naver".equals(registrationId)){
            return ofNaver("id", attributes);
        }
        return ofGoogle(userNameAttributeName, attributes);
    }
    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes){
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }
    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes){
        Map<String, Object> response = (Map<String, Object>) attributes.get("response");

        return OAuthAttributes.builder()
                .name((String) response.get("name"))
                .email((String) response.get("email"))
                .picture((String) response.get("profile_image"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity(){
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST)
                .build();
    }
}

```
    
**index.mustache**   
```mustache
{{>layout/header}}
    <h1>스프링 부트로 시작하는 웹 서비스 ver.2</h1>
    <div class="col-md-12">
        <!-- 로그인 기능 영역 -->
        <div class="row">
            <div class="col-md-6">
                <a href="/posts/save" role="button" class="btn btn-primary">글 등록</a>
                {{#userName}}
                    Looged in as : <span id="user">{{userName}}</span>
                    <a href="/logout" class="btn btn-info active" role="button">Logout</a>
                {{/userName}}
                {{^userName}}
                    <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">
                        Google Login
                    </a>
                    <a href="/oauth2/authorization/naver" class="btn btn-secondary active" role="button">
                        Naver Login
                    </a>
                {{/userName}}
            </div>
        </div>
    </div>
    <br>
    <!-- 목록 출력 영역 -->
    <table class="table table-horizontal table-bordered">
        <thead class="thead-string">
            <tr>
                <th>게시글번호</th>
                <th>제목</th>
                <th>작성자</th>
                <th>최종수정일</th>
            </tr>
        </thead>
        <tbody id="tbody">
            {{#posts}}
                <tr>
                    <td>{{id}}</td>
                    <td><a href="/posts/update/{{id}}">{{title}}</a></td>
                    <td>{{author}}</td>
                    <td>{{modifiedDate}}</td>
                </tr>
            {{/posts}}
        </tbody>
    </table>
{{>layout/footer}}
```
**소스코드 해석**   
```mustache
/oauth2/authorization/naver
   
* 네이버 로그인 URL은 application-oauth.properties 에 등록한 return-uri 값에 맞춰 자동으로 등록됩니다.   
* /oauth2/authorization/ 까지는 고정이고 마지막 Path만 각 소셜 로그인 코드를 사용하면 됩니다.   
* 여기서는 naver 가 마지막 path 가 됩니다.
```

***
# 6. 기존 테스트에 시큐리티 적용하기   
마지막으로 **기존 테스트에 시큐리티 적용으로 문제가 되는 부분들을 해결해보겠습니다.**       
기존에는 바로 API를 호출할 수 있어 테스트 코드 역시 바로 API를 호출하도록 구성하였습니다.     
         
하지만, 시큐리티 옵션이 활성화 되면 인증된 사용자만 API를 호출할 수 있습니다.      
기존의 API 테스트 코드들이 모두 인증에 대한 권한을 받지 못하였으므로,    
테스트 코드마다 인증한 사용자가 호출한 것처럼 작동하도록 수정하겠습니다.      

우선 전체 테스트코드에 대해서 무엇이 에러인지 확인하기 위해 한번에 테스트를 진행해보겠습니다.   
인텔리제이 오른쪽 위에 ```[gradle]``` 탭을 클릭합니다.     
```[Tasks => verification => test]```를 차례로 선택해서 **전체 테스트를 수행합니다**   
   
[사진]  
test를 실행해 보면 다음과 같이 롬복을 이용한 테스트 외에 스프링을 이용한 테스트는 모두 실패하는 것을 확인할 수 있습니다.   
[사진]

## 6.1. CustomOAuth2UserService를 찾을 수 없음     

[사진]   
이는 ```CustomOAuth2UserService``` 를 생성하는데 필요한 소셜 로그인 관련 설정값들이 없기 때문에 발생합니다.        
하지만 우리는 분명 ```application-oatuh.properties```에 설정값들을 추가했는데 왜 설정이 없다고 할까요?        
   
이는 ```src/main``` 환경과 ```src/test``` 환경의 차이 때문입니다.     
둘은 본인만의 환경 구성을 가집니다.    
다만, ```src/main/resources/application-properties```가 테스트 코드를 수행할 때도 적용되는 이유는    
test 에 ```application.properties```가 없으면 main의 설정을 그대로 가져오기 때문입니다.   
다만, 자동으로 가져오는 옵션의 범위는 ```application.properties```파일 까지입니다.   
        
즉, ```application-oauth.properties```는 test파일에 없다고 가져오는 파일이 아니라는 점입니다.     
그래서 이 문제를 해결하기 위해서는 test 폴더에도 ```application-oauth.properties```의 값들을 작성해줘야한다.      
우리는 ```application-oauth.properties```를 새로 만들기보다         
해당 값들을 ```test/resources/application.properties```에 넣어주도록 하겠습니다.               
단, 실제로 연동해서 사용할 것이 아니기 때문에 **가짜 설정값을 등록해줍시다.**        
    
**test/resources/application.properties**
```properties
spring.jpa.show_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
spring.h2.console.enabled=true
spring.session.store-type=jdbc

# Test OAuth
spring.security.oauth2.client.registration.google.client-id=test
spring.security.oauth2.client.registration.google.client-secret=test
spring.security.oauth2.client.registration.google.scope=profile,email
```

 
## 6.2. 302 Status Code   
응답의 결과로 200(정상 응답) Status Code를 원했는데 결과는 302(리다이렉션 응답)Status Code가 와서 실패했습니다.   
이는 스프링 시큐리티 설정 때문에 **인증되지 않은 사용자의 요청은 이동시키기 때문입니다.**   
그래서 이런 API 요청은 **임의로 인증된 사용자를 추가하여 API만 테스트해 볼 수 있게 하겠습니다.**   
    
어려운 방법은 아니며, 이미 스프링 시큐리티에서 공식적으로 방법을 지원하고 있으므로 바로 사용해보겠습니다.   
**스프링 시큐리티 테스트를 위한 여러 도구를 지원하는**   ```spring-security-test```를 ```build.gradle```에 추가합니다.    

**build.gradle**
```gradle
    testCompile("org.springframework.security:spring-security-test")
```
   
그리고 PostsApiControllerTest 의 2개 테스트 메소드에 다음과 같이 임의의 사용자 인증을 추가합니다.

**PostsApiControllerTest**
```java
package com.jojoldu.book.springboot.web;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.jojoldu.book.springboot.domain.posts.Posts;
import com.jojoldu.book.springboot.domain.posts.PostsRepository;
import com.jojoldu.book.springboot.web.dto.PostsSaveRequestDto;
import com.jojoldu.book.springboot.web.dto.PostsUpdateRequestDto;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.*;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import java.util.List;

import static org.springframework.security.test.web.servlet.setup.SecurityMockMvcConfigurers.springSecurity;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @Autowired
    private WebApplicationContext context;

    @After
    public void tearDown() throws Exception{
        postsRepository.deleteAll();
    }
    
    @WithMockUser(roles="USER")
    @Test
    public void Posts_등록된다() throws Exception{
        // given
        String title = "title";
        String content = "content";
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                                                                .title(title)
                                                                .content(content)
                                                                .author("author")
                                                                .build();

        String url = "http://localhost:"+ port + "/api/v1/posts";

        //when 
        ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

        //then
        
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);
        
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }
    
    @WithMockUser(roles="USER")
    @Test
    public void Posts_수정된다() throws Exception{
        // given
        Posts savedPosts = postsRepository.save(Posts.builder()
                                                        .title("title")
                                                        .content("content")
                                                        .author("author")
                                                        .build());

        Long updateId = savedPosts.getId();
        String expectedTitle = "title2";
        String expectedContent = "content2";

        PostsUpdateRequestDto requestDto = PostsUpdateRequestDto.builder()
                                                                    .title(expectedTitle)
                                                                    .content(expectedContent)
                                                                    .build();

        String url = "http://localhost:"+ port + "/api/v1/posts/" + updateId;

        
        //when 
        ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

        //then
        
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);
        
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }

}
```

**소스코드 해석**   
```java
@WithMockUser(roles="USER")

* 인증된 모의(가짜) 사용자를 만들어서 사용합니다.    
* roles에 권한을 추가할 수 있습니다. 
* 즉, 이 어노테이션으로 인해 ROLE_USER 권한을 가진 사용자가 API를 요청하는 것과 동일한 효과를 가지게 됩니다.   
```  
이 정도만 하면 테스트가 될 것 같지만, 실제로 작동하지는 않습니다.       
**```@WithMockUser(roles="USER")``` 가 MockMvcd에서만 작동하기 때문입니다.**       
  
현재 PostsApiControllerTest는 ```@SpringBootTest```로만 되어있으며 MockMvc를 전혀 사용하지 않습니다.   
그래서 ```@SpringBootTest```에서 MockMvc를 사용하는 방법을 소개합니다.  

**PostsApiControllerTest**
```java
package com.jojoldu.book.springboot.web;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.jojoldu.book.springboot.domain.posts.Posts;
import com.jojoldu.book.springboot.domain.posts.PostsRepository;
import com.jojoldu.book.springboot.web.dto.PostsSaveRequestDto;
import com.jojoldu.book.springboot.web.dto.PostsUpdateRequestDto;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.*;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import java.util.List;

import static org.springframework.security.test.web.servlet.setup.SecurityMockMvcConfigurers.springSecurity;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @Autowired
    private WebApplicationContext context;

    @Autowired
    private MockMvc mvc;

    @Before
    public void setup(){
        mvc = MockMvcBuilders
                .webAppContextSetup(context)
                .apply(springSecurity())
                .build();
    }

    @After
    public void tearDown() throws Exception{
        postsRepository.deleteAll();
    }

    @Test
    @WithMockUser(roles = "USER")
    public void Posts_등록된다() throws Exception{
        // given
        String title = "title";
        String content = "content";
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                                                                .title(title)
                                                                .content(content)
                                                                .author("author")
                                                                .build();

        String url = "http://localhost:"+ port + "/api/v1/posts";

        //when
        mvc.perform(post(url)
            .contentType(MediaType.APPLICATION_JSON_UTF8)
            .content(new ObjectMapper().writeValueAsString(requestDto))).andExpect(status().isOk());

        //then
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }
    @Test
    @WithMockUser(roles = "USER")
    public void Posts_수정된다() throws Exception{
        // given
        Posts savedPosts = postsRepository.save(Posts.builder()
                                                        .title("title")
                                                        .content("content")
                                                        .author("author")
                                                        .build());

        Long updateId = savedPosts.getId();
        String expectedTitle = "title2";
        String expectedContent = "content2";

        PostsUpdateRequestDto requestDto = PostsUpdateRequestDto.builder()
                                                                    .title(expectedTitle)
                                                                    .content(expectedContent)
                                                                    .build();

        String url = "http://localhost:"+ port + "/api/v1/posts/" + updateId;

        //when
        mvc.perform(put(url)
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(new ObjectMapper().writeValueAsString(requestDto))).andExpect(status().isOk());

        //then

        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }

}

```
**소스코드 해석**     
```java   
@Before

* 매번 테스트가 시작되기 전에 
* 여기서는 MockMvc 인스턴스를 생성합니다.
_________________________________________________
mvc.perform(post(url)
   .contentType(MediaType.APPLICATION_JSON_UTF8)
   .content(new ObjectMapper().writeValueAsString(requestDto)))
   .andExpect(status().isOk());

* 생성된 MockMvc 를 통해 API를 테스트합니다.   
* 본문(body) 영역은 문자열로 표현하기 위해 ObjectMapper를 통해 문자열 JSON으로 변환합니다.   
```   
   
이제 Posts 테스트도 정상적으로 수행되었습니다!!!   

## 6.3. @WebMvcTest에서 CustomOAuth2UserService을 찾을 수 없음        
HelloControllerTest 는 앞선 테스트와 다른점은 바로           
```@SpringBootTest```가 아니라 ```@WebMvcTest```를 사용한다는 점입니다.        
    
1번 에러 수정을 통해서 스프링 시큐리티 설정은 잘 작동하지만,       
```@WebMvcTest```는 **CustomOAuth2UserService을 스캔하지 않기 때문입니다.**        
         
```@WebMvcTest```는 ```WebSecurityConfigurerAdapter```, ```WebMvcConfigurer``` 를 비롯한         
```@ControllerAdvice```, ```@Controller```를 읽습니다.        
이를 다르게 생각해보면 **```@Repository```, ```@Service```, ```@Component```는 스캔 대상이 아닙니다.**      
   
그러니 SpringConfig는 읽었지만   
SpringConfig를 생성하기 위해 필요한 ```CustomOAuth2UserService``` 는 읽을수가 없어 에러가 발생한 것이다.   
HelloController 에서는 OAuth2 관련 내용을 사용하지 않으므로 제외한 상태로 테스트가 가능하게끔 아래 코드를 입력해주자
그리고 여기서 
```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class,
        excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SecurityConfig.class)
        }
)
public class HelloControllerTest {
```
그리고 여기서 ```@WithMockUser(roles="USER")``` 를 메소드 위에 정의함으로써 가짜로 인증된 사용자를 생성합니다.   
        
**하지만**  다음과 같이 추가로 에러가 생긴 것을 알 수 있다.     
    
```
java.lang.IllegalArgumentException: At least one JPA metamodel must be present!   
```
이 에러는 Application.java의 ```@EnableJpaAuditing``` 으로 인해 발생합니다.      
```@EnableJpaAuditing```를 사용하기 위해선 최소 하나의 ```@Entity```클래스가 필요합니다.     
```@WebMvcTest```이다 보니 ```@Controller```를 제외한 어노테이션을 읽지 못하니 당연히 없다고 판단되어집니다.     
   
```@EnableJpaAuditing```가 ```@SpringBootApplication```과 함께 있다보니     
```@WebMvcTest``` 에서 ```@EnableJpaAuditing```를 스캔하게 되어서 사용된 것입니다.      
그렇기에 우리는 ```@EnableJpaAuditing```과 ```@SpringBootApplication``` 둘을 분리해보도록 하겠습니다.   
   
우선 ```Application.java``` 에서 ```@EnableJpaAuditing```를 주석처리로 지워주자     
      
**Application**
```java
// @EnableJpaAuditing
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
   
그리고 ```@EnableJpaAuditing```를 다른곳에서 설정해주기 위해     
config 패키지에 JpaConfig를 생성하여 ```@EnableJpaAuditing```를 추가해줍시다.     

**JpaConfig**
```java
@Configuration
@EnableJpaAuditing // JPA Auditing 활성화
public class JpaConfig {}
```
```@WebMvcTest```는 ```WebSecurityConfigurerAdapter```, ```WebMvcConfigurer``` 를 비롯한         
```@ControllerAdvice```, ```@Controller```를 읽지만 **일반적인 ```@Configuration```을 읽지는 못합니다.**      



