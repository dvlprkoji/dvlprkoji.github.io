---
title: Spring 서블릿 필터(Filter)와 스프링 인터셉터(Interceptor)
date: 2021-09-28 00:00:01
categories: [Spring, Web]
tags: [spring, filter, interceptor]     # TAG names should always be lowercase
---
# 서블릿 필터? 스프링 인터셉터?
웹 어플리케이션을 개발하다 보면 로그인 처리와 같이 대부분의 로직에서 공통으로 적용해야 할 기능들이 존재한다  
이렇게 애플리케이션 로직에서 공통으로 관심이 있는 것을 **공통 관심사(cross-cutting concern)**라고 한다  
이러한 공통 관심사는 Spring의 AOP를 통해 구현이 가능하지만,  
로직의 복잡성이 증가할 뿐만 아니라 향후 이와 관련된 로직이 변경되면 작성한 모든 로직을 수정해야 할 것이다
<br><br>
따라서 웹과 관련된 공통 관심사는 지금부터 알아볼 서블릿 필터나 스프링 인터셉터를 사용하는 것이 좋다    
웹과 관련된 공통 관심사를 처리할 때는 HTTP의 헤더나 URL의 정보들이 필요한데,   
서블릿 필터나 스프링 인터셉터는 HttpServletRequest를 제공한다  

### 서블릿 필터 소개

서블릿 필터는 서블릿이 지원하는 수문장이다  
필터의 특성은 다음과 같다
####필터 흐름
```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
```
필터를 적용하면 필터가 호출 된 다음 서블릿이 호출된다  
그래서 모든 고객의 요청 로그를 남기는 요구사항이 있다면 필터를 사용하면 된다  

#### 필터 제한
```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러             // 인가 사용자 (로그인 Success)
HTTP 요청 -> WAS -> 필터 -> X                            // 비인가 사용자 (로그인 Fail)
```

#### 필터 체인
필터는 체인으로 구성되는데, 중간에 필터를 자유롭게 추가할 수 있다  
예를 들어 로그를 남기는 필터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 필터를 만들 수 있다  
```
HTTP 요청 -> WAS -> 필터1 -> 필터2 -> 필터3 -> 서블릿 -> 컨트롤러        // 필터 체인
```

#### 필터 인터페이스
필터 인터페이스를 구현하고 등록하면, 서블릿 컨체이너가 필터를 싱글톤 객체로 생성하고, 관리한다  
```java
public interface Filter{
    public default void init() throws ServletException {}
    public void doFilter(ServletRequest request,
                            ServletResponse response,
                            FilterChain chain) throws IOException, ServletException;
    public default void destroy() {}
}
```
- ```init()``` : 필터 초기화 메서드
- ```doFilter()```: 고객의 요청이 올 때마다 해당 메서드가 호출된다
- ```destroy()``` : 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다
<br><br>
  
### 필터의 사용
서비스 접근 로그를 남기는 필터를 작성하고 적용해 보겠다  
먼저 로그 필터를 작성해 보자
```java
@Slf4j
public class LogFilter implements Filter{
    public void init() throws ServletException {
        System.out.println("Filter Initialize");
    }
    public void doFilter(ServletRequest request,
                         ServletResponse response,
                         FilterChain chain) throws IOException, ServletException{
        
        HttpServletRequest httpRequest = (HttpServletRequest)request;
        HttpServletResponse httpResponse = (HttpServletResponse)response;
        
        String randomUUID = UUID.randomUUID().toString();
        try{
            log.info("Request [{}][{}]", httpRequest.getRequestURI, randomUUID);
            chain.doFilter(request, response);
        } catch(Exception e){
            throw e;
        } finally{
            log.info("Response [{}][{}]", httpResponse.getResponseURI, randomUUID);
        }
    }
    public void destroy() {
        System.out.println("Filter Destroy");
    }
}
```
이제 작성한 필터를 등록해 보자  
우리는 Spring Boot를 사용하기 때문에 FilterRegistrationBean을 생성해 사용하면 된다
```java
@Configuration
public class WebConfig{
    
    @Bean
    public FilterRegistrationBean logFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new Logfilter());
        filterRegistrationBean.setOrder(1);
        filterRegistrationBean.addUrlPatterns("/*");
        return filterRegistrationBean;
    }
}
```
이제 프로그램을 실행하고 이 서비스로 요청을 보내면 console에 Request, Response 로그가 출력될 것이다


