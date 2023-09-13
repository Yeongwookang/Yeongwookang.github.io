---
title : JWT 인증 구현 후 테스트 불가 오류
date : 2023-09-13 00:00:00 +09:00
categories : [Project, RESTAPI]
tags : [project, springboot, test, spring, restapi]
mermaid: true
---

## 원인
JWT 인증 구현이 직접적인 원인이다.

### 통합테스트 (@SpringBootTest)
```@SpringBootTest``` 어노테이션의 경우 JWT인증을 받지 않아 ```403 FORBIDDEN``` 응답을 받았다.

### 단위테스트 (@WebMvcTest)
```@WebMvcTest(TodoController.class)  ```, ```@WithMockUser``` 어노테이션을 사용하여 Test를 진행하였는데, MockUser가 JWT인증을 받지 않아 ```403 FORBIDDEN``` 응답을 받았다.

<br>

## 해결

### 통합테스트
아직 해결하지 못했다.
서버를 켜서 Postman으로 하면 정상적으로 작동하는데, SecurityContext에 인증 정보가 안들어가서 그런것으로 추정하고 있다.

### 단위 테스트
MockUser가 임의의 SecurityContext상에서 JWT인증을 받도록 하면된다.

#### WithMockCustomUser

	@Retention(RetentionPolicy.RUNTIME)  
	@WithSecurityContext(factory = WithMockCustomUserSecurityContextFactory.class)  
	public @interface WithMockCustomUser {  
	    String account() default "admin";  
	    String name() default "administrator";  
	    String role() default "USER";  
	}

새로운 커스텀 MockUser를 어노테이션으로 만들어 주었다.

#### WithMockCustomUserSecurityContextFactory 

	public class WithMockCustomUserSecurityContextFactory implements WithSecurityContextFactory<WithMockCustomUser> {  
	  
	    @Override  
	  public SecurityContext createSecurityContext(WithMockCustomUser annotation) {  
	        final SecurityContext securityContext = SecurityContextHolder.createEmptyContext();  
	  
	        var member = Member.builder()  
	                .name(annotation.name())  
	                .account(annotation.account())  
	                .password("admin")  
	                .createdAt(LocalDateTime.now())  
	                .role(Role.ADMIN)  
	                .build();  
	  
	        final UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(  
	            member, null, member.getAuthorities()  
	        );  
	        securityContext.setAuthentication(authenticationToken);  
	        return securityContext;  
	    }  
	}

실제 인증과 비슷하게 member(UserDetails를 implement 받음)를 빌더로 구성하였다. 비밀번호는 저장되지 않으므로 인코딩 하지 않고 그냥 보냈다.

이후 Test파일의 ```@WithMockUser``` 어노테이션을 ```@WithCustomMockUser```로 대체하였다.

![성공 사진]("/assets/img/restApiTrouble/001.png")

## Reference
- Spring Security Docs : [https://docs.spring.io/spring-security/reference/servlet/test/index.html](https://docs.spring.io/spring-security/reference/servlet/test/index.html)