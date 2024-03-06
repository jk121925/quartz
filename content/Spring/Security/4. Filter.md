
이번 장에서는 Security를 구성하는 filter에 대해서 알아본다.

### 1. WebAsyncManagerIntegrationFilter

다른 thread에서도 principal을 공유할 수 있게 해주는 filter

### 2. SecurityContextPersistenceFilter

이 filter를 이용해서 기존의 SecurityContext 정보를 가져오는 것

- 여러 요청간에 SecurityContext를 공유하게 해주는 filter
- 기존 Security Repository에서 가져온다. → **HttpSession에서 읽어온다.**
- 처음에 요청이 왔을 때는 새로운 httpSession을 가져온다.
- **모든 인증 필터 보다 앞에 와서** **principal정보를 담고 있는 정보를 가져온다. → 없다면 비어있는 Security Context를 만들어 준다.**

영속화 필터라고 이야기를 하는데 → jpa에서 처럼 in-memory에 데이터를 올리는 역할을 한다.

### 3. HeaderWriterFilter

**우리가 거의 설정할 일이 없는 filter이다… → 근데 알기는 하자 어차피 모르면 답이 없다.**

- 응답 헤더에 Security정보를 추가해주는 역할
- 여러 header writer를 가지고 있고 이를 header에 추가 해준다.
- 기본적으로 5개
    1. XContentTypeOptionsHeaderWriter : 마임 타입 스니핑 → 실행할 수가 없는 마임 타입인데도 시행할려고 시행하는 것
    
    마임이란??? 
    
    - 이게 nosniff라면 Content-Type에서 지정한 형식으로만 렌더링 된다.
    1. XXssProtectionHeaderWriter : XXss를 방어해준다. 브라우저에 내장된 filter가 있지만 서버에서도 한번 걸러주는 것이다. 
    2. CacheControlHeaderWriter : 기본적으로 Cache를 사용하지 못 하도록 하게 한다.
    - Cache는 정적인 데이터를 사용할 때 유용하지만 동적인 데이터를 사용하게 되면 민감한 정보가 노출 될 수 있다.
    1. HstsHeaderWriter : HTTPS로만 소통하도록 강제하는 헤더를 추가.
    2. XFrameOptionsHeaderWriter : clickjacking 방어 → 눈으로 보이지 않는 곳에 click을 심어서 이를 방지하는 header를 추가해준다.
    

### 4. CSRF filter

csrf(cross … … …) attack을 방어 → 내가 원하지 않는 요청을 임의대로 만들어서 보내는 것

중간에 제 3자가 정보를 훔쳐서 요청을 보내는 것

대부분의 요청이 same origin reqeust로 막혀있다. (react에서 spring으로 reqeust를 보낼때 → 다른 도메인에서 호출) → CORS 정책 : 이걸 활성화하면 CSRF에 노출될 가능성이 있다.

Spring Security에서는 CSRF filter를 이용해서 토큰을 생성하고 이를 맞추어서 공격을 감지한다.

CSRF token이 있는지 없는지를 확인한다.

CSRF token 생성 → 리소스에 접근할 때는 CSRF token을 항상 가지고 있어야한다. → 이후 서버가 가지고 있는 token을 가져온뒤 이를 대조해서 확인을 진행한다.

**CSRF token 사용방법**

딱히 신경은 쓰지 않아도 된다….

강의 내용은 jsp를 사용할 때 form tag를 사용하면 CSRF token을 그냥 가져와 준다.

test code 작성법 → mokmvc를 사용해서 확인은 하지 않겠습니다.

### 5. Logout

logouthandler → composit 객체로 다른 것을 감싸서 사용한다.

logoutsuccesshander → logout하고 난 뒤에 redirect하는 것

logout get 요청은 → defaultLogoutGenerateingFilter가 만들어준 것이다.

logout post요청은 logout을 누를 때 요청이 간다.

### 6. UsernamePasswordAuthenticationFilter

앞서서 배웠지만 post로 들어온 Username과 Password를 DB에 존재하는 User와 비교해서 인증을 진행해주는 것이다.

→ 이중 DaoAuthenticationProvider를 사용한다.

### 7. DefalutLogin/LogoutPageGeneratingFilter

로그인 페이지를 만들어주는 filter

