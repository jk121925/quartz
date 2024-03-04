
인증을 거친 사용자가 유효한 요청인지를 판단하는 것 **AccessDecisionManager (인가를 할 떄 사용한다.)**

- AffirmativeBased를 사용한다 → 구현체 : AccessDecisionVoter가 한개라도 true라면 허용한다.
- ConsensusBased: 다수결
- UnanimousBased: 만장일치

만약 인가가 되지 않는다면 예외를 던진다.

<aside>
💡 decision

configAttribute → return 값이 1이면 허용하는 상황

</aside>

### http.authorizeRequest vs http.authorizeHttpRequest?

<aside>
💡 The difference between `authorizeRequests`and `authorizeHttpRequests`
 is explained [here](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html) The `authorizeHttpRequests`
 uses the new simplified `AuthorizationManager`
 API and the `AuthorizationFilter`
, while `authorizeRequests`
 uses the `AccessDecisionManager`
 and `FilterSecurityInterceptor`
. The latter will be deprecated in future version of Spring Security.

</aside>

[authorizeRequests() vs. authorizeHttpRequests​(Customizer](https://stackoverflow.com/questions/73089730/authorizerequests-vs-authorizehttprequestscustomizerauthorizehttprequestsc)

요 위에서 가져온 건데.. → 초기에 저는 authorizeHttpRequest를 사용했지만 accessDecisionManager를 사용하는데 있어서 문제가 생겼다. 

~~결과적으로 설명하면 authroizeRequest는 약간에 옛날방식으로 사용이 되는 것 같은데~~ 

이는 accessDecisionManager와 filterSecurityInterceptor를 지원하고

authroizeHttpRequest는 AuthorizationFilter와 AuthorizationFilter를 지원한다.

좀더 정리하자면 spring doc.에서는 authorizeHttpRequest가 3가지 측면에서 좀더 이득이라고 설명한다.

1. authorizeHttpRequest를 사용하게 되면 AuthorizationManager를 사용하게 되면서 metadata sources, config attributes, decision managers 과 voter를 간결하게 사용할 수 있다. → customization도 가능하다.
2. Authentication look up을 지연할 수 있다고 말한다. 이전에는 (authorizeRequest)는 모등 요청마다 authentication을 찾았어야 했는데 authorizeHttpRequest를 이용한 Authentication은 Authorize된 객체만 확인을 하기 때문에 효율성이나 시간 면에서도 이득이라고 할 수 있다.
3. Bean-based configuration support를 지원한다.
![[Untitled 15.png]]
위의 도식처럼 인증된 객체가 확인이 되면 filterInvocation이 호출 되고 이후 httpServeltRequest와 httpServeltResponse와 filterChain이 만들어 지게 되면서 나머지 필요한 filter를 거치게 한다.

이후 모든 인증이 끝난 다음 AuthenticationManager가 호출되고 여기서 인가할 수 있는 인증자라면 자원에 접근가능하게 해주고 그렇지 않다면 AccessDeniedException을 발생시켜 적절한 page로 이동시킨다. (403 forbidden으로 response를 보낸다.)

### AccessDecisionManager or Voter 선정

- admin인데 user page에 접근이 안된다. → accessDecisionManager 혹은 Voter를 적절하게 해줘야한다.

```java
public SecurityExpressionHandler securityExpressionHandler() {
        RoleHierarchyImpl roleHierarchy = new RoleHierarchyImpl();
        roleHierarchy.setHierarchy("ROLE_ADMIN > ROLE_USER");

        DefaultWebSecurityExpressionHandler handler = new DefaultWebSecurityExpressionHandler();
        handler.setRoleHierarchy(roleHierarchy);

        return handler;
    }
```

```java
http.authorizeRequests()
                .antMatchers("/","/info").permitAll()
                .antMatchers("/admin").hasRole("ADMIN")
                .antMatchers("/user").hasRole("USER")
                .antMatchers(HttpMethod.POST,"/account").permitAll()
                .anyRequest().authenticated()
                        .expressionHandler(securityExpressionHandler());
        http.formLogin();
```

securityConfig가 위 코드 처럼 바뀌는데 expressionHandler에 securityExpressionHandler를 넣어주면 securityExpressionHandler에서 설정된 Hierarchy를 가지고 login을 진행하게 된다.

### AccessDecisionManager는 어디서 사용하는가?

accessDecisionManager는 filterChainProxy중 하나의 filter에서 사용이 된다.

- 거의 마지막 부분에 인가를 확인하는 상황으로 이해되면 된다.

 **AbstractSecurityIntercepter**

어느시점에 filter를 사용하는가?
![[Untitled 1 1.png]]

### ExceptionTranslationFilter

FilterSecurityIntercepter의 상위 클래스인 accessDecisionManager에서 발생한 

AuthenticationException 과 AccessDeniedException을 처리한다.

- AuthenticationException 은 인증 처리기 → 인증 예외 발생
- AccessDeniedException 은 인가 처리기 → 익명 사용자인지 아닌지 확인
    - 익명 사용자 → AuthenticationEntryPoint 실행
    - 익명 사용자가 아니라면 AccessDeniedHandler에게 위임