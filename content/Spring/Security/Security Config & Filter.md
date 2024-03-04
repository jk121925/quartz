### Config

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends ~~WebSecurityConfigurerAdapter~~ {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests()
                .mvcMatchers("/","info").permitAll()
                .mvcMatchers("/admin").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .httpBasic();
    }
}
```

일딴 코드를 보면 위와 같게 써볼 수 있는데, 강의에서는 mvc 패턴을 사용해서 mvcMatchers를 사용한 것 같지만 restController Annotation을 써도 같게 사용할 수 있다. 

여기서 configure는 WebSecurityConfigurerAdapter에서 구현된 configure를 Override해서 사요하고 있는데 **httpSecurity객체를 받아서 해당 http request의 url를 Pattern matching시키서 원하는 패턴만 허용**을 하고 나머지는 상황에 따라서 다르게 허가를 하는 것을 볼 수 있다.

여기서 문제는 WebSecurityConfigurerAdapter가 더이상 지원을 하지 않을 예정이라는 것이다.

이글은 Config에 대한 서술 위주로 이야기를 해보겠다.

### FilterChain

[Securing a Web Application](https://spring.io/guides/gs/securing-web/)

사실 이 부분과 oauth를 환경 변수로 등록하는 부분때문에 강의를 들어볼라고 했는데 자세히는 설명을 해주시지 않는다.

따로 공부해보면 → 이전에 공부했던 것으로 미루어 보아 SpringSecurity는 Filter 객체를 이용해서 내부적으로 구현이 되어있는 것으로 알고 있다.

Filter는 request가 들어올 때 특정한 조건을 만족하면 해당 필터의 기능을 활성화 한뒤 response를 보내거나 다음 Filter로 순서를 forwarding하는 것으로 알고 있다.

아래는 filter함수의 기본적인 형태가 된다.

```java
public class Filter implements Filter {
    public void init(FilterConfig config) throws ServletException {
			// filter instance를 초기화 하기 위한 함수
    }

    public void destroy() {
			// filter를 종료시키기 전에 부르는 함수
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
      // filter가 하는 로직  
			chain.doFilter(request, response);
				
    }
}
```

아래는 WebSecurityConfigurerAdapter를 까보면서 올라간 코드들의 일부이다.

```java
public abstract class WebSecurityConfigurerAdapter implements WebSecurityConfigurer<WebSecurity> {...}
//WebSecurityConfigurerAdapter는 WebSecurityConfigurer를 구현한 것이고
public interface WebSecurityConfigurer<T extends SecurityBuilder<Filter>> extends SecurityConfigurer<Filter, T>{...}
//WebSecurityConfigurer는 SecurityBuilder<Filter>>를 상속받은 것인데
public interface Filter {

    
    public default void init(FilterConfig filterConfig) throws ServletException {}

    
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    
    public default void destroy() {}
}
//마지막 Filter를 보면 위의 Servlet code와 개 똑같은 것을 알 수 있다.
```

결과적으로 이야기 해보면 일 딴 Config는 filter를 상속받아서 filterChain을 만든 것이라고 할 수 있다.!

그럼 문제는 WebSecurityConfigurerAdapter는 더이상 지원을 하지 않는 다는데 뭘로 만들꺼냐?!

새로운 SecurityFilterChain을 통해서 만든다고 한다! 아래와 같다! 아주 비슷하지만 상속을 받는 것이 아니라 객체를 선언해서 bean에 등록해주면 된다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig{

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception{
        http.authorizeHttpRequests((requests) -> requests
                .antMatchers("/","/info").permitAll()
                .antMatchers("/admin").hasRole("ADMIN")
                .anyRequest().authenticated()
                )
                .formLogin();
        return http.build();
    }
}

// post url을 허용하고 싶으면 http.csrf().disable();를 달아줘야한다.
// csrf -> Cross Site Request Forgery 
// 이게 다는 아닐 것 같지만(현업에서) 일딴은 이렇게 하고 넘어간다.
```

결과적으로 앞으로는 SecurityFilterChain을 사용하면 되고 같은 방식으로 url pattern을 매칭시켜 인가 범위를 적용해 주면 된다.

### UserDetailsService

보통 우리는 inmemoryDB를 잘사용하지 않는다. 이때 DB에서 user객체를 가져와서 request에 담긴 객체를 확인해야 하는데 이때 DB에서 유저정보를 가져와서 담는 VO로 사용되는 UserDetailsService이다.

```java
//UserDetails
public interface UserDetails extends Serializable {

	Collection<? extends GrantedAuthority> getAuthorities();

	String getPassword();

	String getUsername();

	boolean isAccountNonExpired();

	boolean isAccountNonLocked();

	boolean isCredentialsNonExpired();

	boolean isEnabled();

}
```

위에는 userDetails객체인데 여기에는 password, username의 getter가 있으며 함수를 통해서 인증이 만료되었는지 사용가능한 객체인지 확인할 수 있게 되어있다. 아마도 따로 상속해서 구현하면서 작업을 하는 것 같다.

```java
//UserDetailsService
public interface UserDetailsService {

	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;

}
```

우리가 구현하려는 UserDetailsService는 username을 가져와서 UserDetails를 return 하는 interface이다.

```java
@Service
public class AccountService implements UserDetailsService {
@Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Account  account = accountRespostory.findByUsername(username);
        if(account ==null){
            throw new UsernameNotFoundException(username);
        }
        return User.builder()
                .username(account.getUsername())
                .password(account.getPassword())
                .roles(account.getRole())
                .build();
    }
}
```

AccountService를 UserDetailsService를 상속?해서 구현을 하게 되면 loadUserByUsername을 구현하게 되어있다. 이는 repository에서 username을 가져와서 없다면 Exception을 발생시키고 있다면 UserDetails를 return하게 한다.

- 이 때 Security에서 제공하는 User객체의 builder를 이용하면 쉽게 UserDetails를 return 할 수 있다.

**결과적으로 UserDetailsService를 상속받아서 구현한 Service 객체는 UserDetails를 return함으로써 각 request에 인증된 유저인지 아닌지를 확인한다. → 내가 볼때는 구현을 어떻게 하느냐에 따라서 달라진다.**

### PasswordEncoder

Security에서는 (내가 이해한 바로는) hasing을 통해서 비밀번호를 암호화하고 이를 해독한다. 이때 앞에 {noop}라는 문자열을 붙여야 한다. 기본적으로 스프링이 암호화를 해주지만 이를 사용자가 활용할 수 있도록 passwordEncoder를 이용해서 암호화를 진행한다,.

{noop}로 encoding되어야 로그인이 가능하다.

```java
@SpringBootApplication
public class SpringSecurityApplication {

    @Bean
    public PasswordEncoder passwordEncoder(){
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringSecurityApplication.class, args);
    }

}
```

PasswordEncoder 를 PasswordEncoderFactories로 부터 잡와서 passwordEncoder를 만들어 줘서 bean에 등록해줘야한다. → 이래야 스프링에서 사용하는 암호sequence를 사용할 수 있다.

이후 Account에 encoder를 하나 달아주면 로그인이 가능해진다.

```java
//Account
//...
public class Account {
...
public void encodePassword(PasswordEncoder passwordEncoder){
        this.password = passwordEncoder.encode(this.password);
    }
}
```

위에 처럼 encodePassword를 통해서 해당 객체의 비밀번호를 encoding해준다.