---
title : RestAPI JWT 인증 구현
date : 2023-09-13 00:00:00 +09:00
categories : [Project, RestAPI]
tags : [project, java, spring, restapi, jwt] 
mermaid: true
---

## 구현
```Refresh Token``` 의 구현은 하지 않고 ```Access Token```의 구현만 하였다.

### 환경
 ```spring boot:3.1.3```, ```Spring Security:6.1.3```, ```jjwt:0.11.5``` 을 사용하였다.

### Controller 

	@RestController  
	@RequestMapping("/api/v1/auth")  
	@RequiredArgsConstructor  
	public class authenticationController {  
	  
	    private final AuthenticationService service;  
	  
	    @PostMapping("register")  
	    public ResponseEntity<AuthenticationResponse> register(  
	            @RequestBody RegisterRequest request  
	    ){  
	        return ResponseEntity.ok(service.register(request));  
	    }  
	  
	    @PostMapping("/authenticate")  
	    public ResponseEntity<AuthenticationResponse> register(  
	            @RequestBody AuthenticationRequest request  
	    ){  
	        return ResponseEntity.ok(service.authenticate(request));  
	    }  
	  
	}

```AuthenticationService```의 ```register, authenticate``` 메소드를 사용해서 ```RegisterRequest```, ```AuthenticationRequest ``` 객체를 받는다는 것을 볼수 있다.

### AuthenticationService
	@Service  
	@RequiredArgsConstructor  
	public class AuthenticationService {  
	  
	    private final MemberRepository repository;  
	    private final PasswordEncoder passwordEncoder;  
	    private final JwtService jwtService;  
	    private final AuthenticationManager authenticationManager;  
	    public AuthenticationResponse register(RegisterRequest request) {  
	        var user = Member.builder()  
	                .name(request.getName())  
	                .account(request.getAccount())  
	                .password(passwordEncoder.encode(request.getPassword()))  
	                .createdAt(request.getCreatedAt())  
	                .role(Role.USER)  
	                .build();  
	        repository.save(user);  
	        var jwtToken = jwtService.generateToken(user);  
	  
	        return AuthenticationResponse.builder()  
	                .token(jwtToken)  
	                .build();  
	    }  
	  
	    public AuthenticationResponse authenticate(AuthenticationRequest request) {  
	        authenticationManager.authenticate(  
	                new UsernamePasswordAuthenticationToken(  
	                        request.getAccount(),  
	                        request.getPassword()  
	                )  
	        );  
	        var user = repository.findByAccount(request.getAccount())  
	                .orElseThrow();  
	        var jwtToken = jwtService.generateToken(user);  
	  
	        return AuthenticationResponse.builder()  
	                .token(jwtToken)  
	                .build();  
	    }  
	}

```Member.class``` 는 ```@Data``` 어노테이션으로 빌더가 사용 가능하다. 요청 객체에서 가져온 데이터로 repository에 저장한 후 ```jwtService```의 ```generateToken``` 메소드를 사용함을 알 수 있다.

	@Service  
	public class JwtService {  
	  
	    private static final String SECRET_KEY="여기에 키를 쓰세요";  
	    private static final int EXPIRE_TIME = 1000*60*30; // 만료시간 [ms] 단위, 30분이다.  
	  public String extractUsername(String token) {  
	        return extractClaim(token, Claims::getSubject);  
	    }  
	  
	    //Claims를 토큰으로 매핑  
	  public <T> T extractClaim(String token, Function<Claims, T> claimsResolver){  
	        final Claims claims = extractAllClaims(token);  
	        return claimsResolver.apply(claims);  
	    }  
	  
	    public String generateToken(UserDetails userDetails){  
	        return generateToken(new HashMap<>(), userDetails);  
	    }  
	    // new HashMap<>() 자리에 추가적인 값을 넣을수 있다  
	  
	  public String generateToken(  
	            Map<String, Object> extraClaims,  
	            UserDetails userDetails  
	    ){  
	        return Jwts  
	                .builder()  
	                .setClaims(extraClaims)  
	                .setSubject(userDetails.getUsername())  
	                .setIssuedAt(new Date(System.currentTimeMillis()))  
	                .setExpiration(new Date(System.currentTimeMillis() + EXPIRE_TIME))  
	                .signWith(getSignInKey(), SignatureAlgorithm.HS256)  
	                .compact();  
	    }  
	  
	    public boolean isTokenVaild(String token, UserDetails userDetails){  
	        final String username = extractUsername(token);  
	        return (username.equals(userDetails.getUsername())) && !isTokenExpired(token);  
	    }  
	  
	    private boolean isTokenExpired(String token) {  
	        return extractExpriation(token).before(new Date());  
	    }  
	  
	    private Date extractExpriation(String token) {  
	        return extractClaim(token, Claims::getExpiration);  
	    }  
	  
	    private Claims extractAllClaims(String token){  
	        return Jwts  
	                .parserBuilder()  
	                .setSigningKey(getSignInKey())  
	                .build()  
	                .parseClaimsJws(token)  
	                .getBody();  
	    }  
	  
	    private Key getSignInKey() {  
	        byte[] keyBytes = Decoders.BASE64.decode(SECRET_KEY);  
	        return Keys.hmacShaKeyFor(keyBytes);  
	    }  
	  
	}

**Secret Key**가 *너무 짧거나 추론되기 쉬운 문자로 하면 해킹에 매우 취약*하니 [Random hex generator](https://codebeautify.org/generate-random-hexadecimal-numbers) 를 이용하는 것이 좋다.

```Claims```는 jjwt 라이브러리에서 JSON의 BODY 부분으로 Map과 동일하다고 보면 된다.

```generateToken``` 이 두 가지가 존재하는데, 외부에서 쓰는 것이 위이고 new HashMap<>() 에 새로운 Map을 통해  추가적인 조치를 할 수 있다.

```HS256``` 알고리즘과 만료시간을 설정하고 JWT를 만들어준다.
