N---
title: Spring Security (3) - 인가(Authorization)
date: 2021-10-02 00:00:01
categories: [Spring, Security]
tags: [spring security, authorization]     # TAG names should always be lowercase
---
저번 포스팅에 Spring Security 인증(Authentication) 프로세스를 알아보고 직접 구현해보았다
이번에는 인가(Authorization) 프로세스를 알아보자

#Spring Security 인가 프로세스

##FilterSecurityInteceptor
인가를 담당하는 필터는 필터 체인의 가장 마지막에 있는 ```FilterSecurityInterceptor```이다  
이 필터는 실제 인가 처리를 하지 않으며 단순히 컨트롤러의 역할만 한다  
이 ```FilterSecurityInteceptor```는 인증 정보, 요청 정보, 권한 정보를 담아 AccessDecisionManager에게 전달한다


![image](https://user-images.githubusercontent.com/46219687/135704566-5a89db0a-fc2a-4cce-81e8-9ef01050eaf5.png)

Custom FilterSecurityInterceptor 필터를 작성하려면 총 3가지 멤버에 주입해야 하는데  
이것은 각각 ```SecurityMetadataSource```, ```AccessDecisionManager```, ```AuthenticationManager```이다

```java
@Bean
public FilterSecurityInterceptor CustomFilterSecurityInterceptor() throws Exception{    
    FilterSecurityInterceptor filterSecurityInterceptor = new FilterSecurityInterceptor();
    filterSecurityInterceptor.setSecurityMetadataSource(urlFilterInvocationSecurityMetadataSource);
    filterSecurityInterceptor.setAccessDecisionManager(affirmativeBased());
    filterSecurityInterceptor.setAuthenticationManager(authenticationManagerBean());    
}
```
##FilterInvocationSecurityMetadataSource
> (자원, 인가권한) 정보를 가지고 있는 ```requestMap```을 검사하여 요청정보와 매칭되는 권한정보를 ```FilterSecurityInterceptor```에게 전달한다

![image](https://user-images.githubusercontent.com/46219687/135706857-e3fc9765-b977-490a-8aa7-48eccad418a4.png)

requestMap에 저장된 권한정보들은 직접 작성할 수 있지만 최종적으로는 DB에서 가져오는 것이 목적이다  
다음은 DB로부터 자원/권한 정보를 얻어와 Bean으로 등록하는 FactoryBean을 작성한 것이다

```java
public class UrlResourcesMapFactoryBean implements FactoryBean<LinkedHashMap<RequestMatcher, List<ConfigAttribute>>> {

    private SecurityResourceService securityResourceService;

    public void setSecurityResourceService(SecurityResourceService securityResourceService) {
        this.securityResourceService = securityResourceService;
    }

    private LinkedHashMap<RequestMatcher, List<ConfigAttribute>> resourcesMap;

    public void init() {
            resourcesMap = securityResourceService.getResourceList();
    }

    public LinkedHashMap<RequestMatcher, List<ConfigAttribute>> getObject() {
        if (resourcesMap == null) {
            init();
        }
        return resourcesMap;
    }

	public Class<LinkedHashMap> getObjectType() {
        return LinkedHashMap.class;
    }

    public boolean isSingleton() {
        return true;
    }
}
```

```java
@Service
public class SecurityResourceSerivce{
    private ResourceRepository resourceRepository;
    
    public LinkedHashMap<RequestMatcher, List<ConfigAttribute>> getResourceList() {

        LinkedHashMap<RequestMatcher, List<ConfigAttribute>> result = new LinkedHashMap<>();
        List<Resources> resourcesList = resourcesRepository.findAllResources();

        resourcesList.forEach(re ->
                {
                    List<ConfigAttribute> configAttributeList = new ArrayList<>();
                    re.getRoleSet().forEach(ro -> {
                        configAttributeList.add(new SecurityConfig(ro.getRoleName()));
                        result.put(new AntPathRequestMatcher(re.getResourceName()), configAttributeList);
                    });
                }
        );
        log.debug("cache test");
        return result;
    }    
}

```

##인가처리 실시간 반영하기
관리자가 인가처리 정책을 바꾼 경우 실시간으로 정책을 수정해주어야 할 것이다  
이는 작성한 ```UrlFilterInvocationSecurityMetadataSource```에 다음 메서드만 추가함으로서 해결할 수 있다
```java
public void reload(){
    LinkedHashMap<RequestMatcher, List<ConfigAttribute>> reloadedMap = securityResourceService.getResourceList();
    Iterator<Map.Entry<RequestMatcher, List<ConfigAttribute>>> iterator = reloadedMap.entrySet().iterator();
    
    requestMap.clear();
    
    while(iterator.hasNext){
        Map.Entry<RequestMatcher, List<ConfigAttribute>> entry = iterator.next();
        requestMap.put(entry.getKey(), entry.getValue());
    }
}
```

##PermitAllFilter 구현하기
인증 및 권한심사를 할 필요가 없는 자원들("/", "/login", ... )을 미리 설정해 바로 접근이 가능하도록 하는 필터를 구현해보자  
부모의 ```beforeInvocation()```메서드를 호출하기 전, 미리 검사를 실행하는 코드를 작성하면 된다
    
![image](https://user-images.githubusercontent.com/46219687/135737313-b10b927f-dc2d-44fd-a1a9-f8367afaaac2.png)

```java
public PermitAllFilter(String... permitAllPattern) {
    createPermitAllPattern(permitAllPattern);
}

@Override
protected InterceptorStatusToken beforeInvocation(Object object) {
    boolean permitAll = false;
    HttpServletRequest request = ((FilterInvocation) object).getRequest();
    for (RequestMatcher requestMatcher : permitAllRequestMatcher) {
        if (requestMatcher.matches(request)) {
            permitAll = true;
            break;
        }
    }

    if (permitAll) {
        return null;
    }

    return super.beforeInvocation(object);
}
```


##계층 권한 
상위 계층 권한은 하위 계층 권한을 가지고 있어야 한다  
하지만 지금까지 작성한 코드는 이 계층 권한이 적용되지 않고 있는데 이를 수정해보자  

![image](https://user-images.githubusercontent.com/46219687/135737325-71830759-a801-4d70-9c3f-ab5aa650b5e7.png)

