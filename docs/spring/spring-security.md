# Spring Security

## Spring Security Web 端架构

![](../images/security-filter-chain.png)

![](../images/multi-securityfilterchain.png)

Spring Security Filter 顺序:

- ChannelProcessingFilter

- WebAsyncManagerIntegrationFilter

- SecurityContextPersistenceFilter

- HeaderWriterFilter

- CorsFilter

- CsrfFilter

- LogoutFilter

- OAuth2AuthorizationRequestRedirectFilter

- Saml2WebSsoAuthenticationRequestFilter

- X509AuthenticationFilter

- AbstractPreAuthenticatedProcessingFilter

- CasAuthenticationFilter

- OAuth2LoginAuthenticationFilter

- Saml2WebSsoAuthenticationFilter

- UsernamePasswordAuthenticationFilter

- OpenIDAuthenticationFilter

- DefaultLoginPageGeneratingFilter

- DefaultLogoutPageGeneratingFilter

- ConcurrentSessionFilter

- DigestAuthenticationFilter

- BearerTokenAuthenticationFilter

- BasicAuthenticationFilter

- RequestCacheAwareFilter

- SecurityContextHolderAwareRequestFilter

- JaasApiIntegrationFilter

- RememberMeAuthenticationFilter

- AnonymousAuthenticationFilter

- OAuth2AuthorizationCodeGrantFilter

- SessionManagementFilter

- ExceptionTranslationFilter

- FilterSecurityInterceptor

- SwitchUserFilter

认证和权限相关异常处理:

![](../images/exceptiontranslationfilter.png)


## References

- [Security with Spring](https://www.baeldung.com/security-spring)
- [Spring Security 官方文档](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
