# 프로젝트 소개

Spring Boot 2.1 기반으로 Spring Security OAuth2를 살펴보는 프로젝트입니다. Authorization Code Grant Type, Implicit Grant, Resource Owner Password Credentials Grant, Client Credentials Grant Type OAuth2 인증 방식에 대한 간단한 셈플 코드부터 OAuth2 TokenStore 저장을 mysql, redis 등 저장하는 예제들을 다룰 예정입니다. 계속 학습하면서 정리할 예정이라 심화 과정도 다룰 수 있게 될 거 같습니다. 지속해서 해당 프로젝트를 이어 나아갈 예정이라 깃허브 Start, Watching 버튼을 누르시면 구독 신청받으실 수 있습니다. 저의 경험이 여러분에게 조금이라도 도움이 되기를 기원합니다.

# 목차

<!-- TOC -->

- [프로젝트 소개](#%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%86%8C%EA%B0%9C)
- [목차](#%EB%AA%A9%EC%B0%A8)
- [OAuth2 승인 방식의 종류](#oauth2-%EC%8A%B9%EC%9D%B8-%EB%B0%A9%EC%8B%9D%EC%9D%98-%EC%A2%85%EB%A5%98)
    - [Authorization Code Grant Type 방식](#authorization-code-grant-type-%EB%B0%A9%EC%8B%9D)
        - [Code](#code)
        - [인증](#%EC%9D%B8%EC%A6%9D)
        - [API 호출](#api-%ED%98%B8%EC%B6%9C)
    - [Implicit Grant 방식](#implicit-grant-%EB%B0%A9%EC%8B%9D)
    - [Resource Owner Password Credentials Grant 방식](#resource-owner-password-credentials-grant-%EB%B0%A9%EC%8B%9D)
    - [Client Credentials Grant Type 방식](#client-credentials-grant-type-%EB%B0%A9%EC%8B%9D)
- [참고](#%EC%B0%B8%EA%B3%A0)

<!-- /TOC -->
# OAuth2 승인 방식의 종류

* Authorization Code Grant Type  : 권한 부여 코드 승인 타입
클라이언트가 다른 사용자 대신 특정 리소스에 접근을 요청할 떄 사용됩니다. 리스소 접근을 위한 사용자 명과 비밀번호, 권한 서버에 요청해서 받은 권한 코드를 함ㄲ 활용하여 리소스에 대한 엑세스 토큰을 받는 방식입니다.
* Implicit Grant Type : 암시적 승인
권한 부여 코드 승인 타입과 다르게 권한 코드 교환 단계 없이 엑세스 토큰을 즉시 반환받아 이를 인증에 이용하는 방식입니다.
* Resource Owner Password Credentials Grant Type : 리소스 소유자 암호 자격 증명 타입
클라이언트가 암호를 사용하여 엑세스 토큰에 대한 사용자의 자격 증명을 교환하는 방식식입니다.
* Client Credentials Grant Type : 클라이언트 자격 증명 타입
클라이언트가 컨텍스트 외부에서 액세스 토큰을 얻어 특정 리소스에 접근을 요청할 때 사용하는 방식입니다.

## Authorization Code Grant Type 방식

![oauth2-doe-grant-type](https://github.com/cheese10yun/TIL/raw/master/assets/oauth2-doe-grant-type_gnojt19me.png)

* (1) 클라이언트가 파리미터러 클라이언트 ID, 리다이렉트 URI, 응답 타입을 code로 지정하여 권한 서버에 전달합니다. 정상적으로 인증이 되면 권한 코드 부여 코드를 클라이언트에게 보냅니다.
  - 응답 타입은 code, token 이 사용 가능합니다.
  - **응답 타입이 token 일 경우 암시적 승인 타입에 해당합니다.**
* (2) 성공적으로 권한 부여 코드를 받은 클라이언트는 권한 부여 코드를 사용하여 엑세스 토큰을 권한 서버에 추가로 요청합니다. 이때 필요한 파라미터는 클라이언트 ID, 클라이언트 비밀번호, 리다이렉트 URI, 인증 타입 입니다.
* (3) 마지막으로 받은 엑세스 토큰을 사용하여 리소스 서버에 사용자의 데이터를 보냅니다.

### Code

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients
                .inMemory() // (1)
                .withClient("client") //(2)
                .secret("{bcrypt} $2a$10$iP9ejueOGXO29.Yio7rqeuW9.yOC4YaV8fJp3eIWbP45eZSHFEwMG")  //(3) password
                .redirectUris("http://localhost:9000/callback") // (4)
                .authorizedGrantTypes("authorization_code") // (5)
                .scopes("read_profile"); // (6)
    }
}
```
* (1): 간단한 설정을 위해 메모리에 저장시키겠습니다. 
* (2): 클라이언트 이름을 작성합니다.
* (3): 시크릿을 작성해야합니다. 스프링 시큐리티 5.0 부터는 암호화 방식이 조 변경되서 반드시 암호화해서 저장하는 것을 권장합니다. 해당 암호는 password입니다. (현재 프로젝트는 스프링부트 2.1 이기 때문에 스프링 시큐리티 5.0 의존성을 주입받습니다.)
* (4): 리다이렉트 URI을 설정합니다.
* (5): `Authorization Code Grant` 타입을 설정합니다.
* (6): scope를 지정합니다.


```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
    
    @Override
    public void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().anyRequest().authenticated().and()
                .requestMatchers().antMatchers("/api/**");
    }
}

@EnableWebSecurity
@AllArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //@formatter:off
		http
                .csrf().disable()
                .authorizeRequests().anyRequest().authenticated().and()
                .formLogin().and()
                .httpBasic();
        //@formatter:on
    }
}
```
* 리소스 서버 설정은 configure 설정만 간단하게 설정합니다.
* 시큐리티 설정은 기본설정에서 csrf 설정만 disable 설정했습니다.


### 인증
[http://localhost:8080/oauth/authorize?client_id=client&redirect_uri=http://localhost:9000/callback&response_type=code&scope=read_profile](http://localhost:8080/oauth/authorize?client_id=client&redirect_uri=http://localhost:9000/callback&response_type=code&scope=read_profile) 해당 페이지로 이동하면 아래와 같이 로그인 페이지(스프링 시큐리티 기본폼 이전보다 많이 이뻐졌다...)로 리다이렉트 됩니다.

![oauth2-login](/assets/oauth2-login.png)
로그인 정보를 입력합니다. 소셜 가입에서 해당 소셜의 아이디로 로그인하는 것과 동일한 행위입니다.

```
username: user
password: pass
```
application.yml 에서 설정한 security user 정보를 입력해줍니다.


![oauth-code](/assets/oauth-prove.png)
유저 정보 인증이 완료되면 scope에 대한 권한 승인이 페이지가 나옵니다. 소셜 가입에서 프로필 정보, 프로필 사진 등을 요구하는 것과 마찬가지입니다.

![oauth-code](/assets/oauth-code.png)
권한 승인이 완료하면 권한 코드가 전송됩니다. [Authorization Code Grant Type 방식](#authorization-code-grant-type-%EB%B0%A9%EC%8B%9D) 에서 말한 `권한 부여 코드`를 응답받은 것입니다. 

리다이렉트된 페이지에서 위에서 발급 받은 `권한 부여 코드`로 권한 서버에 `Access Token`을 요청 받을 수 있습니다. 이 부분이 실제로 구현하지 않았고 넘겨 받은 `권한 부여 코드`를 기반으로 권한 서버에 수동으로 호출 해보겠습니다.

아래의 curl을 실행 해보면 응답값을 확인해 볼 수 있습니다.
```curl
curl -X POST \
  http://localhost:8080/oauth/token \
  -H 'Authorization: Basic Y2xpZW50OnBhc3N3b3Jk' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'code=xoS4mt&grant_type=authorization_code&redirect_uri=http%3A%2F%2Flocalhost%3A9000%2Fcallback&scope=read_profile'
```
만약 IntelliJ를 사용하신다면 `api.http`를 이용해서 더 쉽게 호출 해볼 수 있습니다.
![ouath2-http](/assets/ouath2-http.png)


```json
{
    "access_token": "623d5bc4-7172-44ae-85c1-73a297e6ab04",
    "token_type": "bearer",
    "expires_in": 43199,
    "scope": "read_profile"
}
```

### API 호출
```
curl -X GET \
  http://localhost:8080/api/session \
  -H 'Authorization: Bearer 623d5bc4-7172-44ae-85c1-73a297e6ab04'
```

curl을 이용해서 요청을 보내면 아래와 같이 응답값을 확인할 수 있습니다.
```json
{
    "authorities": [],
    "details": {
        "remoteAddress": "0:0:0:0:0:0:0:1",
        "sessionId": null,
        "tokenValue": "623d5bc4-7172-44ae-85c1-73a297e6ab04"
    }
    ....
}
```

물론 token 정보를 넘기지않거나 유효하지 않으면 401 응답을 받습니다.


## Implicit Grant 방식

![Implicit Grant](https://github.com/cheese10yun/TIL/raw/master/assets/Implicit%20Grant.png)

* (1) 클라이언트가 파리미터러 클라이언트 ID, 리다이렉트 URI, 응답 타입을 code로 지정하여 권한 서버에 전달합니다. 정상적으로 인증이 되면 권한 코드 부여 코드를 클라이언트에게 보냅니다.
  - 응답 타입은 code, token 이 사용 가능합니다.
  - **응답 타입이 token 일 경우 암시적 승인 타입에 해당합니다.**
* (2) 응답 해준 Access Token 이 유효한지 검증 요청을 합니다.
* (3) 요청 받은 Access Token 정보에 대한 검증에 대한 응답값을 돌려줍니다.
* (4) 유효한 Access Token 기반으로 Resource Server와 통신합니다.

## Resource Owner Password Credentials Grant 방식

![Resource Owner Password Credentials Grant](https://github.com/cheese10yun/TIL/raw/master/assets/Resource%20Owner%20Password%20Credentials%20Grant.png)

* (1) 인증을 진행합니다. 대부분 ID, Password를 통해서 자격 증명이 진행됩니다.
* (2) 넘겨 받은 정보기반으로 권한 서버에 Access Token 정보를 요청합니다.
* (3) Access Token 정보를 응답 받습니다. 이때 Refresh Token 정보도 넘겨 줄 수도 있습니다.
* (4) Access Token 기반으로 Resource Server와 통신합니다.

## Client Credentials Grant Type 방식

![Client Credentials Grant Type](https://github.com/cheese10yun/TIL/raw/master/assets/Client%20Credentials%20Grant%20Type.png)

* (1) Access Token 정보를 요청합니다.
* (3) Access Token 정보를 응답합니다. 이때 Refresh Token 정보는 응답하지 않는 것을 권장합니다. 별다른 인증 절차가 없기 떄문에 Refresh Token 까지 넘기지 않는 것이라고 생각합니다.
* (4) Access Token 기반으로 Resource Server와 통신합니다.


# 참고
* [OAuth2 이해하기](http://www.bubblecode.net/en/2016/01/22/understanding-oauth2/)
* [처음으로 배우는 스프링 부트2](http://www.hanbit.co.kr/store/books/look.php?p_code=B4458049183)
* [OAuth 2.0 쿡북](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791161752211&orderClick=LAG&Kc=)