```java
// 이 친구가 사용을 한다.
http.formLogin()

// Login page를 다시만들고 싶다면?
http.formLogin()
	.loginPage("/myloginPage")
// 이렇게 사용했을 떄 가능하다.
```

### 8. login logout customize

```java
// config를 변경해야한다.
http.formLogin()
	.loginPage("/myloginPage")

// 이후 login Controller를 만들어주고 post요청은 UsernamePasswordAuthenticationFilter를 사용한다.

```

### 9. BasicAuthenticationFilter

```java
// 요걸 달아 놨기 때문에 가능하다.
http.httpBasic()
```

http 요청에서 username:password를 BASE 64로 incoding하는 요청 

요청이 하나라도 sniffing된다면 인증정보가 노출된다.

### 10. RequestCacheAwareFilter

현재요청에 matching되는 관련있는 요청을 처리한다. 

- dashboard로 가고 싶다. → Authentication Manager가 login으로 보낸다. → 이후 로그인을 하면 Cache된 dashboard요청을 처리할 수 있게 해주는 요청이다.

### 11. SecurityContextHolderAwareRequestFilter

인증이 되었는지 안되었는지를 확인해서 전체 인증과 인가 로그인 아웃을 확인하는 filter

### 12.AnonymousAuthenticationFilter

아무도 인증을 하지 않은 요청일 때 Anonymous객체를 SecurityContextHolder를 만들어서 넣어준다.

- null object pattern

### 13. SessionManagementFilter

1. 세션 변조 방지 전략 설정
    - 세션 변조 : 공격자가 자신의 Session을 받고 다른 요청자에게 이 Session을 가지고 로그인하게 되면 개인정보가 나간다. → 이를 방지하기 위해서 세션을 방지한다.
    - ServletContainer에 따라서 달라진다. → Servlet 3.1이상에서 Session ID를 변경한다. = **changeSessiondID**
    - 유효하지 않은 세션을 리다이렉트 시킬 URL 설정
        - 하나의 계정을 여러개 사용할 수 있지만 하나만 쓰고 싶다면
        
        ```java
        http.sessionManagement()
        	.maximunSession({number})
        	.maxSessionPreventsLogin(false)
        // maxSessionPreventsLogin은 다른곳에서 로그인을 하면 false일 때 강제 로그아웃 해준다.
        ```
        
    - 세션 생성 전략
        
        ```java
        http.sessionMangement()
        	.sessionCreationPolicy(SessionCreationPolicy.STATELESS));
        //stateless한 application일 경우는 이렇게 사용할 수 있지만 form login의 경우는 적절하지 않다.
        
        ```
        

### 14. ExceptionTranslationFilter

다음 filter인 filterSecurityInterceptor를 try/catch로 감싸고 filter를 진행한다.

2가지 예외가 발생할수 있는데

1. Authentication Error
    
    인증문제 → AuthenticationEntryPoint를 통해서 해결 → 해당 유저를 인증이 가능한 페이지로 보낸다.
    
2. AccessDeniedException Error
    
    인가문제 → AccessDeniedHandler 403에러 메시지를 보내준다 (권한이 없다고 → forbidden)
    
    - page가 까리하니
    
    ```java
    http.exceptionHandling()
    	.accessDeniedPage("/access_denied")
    	//or
    	.accessDeniedHandler()
    	//이를 통해서 logger를 통해서 logging을 하는 것이 유효하다.
    ```
    

### 15. FilterSecurityInterceptor

인가에 관한 filter

Http 리소스 시큐리티 처리를 담당하는 필터, AccessDecisionManager를 이용하여 인가를 처리한다.

### 16. RememberMeAuthenticationManager

로그인 기억하기에 사용한다.

Session이 만료가 되어도 cookie나 token이 서버에 저장되어있거나 한다.

```java
http.rememberMe()
	.userDetailsService(accountService)
	.key("remember-me-sample");
```

### Custom filter

```java
// 새로운 객체
public somthingFilter extends GenericFilterBean{

	public void doFilter(ServletRequest request, ServeletResponse response, FilterChain chain) throws Exception{
		//do Something ... 
		//이부분을 넣어줘야 다음 filter로 넘어간다.		
		chain.doFilter(request,response)

	}
}
```

```java
//config에 filter를 추가해 준다.
http.addFilter(new somthingFilter(), WebAsyncManagerIntegrationFilter.class)
//이렇게 넣어주면 추가가 된다.
```