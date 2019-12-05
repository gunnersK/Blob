https://github.com/wuyouzhuguli/SpringAll

# 一、结构

   1. config：定义配置，如跳转登录页等
   
      1. 继承WebSecurityConfigurerAdapter抽象类，重写config方法，在config方法中用链式写法
      
      2. 代码
      
         ```
            @Override
            protected void configure(HttpSecurity http) throws Exception {
                    http.formLogin()
                        .loginPage("/authentication/require")
                        .loginProcessingUrl("/login")
                        .successHandler(authenticationSucessHandler)
                        .failureHandler(authenticationFailureHandler)
                        .and()

                        .authorizeRequests()
                        .antMatchers("/authentication/require", "/login.html").permitAll()
                        .anyRequest()
                        .authenticated()
                        .and().csrf().disable();

                    http.exceptionHandling()
                            .accessDeniedHandler(authenticationAccessDeniedHandler);
            }
         ```
   
   2. service：自定义认证规则、授权等
   
      1. 实现UserDetailsService接口，重写loadUserByUsername方法，自定义认证规则、授权信息
      
      2. 代码
      
         ```
            @Override
            public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
                // 模拟一个用户，替代数据库获取逻辑
                MyUser user = new MyUser();
                user.setUserName(username);
                user.setPassword(this.passwordEncoder.encode("123456"));
              //        user.setPassword("123456");
                // 输出加密后的密码
                System.out.println(user.getPassword());

                List<GrantedAuthority> authorities = new ArrayList<>();
                if (StringUtils.equalsIgnoreCase("user", username)) {
                    authorities = AuthorityUtils.commaSeparatedStringToAuthorityList("admin");
                } else {
                    authorities = AuthorityUtils.commaSeparatedStringToAuthorityList("test");
                }

                return new User(username, user.getPassword(), user.isEnabled(),
                        user.isAccountNonExpired(), user.isCredentialsNonExpired(),
                        user.isAccountNonLocked(), authorities);
            }
         ```
   
   3. handler：一些情况发生时的处理流程，例如登录成功or失败，权限不足等
   
      1. 例如自定义登录成功之后做的事情，实现AuthenticationSuccessHandler接口，重写onAuthenticationSuccess方法
      
      2. 代码
      
         ```
            @Override
            public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
           //        response.setContentType("application/json;charset=utf-8");
           //        response.getWriter().write(mapper.writeValueAsString(authentication));

                SavedRequest savedRequest = requestCache.getRequest(request, response);
           //        redirectStrategy.sendRedirect(request, response, savedRequest.getRedirectUrl());
           //        redirectStrategy.sendRedirect(request, response, "/a.html");
           //        redirectStrategy.sendRedirect(request, response, "/index");
                redirectStrategy.sendRedirect(request, response, "/auth/admin");
            }
         ```
   
   4. controller：处理跳转的请求
   
      1. 代码
      
         ```
            @RestController
            public class TestController {

                @GetMapping("/hello")
                public String hello() {
                    return "hello spring security";
                }

                @GetMapping("/index")
                public Object index(Authentication authentication) {
                //        return SecurityContextHolder.getContext().getAuthentication();
                    return authentication;
                }

                @GetMapping("/auth/admin")
                @PreAuthorize("hasAuthority('admin')")
                public String authenticationTest() {
                    return "您拥有admin权限，可以查看";
                }
            }
         ```
         
# 二、认证

   1. 在service中注入PasswordEncoder对象（先在config中配置）用于密码加密，然后再重写的loadUserByUsername方法中返回一个User，User中自定义认证规则
   
   2. 处理成功和失败的逻辑：分别实现AuthenticationSuccessHandler的onAuthenticationSuccess方法和AuthenticationFailureHandler的onAuthenticationFailure方法，在里面自定义逻辑

# 三、授权

   1. 在config中打上@EnableGlobalMethodSecurity(prePostEnabled = true)注解
   
   2. 在service中写授权逻辑
   
   3. 在对应的controller上打上@PreAuthorize("hasAuthority('权限名')")注解
   
   4. 可以自定义权限不足处理器来处理权限不足时候的操作
   
      1. 实现MyAuthenticationAccessDeniedHandler接口
      
      2. 将MyAuthenticationAccessDeniedHandler实现类对象注入到config中，并添加配置
      
         ```
            http.exceptionHandling()
                    .accessDeniedHandler(authenticationAccessDeniedHandler);
         ```

# 四、session管理

   1. 在yml文件中定义超时时间，最低不少于60s，如果设置少于60s，有效期也还是60s

   2. 在config中添加定义session失效之后的配置
   
      ```
         http.formLogin()
            ....
            .and()
                       .sessionManagement()
                       .invalidSessionUrl("/session/invalid")
      ```
      
   3. 在对应controller上打上@ResponseStatus(HttpStatus.UNAUTHORIZED)注解
      
   
