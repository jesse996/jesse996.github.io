# Spring Security整合JWT教程


## 大体思路：

### 登入：

1.  POST 用户名密码到 \login
2.  请求到达 JwtAuthenticationFilter 中的 attemptAuthentication() 方法，获取 request 中的 POST 参数，包装成一个 UsernamePasswordAuthenticationToken 交付给 AuthenticationManager 的 authenticate() 方法进行鉴权。
3.  AuthenticationManager 会从 CachingUserDetailsService 中查找用户信息，并且判断账号密码是否正确。
4.  如果账号密码正确跳转到 JwtAuthenticationFilter 中的 successfulAuthentication() 方法，我们进行签名，生成 token 返回给用户。
5.  账号密码错误则跳转到 JwtAuthenticationFilter 中的 unsuccessfulAuthentication() 方法，我们返回错误信息让用户重新登入。

### 请求鉴权：

请求鉴权的主要思路是我们会从请求中的 Authorization 字段拿取 token，如果不存在此字段的用户，Spring Security 会默认会用 AnonymousAuthenticationToken() 包装它，即代表匿名用户。

1. 任意请求发起
2. 到达 JwtAuthorizationFilter 中的 doFilterInternal() 方法，进行鉴权。
3. 如果鉴权成功我们把生成的 Authentication 用 SecurityContextHolder.getContext().setAuthentication() 放入 Security，即代表鉴权完成。此处如何鉴权由我们自己代码编写，后序会详细说明。

## JWT 的组成

-   JWT token 的格式：`header.payload.signature`
-   header 中用于存放签名的生成算法

    ```bash
    {"alg": "HS512"}
    ```

-   payload 中用于存放用户名、token 的生成时间和过期时间

    ```bash
    {"sub":"admin","created":1489079981393,"exp":1489684781}
    ```

-   signature 为以 header 和 payload 生成的签名，一旦 header 和 payload 被篡改，验证将失败

    ```bash
    //secret 为加密算法的密钥
    String signature = HMACSHA512(base64UrlEncode(header) + "." +base64UrlEncode(payload),secret)
    ```

## JWT 实例

```
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhZG1pbiIsImNyZWF0ZWQiOjE1NTY3NzkxMjUzMDksImV4cCI6MTU1NzM4MzkyNX0.d-iki0193X0bBOETf2UN3r3PotNIEAV7mzIxxeI5IxFyzzkOZxS0PGfF_SK6wxCv2K8S0cZjMkv6b5bCqc0VBw

```

## JWT 实现认证和授权的原理

-   用户调用登录接口，登录成功后获取到 JWT 的 token；
-   之后用户每次调用接口都在 http 的 header 中添加一个叫 Authorization 的头，值为 JWT 的 token；
-   后台程序通过对 Authorization 头中信息的解码及数字签名校验来获取其中的用户信息，从而实现认证和授权。

## pom 依赖

```xml
<dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>nimbus-jose-jwt</artifactId>
    <version>9.1</version>
</dependency>
```

## 使用

接下来我们将介绍下 nimbus-jose-jwt 库的使用，主要使用对称加密（HMAC）和非对称加密（RSA）两种算法来生成和解析 JWT 令牌。

### 对称加密（HMAC）

-   创建 JwtTokenServiceImpl 作为 JWT 处理的业务类，添加根据 HMAC 算法生成和解析 JWT 令牌的方法，可以发现 nimbus-jose-jwt 库操作 JWT 的 API 非常易于理解；

```java
@Service
public class JwtTokenServiceImpl implements JwtTokenService {
    @Override
    public String generateTokenByHMAC(String payloadStr, String secret) throws JOSEException {
        //创建JWS头，设置签名算法和类型
        JWSHeader jwsHeader = new JWSHeader.Builder(JWSAlgorithm.HS256).
                type(JOSEObjectType.JWT)
                .build();
        //将负载信息封装到Payload中
        Payload payload = new Payload(payloadStr);
        //创建JWS对象
        JWSObject jwsObject = new JWSObject(jwsHeader, payload);
        //创建HMAC签名器
        JWSSigner jwsSigner = new MACSigner(secret);
        //签名
        jwsObject.sign(jwsSigner);
        return jwsObject.serialize();
    }

    @Override
    public PayloadDto verifyTokenByHMAC(String token, String secret) throws ParseException, JOSEException {
        //从token中解析JWS对象
        JWSObject jwsObject = JWSObject.parse(token);
        //创建HMAC验证器
        JWSVerifier jwsVerifier = new MACVerifier(secret);
        if (!jwsObject.verify(jwsVerifier)) {
            throw new JwtInvalidException("token签名不合法！");
        }
        String payload = jwsObject.getPayload().toString();
        PayloadDto payloadDto = JSONUtil.toBean(payload, PayloadDto.class);
        if (payloadDto.getExp() < new Date().getTime()) {
            throw new JwtExpiredException("token已过期！");
        }
        return payloadDto;
    }
}
```