### 스프링 인터셉터 소개

스프링 인터셉터도 서블릿 필터와 같이 웹과 관련된 공통 관심 사항을 효과적으로 해결할 수 있는 기술이다 
서블릿 필터가 서블릿의 하위 기술이라면 스프링 인터셉터는 스프링 MVC가 제공하는 기술이다

####스프링 인터셉터 흐름
```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러
```
스프링 인터셉터는 스프링 MVC의 하위 기술이므로 스프링 MVC의 시작점인 디스패처 서블릿 이후에 등장한다  

#### 스프링 컨테이너 제한
```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러             // 인가 사용자 (로그인 Success)
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> X                   // 비인가 사용자 (로그인 Fail)
```

#### 스프링 인터셉터 체인
스프링 인터셉터도 체인으로 구성되는데, 중간에 필터를 자유롭게 추가할 수 있다  
예를 들어 로그를 남기는 인터셉터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 인터셉터를 만들 수 있다
```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터1 -> 인터셉터2 -> 인터셉터3 -> 컨트롤러        // 필터 체인
```

#### 스프링 인터셉터 인터페이스
스프링 인터셉터를 사용하려면 HandlerInterceptor 인터페이스를 구현하면 된다
```java
public interface HandlerInterceptor {
    
    default boolean preHandle(HttpServletRequest request,
                              HttpServletResponse response,
                              Object handler) throws Exception {}
    
    default void postHandle(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler,
                            @Nullable ModelAndView modelAndView) throws Exception {}
    
    default void afterCompletion(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Object handler,
                                 @Nullable Exception ex) throws Exception {}
}
```
- ```preHandle()``` : 컨트롤러 호출 전(정확히는 핸들러 어댑터 호출 전)에 호출된다
- ```postHandle()```: 컨트롤러 호출 후에 호출된다
- ```aftercompletion()``` : 뷰가 렌더링된 이후 호출된다. 오류가 발생해도 항상 호출된다
  <br><br>
  


### 스프링 인터셉터의 사용
서비스 접근 로그를 남기는 필터를 작성하고 적용해 보겠다  
먼저 로그 필터를 작성해 보자
```java
@Slf4j
public class LogInterceptor implements HandlerInterceptor{
    default boolean preHandle(HttpServletRequest request,
                              HttpServletResponse response,
                              Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();
        request.setAttribute(LOG_ID, uuid);
        
        if (handler instanceof HandlerMethod) {
            HandlerMethod hm = (HandlerMethod) handler;
        }
        log.info("REQUEST [{}][{}][{}]", uuid, requestURI, handler);
        return true;
    }

    default void postHandle(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler,
                            @Nullable ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    default void afterCompletion(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Object handler,
                                 @Nullable Exception ex) throws Exception {
        String requestURI = request.getRequestURI();
        String logId = (String)request.getAttribute(LOG_ID);
        log.info("RESPONSE [{}][{}]", logId, requestURI);
        if (ex != null) {
            log.error("afterCompletion error!!", ex);
        }
    }
}
```
preHandle에서 등록한 값을 postHandle이나 afterCompletion에서 사용하려면 request에 담아두어야 한다
이제 작성한 인터셉터를 등록해 보자  
WebMvcConfigurer 인터셉터를 구현한 WebConfig 클래스를 작성했다
```java
@Configuration
public class WebConfig implements WebMvcConfigurer{
    
    @Override
    public void addInterceptors(InterceptorRegistry registry){
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/*.ico", "/error");
    }
}
```
이제 프로그램을 실행하고 이 서비스로 요청을 보내면 console에 Request, Response 로그가 출력될 것이다