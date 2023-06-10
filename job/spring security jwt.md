### 인증순서 (직접 해본결과
로그인을 진행한다면 
1. 로그인 아이디비번 확인하고 getJwtToken 메소드를 호출한다.
2. getToken은 
```java
private String getJwtToken(String email, String pwd) throws Exception {
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(email, pwd);
        // authenticate 메서드가 실행이 될 때 UserDetailsServiceImpl 에서 만들었던 loadUserByUsername 메서드가 실행됨
        Authentication authentication = authenticationManagerBuilder.getObject().authenticate(authenticationToken);
        SecurityContextHolder.getContext().setAuthentication(authentication);
        return tokenProvider.tokenGenerator(authentication);
    }
```
다음과 같다 UserDetailService이제 권한이랑 이런거를 넣어서 Authetication이 나오게 되는것 같고 이것을 통해 토큰을 발급한다
토큰 발급에는 권한을 UserDetailService에서 준 Authentication에 권한이 들어있고 여기서 빼서 권한을 진행하고 그리고 getPrincipal()하면 UserDetails를 구현한 기본설정이면 알아서 해준다 나는 기본설정을 가져갔다
그래서 User기본설정에서 이메일을 빼오고 이를 암호화 시킨다 그리고 토큰 생성자로 Subject에 넣었다 또 필요한 정보는 claim애 넣어도 된다 

```
클레임은 JWT에 추가적인 정보를 담을 수 있는 페이로드(payload)의 일부입니다. 클레임은 일반적으로 사용자에 대한 추가적인 정보를 포함하거나 토큰의 권한 등을 지정하는 데 사용됩니다.
JWT는 클레임(claim)들의 집합으로 이루어진 토큰입니다. "sub" 클레임은 JWT의 주제(subject)를 나타냅니다. 주로 토큰을 발급한 사용자 또는 서버의 식별자를 포함합니다. 유저 이메일을 넣었다 나는
```


```java
    public String tokenGenerator(Authentication authentication) throws Exception {
        List<String> authorities = authentication.getAuthorities().stream()
                .map(GrantedAuthority::toString).toList();

        // 리펙토링 가능
        String payloadAuthEnc = encrypt(((org.springframework.security.core.userdetails.User) authentication.getPrincipal()).getUsername());
        String payLoadAuth = encrypt(authorities.get(0));

        return Jwts.builder()
                .setSubject(payloadAuthEnc)
//                .claim("data", payloadEnc)
                .claim("auth", payLoadAuth)
                .signWith(privateKey, SignatureAlgorithm.RS256) //Jwt의 서명 알고리즘과 대칭키 또는 비밀키를 설정하는데 사용된다
                .compact();
    }
```
이를 발급했다.

그리고 토큰을 가지고 로그인할경우는 어떤식이냐면
```java
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String jwt = resolveToken(httpServletRequest);
        String requestURI = httpServletRequest.getRequestURI();

        if (StringUtils.hasText(jwt) && tokenProvider.validateToken(jwt)) {
            Authentication authentication = tokenProvider.getAuthentication(jwt);
            SecurityContextHolder.getContext().setAuthentication(authentication);
            log.info("Security Context에 '{}' 인증 정보를 저장했습니다, uri: {}", authentication.getName(), requestURI);
        } else {
            log.warn("유효한 JWT 토큰이 없습니다, uri: {}", requestURI);
        }

        filterChain.doFilter(servletRequest, servletResponse);
    }

    private String resolveToken(HttpServletRequest request) {
        String bearerToken = request.getHeader(AUTHORIZATION_HEADER);
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }

        return null;
    }
```


1. jwtFilter필터를 타게되고
2. 그러면은 바로 인증으로 들어가고 헤더에서 토큰을 꺼낸다
3. 토큰을 이제 검증에 들어간다
4. 토큰을 파싱하고 암호화된것을 다 열어서 정보를 찾는다
5. 그리고 이 정보를 시큐리티에 저장한다
6. 그러면 저장된 정보는 @Authenticalprincipal에서 뺄수있는것이다 아마이때도  authenticate 메서드가 실행이 될 때 UserDetailsServiceImpl 에서 만들었던 loadUserByUsername 메서드가 실행되기떄문에
7. 컨틀롤러단에서 시큐리티 컨텍스트에서 쉽게 가져올수있는것같다

```
SecurityContextHolder의 getContext() 메소드를 통해 SecurityContext 객체를 얻고 그 안의 getAuthentication() 메소드를 통해 Authentication (인증객체)를 얻게 될 것입니다. 
```

```java
public Authentication getAuthentication(String token) {
        Claims claims = Jwts
                .parserBuilder()
                .setSigningKey(publicKey)
                .build()
                .parseClaimsJws(token)
                .getBody();


        String authoritiesObjDecrypt = decrypt(String.valueOf(claims.getSubject()));
        String auth = decrypt(String.valueOf(claims.get("auth")));
        List<SimpleGrantedAuthority> grantedAuthorities = new ArrayList<>();
        grantedAuthorities.add(new SimpleGrantedAuthority(auth));

        org.springframework.security.core.userdetails.User principal = new org.springframework.security.core.userdetails.User(authoritiesObjDecrypt, "", grantedAuthorities);
        return new UsernamePasswordAuthenticationToken(principal, "", grantedAuthorities);
    }
```