-   创建 PayloadDto 实体类，用于封装 JWT 中存储的信息；

```java
@Data
@EqualsAndHashCode(callSuper = false)
@Builder
public class PayloadDto {
    @ApiModelProperty("主题")
    private String sub;
    @ApiModelProperty("签发时间")
    private Long iat;
    @ApiModelProperty("过期时间")
    private Long exp;
    @ApiModelProperty("JWT的ID")
    private String jti;
    @ApiModelProperty("用户名称")
    private String username;
    @ApiModelProperty("用户拥有的权限")
    private List<String> authorities;
}
```

-   在 JwtTokenServiceImpl 类中添加获取默认的 PayloadDto 的方法，JWT 过期时间设置为 60min；

```java
@Service
public class JwtTokenServiceImpl implements JwtTokenService {
    @Override
    public PayloadDto getDefaultPayloadDto() {
        Date now = new Date();
        Date exp = DateUtil.offsetSecond(now, 60*60);
        return PayloadDto.builder()
                .sub("macro")
                .iat(now.getTime())
                .exp(exp.getTime())
                .jti(UUID.randomUUID().toString())
                .username("macro")
                .authorities(CollUtil.toList("ADMIN"))
                .build();
    }
}
```

-   创建 JwtTokenController 类，添加根据 HMAC 算法生成和解析 JWT 令牌的接口，由于 HMAC 算法需要长度至少为 32 个字节的秘钥，所以我们使用 MD5 加密下；

```java
@Api(tags = "JwtTokenController", description = "JWT令牌管理")
@Controller
@RequestMapping("/token")
public class JwtTokenController {

    @Autowired
    private JwtTokenService jwtTokenService;

    @ApiOperation("使用对称加密（HMAC）算法生成token")
    @RequestMapping(value = "/hmac/generate", method = RequestMethod.GET)
    @ResponseBody
    public CommonResult generateTokenByHMAC() {
        try {
            PayloadDto payloadDto = jwtTokenService.getDefaultPayloadDto();
            String token = jwtTokenService.generateTokenByHMAC(JSONUtil.toJsonStr(payloadDto), SecureUtil.md5("test"));
            return CommonResult.success(token);
        } catch (JOSEException e) {
            e.printStackTrace();
        }
        return CommonResult.failed();
    }

    @ApiOperation("使用对称加密（HMAC）算法验证token")
    @RequestMapping(value = "/hmac/verify", method = RequestMethod.GET)
    @ResponseBody
    public CommonResult verifyTokenByHMAC(String token) {
        try {
            PayloadDto payloadDto  = jwtTokenService.verifyTokenByHMAC(token, SecureUtil.md5("test"));
            return CommonResult.success(payloadDto);
        } catch (ParseException | JOSEException e) {
            e.printStackTrace();
        }
        return CommonResult.failed();

    }
}
```

## 添加 SpringSecurity 的配置类

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled=true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UmsAdminService adminService;
    @Autowired
    private RestfulAccessDeniedHandler restfulAccessDeniedHandler;
    @Autowired
    private RestAuthenticationEntryPoint restAuthenticationEntryPoint;

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.csrf()// 由于使用的是JWT，我们这里不需要csrf
                .disable()
                .sessionManagement()// 基于token，所以不需要session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                .antMatchers(HttpMethod.GET, // 允许对于网站静态资源的无授权访问
                        "/",
                        "/*.html",
                        "/favicon.ico",
                        "/**/*.html",
                        "/**/*.css",
                        "/**/*.js",
                        "/swagger-resources/**",
                        "/v2/api-docs/**"
                )
                .permitAll()
                .antMatchers("/admin/login", "/admin/register")// 对登录注册要允许匿名访问
                .permitAll()
                .antMatchers(HttpMethod.OPTIONS)//跨域请求会先进行一次options请求
                .permitAll()
//                .antMatchers("/**")//测试时全部运行访问
//                .permitAll()
                .anyRequest()// 除上面外的所有请求全部需要鉴权认证
                .authenticated();
        // 禁用缓存
        httpSecurity.headers().cacheControl();
        // 添加JWT filter
        httpSecurity.addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        //添加自定义未授权和未登录结果返回
        httpSecurity.exceptionHandling()
                .accessDeniedHandler(restfulAccessDeniedHandler)
                .authenticationEntryPoint(restAuthenticationEntryPoint);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService())
                .passwordEncoder(passwordEncoder());
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        //获取登录用户信息
        //这里返回了一个实现了UserDetailsService接口的匿名类
        return username -> {
            UmsAdmin admin = adminService.getAdminByUsername(username);
            if (admin != null) {
                List<UmsPermission> permissionList = adminService.getPermissionList(admin.getId());
                return new AdminUserDetails(admin,permissionList);
            }
            throw new UsernameNotFoundException("用户名或密码错误");
        };
    }

    @Bean
    public JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter(){
        return new JwtAuthenticationTokenFilter();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

}
```

