# 🔐 OAuth2 — Complete In-Depth Guide

> **"OAuth2 is the industry standard for authorization. 'Login with Google/GitHub' and API access control all use OAuth2."**

---

## 📑 Table of Contents

1. [What is OAuth2?](#1-what-is-oauth2)
2. [OAuth2 Roles](#2-oauth2-roles)
3. [OAuth2 Grant Types](#3-oauth2-grant-types)
4. [Authorization Code Flow (Most Common)](#4-authorization-code-flow-most-common)
5. [OAuth2 in Spring Boot (Client)](#5-oauth2-in-spring-boot-client)
6. [OAuth2 Resource Server](#6-oauth2-resource-server)
7. [OpenID Connect (OIDC)](#7-openid-connect-oidc)
8. [OAuth2 vs JWT vs Session](#8-oauth2-vs-jwt-vs-session)
9. [Security Considerations](#9-security-considerations)
10. [Interview Questions & Answers (50+)](#10-interview-questions--answers-50)

---

## 1. What is OAuth2?

OAuth2 is an **authorization framework** that allows third-party apps to access user data WITHOUT sharing passwords.

```
WITHOUT OAuth2:
  "Give me your Google password so I can read your contacts"  😱

WITH OAuth2:
  "Click 'Login with Google'. Google asks YOU for permission.
   I get a LIMITED token — never see your password."  ✅
```

---

## 2. OAuth2 Roles

```
┌────────────────┐          ┌────────────────────┐
│ Resource Owner  │          │ Authorization Server│
│ (User — You)    │          │ (Google, GitHub)    │
│                 │          │ Issues tokens       │
│ "Yes, I allow   │          │                    │
│  MyApp to read  │          │ /authorize         │
│  my profile"    │          │ /token             │
└───────┬────────┘          └─────────┬──────────┘
        │                             │
        │  Grants permission          │  Issues tokens
        │                             │
┌───────▼────────┐          ┌─────────▼──────────┐
│ Client          │          │ Resource Server     │
│ (MyApp)         │──token──►│ (Google API)        │
│                 │          │ Has the user's data │
│ Wants to access │◄─data────│                    │
│ user's data     │          │ /api/userinfo      │
└────────────────┘          └────────────────────┘
```

| Role | Who | Example |
|------|-----|---------|
| **Resource Owner** | The user | You (Dilip) |
| **Client** | App wanting access | MyApp (Spring Boot) |
| **Authorization Server** | Issues tokens | Google, GitHub, Keycloak |
| **Resource Server** | Hosts protected data | Google API, Your API |

---

## 3. OAuth2 Grant Types

| Grant Type | Use Case | How |
|------------|----------|-----|
| **Authorization Code** | Web apps with server | Most secure, uses code exchange ✅ |
| **Authorization Code + PKCE** | SPAs, mobile apps | Code + code verifier ✅ |
| **Client Credentials** | Machine-to-machine | No user, app authenticates itself |
| **Refresh Token** | Extend session | Exchange refresh token for new access token |
| ~~Implicit~~ | ~~SPAs (DEPRECATED)~~ | ~~Token in URL — insecure ❌~~ |
| ~~Password~~ | ~~Trusted apps (DEPRECATED)~~ | ~~Direct credentials — avoid ❌~~ |

---

## 4. Authorization Code Flow (Most Common)

```
Step 1: User clicks "Login with Google" on MyApp

Step 2: MyApp redirects user to Google:
  https://accounts.google.com/o/oauth2/v2/auth?
    response_type=code&
    client_id=myapp123&
    redirect_uri=http://myapp.com/callback&
    scope=openid profile email&
    state=random123

Step 3: User logs in to Google & grants permission
  Google: "MyApp wants to access your name and email. Allow?"
  User: "Yes!"

Step 4: Google redirects back to MyApp with an AUTH CODE:
  http://myapp.com/callback?code=AUTH_CODE_123&state=random123

Step 5: MyApp exchanges code for tokens (SERVER-SIDE, not visible to user):
  POST https://oauth2.googleapis.com/token
  Body: {
    grant_type=authorization_code,
    code=AUTH_CODE_123,
    client_id=myapp123,
    client_secret=SECRET,
    redirect_uri=http://myapp.com/callback
  }

Step 6: Google returns tokens:
  {
    "access_token": "ya29.abc...",
    "refresh_token": "1//xyz...",
    "id_token": "eyJhbG...",     // JWT with user info (OpenID Connect)
    "expires_in": 3600
  }

Step 7: MyApp uses access_token to get user info:
  GET https://www.googleapis.com/oauth2/v3/userinfo
  Authorization: Bearer ya29.abc...

Step 8: Google returns user profile:
  {"sub": "12345", "name": "Dilip", "email": "dilip@gmail.com"}
```

```
VISUAL FLOW:

User          MyApp            Google Auth         Google API
 │              │                  │                   │
 │  Click Login │                  │                   │
 │─────────────►│                  │                   │
 │              │  Redirect to     │                   │
 │◄─────────────│  Google login    │                   │
 │              │                  │                   │
 │  Login + Consent               │                   │
 │────────────────────────────────►│                   │
 │              │                  │                   │
 │  Redirect with code            │                   │
 │◄────────────────────────────────│                   │
 │              │                  │                   │
 │  Code        │                  │                   │
 │─────────────►│  Exchange code   │                   │
 │              │─────────────────►│                   │
 │              │  Access token    │                   │
 │              │◄─────────────────│                   │
 │              │                  │                   │
 │              │  GET /userinfo   │                   │
 │              │──────────────────────────────────────►│
 │              │  User profile    │                   │
 │              │◄──────────────────────────────────────│
 │  Welcome!    │                  │                   │
 │◄─────────────│                  │                   │
```

---

## 5. OAuth2 in Spring Boot (Client)

### Login with Google/GitHub

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope: openid, profile, email
          github:
            client-id: ${GITHUB_CLIENT_ID}
            client-secret: ${GITHUB_CLIENT_SECRET}
            scope: user:email, read:user
```

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login", "/css/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard", true)
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(customOAuth2UserService)
                )
            );
        
        return http.build();
    }
}

// Custom OAuth2 user service — load/create user in YOUR database
@Service
public class CustomOAuth2UserService extends DefaultOAuth2UserService {
    
    private final UserRepository userRepository;
    
    @Override
    public OAuth2User loadUser(OAuth2UserRequest request) throws OAuth2AuthenticationException {
        OAuth2User oauth2User = super.loadUser(request);
        
        String email = oauth2User.getAttribute("email");
        String name = oauth2User.getAttribute("name");
        String provider = request.getClientRegistration().getRegistrationId(); // "google"
        
        // Find or create user in YOUR database
        User user = userRepository.findByEmail(email)
            .orElseGet(() -> {
                User newUser = new User();
                newUser.setName(name);
                newUser.setEmail(email);
                newUser.setProvider(provider);
                newUser.setRole(UserRole.USER);
                return userRepository.save(newUser);
            });
        
        return oauth2User;
    }
}
```

---

## 6. OAuth2 Resource Server

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://accounts.google.com
          # Or: jwk-set-uri: https://www.googleapis.com/oauth2/v3/certs
```

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/public/**").permitAll()
            .anyRequest().authenticated()
        )
        .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
    
    return http.build();
}
```

---

## 7. OpenID Connect (OIDC)

```
OAuth2 = Authorization ("can this app access my data?")
OIDC   = Authentication ("who is this user?")

OIDC adds:
  1. ID Token (JWT with user identity)
  2. /userinfo endpoint
  3. Standard claims (name, email, picture)
  4. Discovery endpoint (/.well-known/openid-configuration)

OAuth2 alone: "App X can read your photos" (access_token)
OIDC:         "You are Dilip, dilip@gmail.com" (id_token)
```

---

## 8. OAuth2 vs JWT vs Session

| Feature | OAuth2 | JWT | Session |
|---------|--------|-----|---------|
| Type | Framework | Token format | Server-side storage |
| Stateless | Depends | Yes | No |
| Third-party login | Yes ✅ | No | No |
| Token format | Any (often JWT) | JWT | Cookie (JSESSIONID) |
| Use case | "Login with Google" | API auth | Traditional web apps |

---

## 9. Security Considerations

```
1. Always use HTTPS — tokens are sent over the wire
2. Validate state parameter — prevents CSRF in OAuth2 flow
3. Use PKCE for public clients — SPAs, mobile apps
4. Validate tokens properly — check signature, expiry, issuer, audience
5. Use short-lived access tokens — minimize damage if stolen
6. Store client secret securely — never in frontend code
7. Restrict scopes — request minimum permissions needed
8. Validate redirect URIs — prevent open redirect attacks
```

---

## 10. Interview Questions & Answers (50+)

### Beginner

**Q1: What is OAuth2?** Authorization framework allowing third-party apps to access user data without sharing passwords.

**Q2: What is the difference between authentication and authorization?** Authentication: verify identity. Authorization: verify permissions. OAuth2 is primarily authorization.

**Q3: What is an access token?** Token granting limited access to a resource. Short-lived. Sent with API requests.

**Q4: What is a refresh token?** Long-lived token used to get new access tokens without re-login.

**Q5: What is scope in OAuth2?** Permissions the client is requesting: `email`, `profile`, `read:user`.

**Q6: What is the Authorization Code flow?** Most secure flow. User gets auth code → app exchanges code for tokens server-side.

**Q7: What is a client_id?** Unique identifier for your application registered with the OAuth2 provider.

**Q8: What is client_secret?** Secret key known only to your app and the auth server. Never expose to frontend.

---

### Intermediate

**Q9: What is PKCE?** Proof Key for Code Exchange. Adds code_verifier/code_challenge to prevent code interception. Required for SPAs/mobile.

**Q10: What is OpenID Connect?** Identity layer on top of OAuth2. Adds id_token (JWT with user identity), /userinfo endpoint.

**Q11: What is the state parameter?** Random value to prevent CSRF. Client sends it, server returns it unchanged.

**Q12: Implicit flow vs Authorization Code flow?** Implicit: token in URL (deprecated, insecure). Authorization Code: code exchanged server-side (secure ✅).

**Q13: What is a Resource Server?** Server hosting protected resources. Validates access tokens. Your Spring Boot API.

**Q14: What is an Authorization Server?** Server that issues tokens. Google, GitHub, Keycloak, Auth0.

**Q15: Client Credentials flow use case?** Machine-to-machine. No user involved. App authenticates with client_id + client_secret.

---

### Advanced

**Q16: How does token introspection work?** Resource server sends token to auth server's `/introspect` endpoint to check validity. Used for opaque tokens.

**Q17: What is JWKS?** JSON Web Key Set — public key endpoint. Resource server fetches keys to verify JWT signatures.

**Q18: How to implement custom OAuth2 Authorization Server?** Use Spring Authorization Server project. Configure client registrations, token settings, user authentication.

**Q19: What is token exchange?** Exchanging one token for another with different scope/audience. Used in microservices.

**Q20: What is device authorization grant?** For devices with limited input (smart TVs). User authorizes on another device.

---

### Rapid-Fire (Q21–Q50)

**Q21: OAuth2 vs OAuth1?** OAuth2 is simpler, uses bearer tokens, HTTPS required. OAuth1 uses signatures.

**Q22: What is redirect_uri?** URL where auth server sends the user back after authorization.

**Q23: What is grant_type?** Type of OAuth2 flow: authorization_code, client_credentials, refresh_token.

**Q24: What is id_token?** JWT containing user identity claims (OIDC). Name, email, picture.

**Q25: What is `response_type=code`?** Requests authorization code (Authorization Code flow).

**Q26: What is nonce in OIDC?** Random value to prevent replay attacks on id_token.

**Q27: What is token revocation?** Invalidating a token: POST /revoke with the token.

**Q28: What is `.well-known/openid-configuration`?** Discovery endpoint listing all OIDC endpoints and capabilities.

**Q29: What is `offline_access` scope?** Requests a refresh token for accessing resources when user is offline.

**Q30: What is consent screen?** UI where user approves/denies requested permissions.

**Q31: What is Keycloak?** Open-source Identity and Access Management server supporting OAuth2/OIDC.

**Q32: What is Auth0?** Cloud-based authentication platform. OAuth2/OIDC provider.

**Q33: What is `spring-boot-starter-oauth2-client`?** Spring Boot starter for OAuth2 login (Social login).

**Q34: What is `spring-boot-starter-oauth2-resource-server`?** Starter for validating OAuth2 tokens in your API.

**Q35: Single Sign-On (SSO)?** Login once, access multiple apps. Achieved with shared auth server (OAuth2/OIDC).

**Q36: What is bearer token?** Token sent in Authorization header. Whoever "bears" the token gets access.

**Q37: Can OAuth2 use JWTs?** Yes! Access tokens and id_tokens are often JWTs.

**Q38: What is opaque token in OAuth2?** Random string token. Must be validated by calling auth server.

**Q39: OAuth2 for microservices?** API Gateway validates token. Services trust the gateway or validate independently.

**Q40: What is OAuth2 client registration?** Register your app with the provider to get client_id and client_secret.

**Q41: What is `issuer-uri`?** URL of the auth server. Spring uses it to auto-discover endpoints.

**Q42: What is `jwk-set-uri`?** URL to fetch public keys for JWT verification.

**Q43: What is the `sub` claim in id_token?** Unique identifier for the user at the provider.

**Q44: What is fine-grained authorization?** Controlling access to specific resources/actions beyond roles.

**Q45: How to test OAuth2 endpoints?** Get token from auth server, send in Authorization header. Or mock in tests.

**Q46: What is OAuth2AuthorizedClient?** Spring Security object holding the access token for an authenticated OAuth2 client.

**Q47: What is OAuth2AuthorizedClientService?** Service to save/load OAuth2AuthorizedClient (tokens).

**Q48: What is Spring Authorization Server?** Spring project for building your own OAuth2/OIDC authorization server.

**Q49: What is token binding?** Binding token to specific client (TLS, device). Prevents token theft.

**Q50: OAuth2 vs SAML?** OAuth2: modern, JSON/JWT, mobile-friendly. SAML: enterprise/legacy, XML, browser-based.

---

## 📚 References

- [OAuth2 RFC 6749](https://tools.ietf.org/html/rfc6749)
- [Spring Security OAuth2](https://docs.spring.io/spring-security/reference/servlet/oauth2/)
- [OAuth2.0 Simplified](https://www.oauth.com/)
- [OpenID Connect Spec](https://openid.net/connect/)

---

> **Previous Topic:** [← 18 - JWT](../18-jwt/README.md)  
> **Next Topic:** [20 - Logging & Log4j →](../20-logging-log4j/README.md)
