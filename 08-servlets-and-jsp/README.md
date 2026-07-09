# 🌐 Servlets and JSP — Complete In-Depth Guide (Beginner to Expert)

> **"Servlets and JSP are the foundation of Java web development. Spring Boot builds on top of Servlets — understanding them is essential for debugging and mastering the framework."**

---

## 📑 Table of Contents

1. [Introduction & Overview](#1-introduction--overview)
2. [Servlet Architecture](#2-servlet-architecture)
3. [Servlet Lifecycle](#3-servlet-lifecycle)
4. [HTTP Servlets](#4-http-servlets)
5. [HttpServletRequest & HttpServletResponse](#5-httpservletrequest--httpservletresponse)
6. [Servlet Configuration](#6-servlet-configuration)
7. [Request Dispatching](#7-request-dispatching)
8. [Session Management](#8-session-management)
9. [Filters & Listeners](#9-filters--listeners)
10. [JSP (JavaServer Pages)](#10-jsp-javaserver-pages)
11. [JSP Implicit Objects](#11-jsp-implicit-objects)
12. [JSTL & Expression Language](#12-jstl--expression-language)
13. [MVC Pattern with Servlets & JSP](#13-mvc-pattern-with-servlets--jsp)
14. [Servlets in Spring Boot](#14-servlets-in-spring-boot)
15. [Best Practices](#15-best-practices)
16. [Common Mistakes & Pitfalls](#16-common-mistakes--pitfalls)
17. [Interview Questions & Answers (50+)](#17-interview-questions--answers-50)

---

## 1. Introduction & Overview

### What are Servlets?

A **Servlet** is a Java class that handles HTTP requests and generates HTTP responses on a web server. It runs inside a **Servlet Container** (like Apache Tomcat), which manages its lifecycle.

```
Client (Browser)          Server (Tomcat)
┌──────────┐    HTTP    ┌─────────────────────┐
│          │ Request    │  Servlet Container   │
│ Browser  │──────────►│                     │
│          │           │  ┌───────────────┐  │
│          │◄──────────│  │   Servlet     │  │
│          │ Response   │  │ (Java Class)  │  │
└──────────┘           │  └───────────────┘  │
                       └─────────────────────┘
```

### Why Learn Servlets?

- **Foundation** — Spring MVC's DispatcherServlet IS a Servlet
- **Spring Boot** — Runs on an embedded Tomcat servlet container
- **Filters** — Spring Security's filter chain uses Servlet Filters
- **Understanding** — Knowing what happens "under the hood" makes debugging easier
- **Interviews** — Commonly asked in Java backend interviews

### Servlet vs Spring MVC

| Feature | Raw Servlet | Spring MVC |
|---------|------------|------------|
| Configuration | web.xml or annotations | Auto-configured |
| URL Mapping | `@WebServlet` | `@RequestMapping` |
| Request handling | Override `doGet`/`doPost` | Methods with `@GetMapping` |
| View technology | JSP, manual HTML | Thymeleaf, JSP, etc. |
| Dependency injection | Manual | Automatic (`@Autowired`) |
| Boilerplate | Lots | Minimal |

---

## 2. Servlet Architecture

### Servlet Container (Tomcat)

```
┌──────────────────────────────────────────┐
│            Servlet Container              │
│                                           │
│  ┌─────────────────────────────────────┐ │
│  │           Web Application           │ │
│  │                                     │ │
│  │  ┌──────────┐  ┌──────────┐        │ │
│  │  │ Servlet1 │  │ Servlet2 │  ...   │ │
│  │  └──────────┘  └──────────┘        │ │
│  │                                     │ │
│  │  ┌──────────┐  ┌──────────┐        │ │
│  │  │ Filter1  │  │ Filter2  │  ...   │ │
│  │  └──────────┘  └──────────┘        │ │
│  │                                     │ │
│  │  ┌──────────┐  ┌──────────┐        │ │
│  │  │Listener1 │  │Listener2 │  ...   │ │
│  │  └──────────┘  └──────────┘        │ │
│  └─────────────────────────────────────┘ │
│                                           │
│  Thread Pool (handles concurrent requests)│
└──────────────────────────────────────────┘
```

### Request Processing Flow

```
1. Client sends HTTP request
2. Servlet Container receives request
3. Container finds the matching Servlet (by URL pattern)
4. Container creates/reuses HttpServletRequest and HttpServletResponse objects
5. Container calls the Servlet's service() method
6. service() dispatches to doGet(), doPost(), etc.
7. Servlet processes request, generates response
8. Container sends response back to client
```

---

## 3. Servlet Lifecycle

```
┌──────────────┐
│   Loading     │ ← Class is loaded when first request arrives
│   (ClassLoader)│   (or at startup if loadOnStartup > 0)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Instantiation │ ← Container creates ONE instance of the Servlet
│ (Constructor) │   (Servlets are SINGLETONS!)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  init()      │ ← Called ONCE after instantiation
│              │   Initialize resources (DB connections, config)
└──────┬───────┘
       │
       ▼
┌──────────────┐
│  service()   │ ← Called for EVERY request (in a new thread)
│  doGet()     │   Handles the actual request processing
│  doPost()    │   Multiple threads can call this simultaneously!
│  doPut()     │
│  doDelete()  │
└──────┬───────┘
       │ (when container shuts down or app is undeployed)
       ▼
┌──────────────┐
│  destroy()   │ ← Called ONCE before the Servlet is removed
│              │   Release resources (close DB connections)
└──────────────┘
```

```java
import jakarta.servlet.*;
import jakarta.servlet.http.*;
import jakarta.servlet.annotation.WebServlet;
import java.io.*;

@WebServlet("/lifecycle")
public class LifecycleServlet extends HttpServlet {
    
    // Called ONCE when servlet is first loaded
    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        System.out.println("init() called — Servlet is born! 🎉");
        // Initialize resources: DB connection pool, load config, etc.
    }
    
    // Called for EVERY GET request (each in its own thread)
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        System.out.println("doGet() called — Thread: " + Thread.currentThread().getName());
        response.setContentType("text/html");
        response.getWriter().println("<h1>Hello from Servlet!</h1>");
    }
    
    // Called ONCE when servlet is destroyed (server shutdown)
    @Override
    public void destroy() {
        System.out.println("destroy() called — Servlet is dying! 💀");
        // Release resources: close DB connections, flush caches, etc.
    }
}
```

---

## 4. HTTP Servlets

### Handling Different HTTP Methods

```java
@WebServlet("/users")
public class UserServlet extends HttpServlet {
    
    // Handle GET request (read/fetch data)
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        
        String userId = req.getParameter("id");  // Get query parameter: /users?id=1
        
        resp.setContentType("application/json");
        resp.setCharacterEncoding("UTF-8");
        
        PrintWriter out = resp.getWriter();
        out.print("{\"id\": \"" + userId + "\", \"name\": \"Dilip\"}");
        out.flush();
    }
    
    // Handle POST request (create data)
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        
        // Read request body
        String name = req.getParameter("name");
        String email = req.getParameter("email");
        
        // Process (save to database, etc.)
        
        resp.setStatus(HttpServletResponse.SC_CREATED);  // 201
        resp.getWriter().println("User created: " + name);
    }
    
    // Handle PUT request (update data)
    @Override
    protected void doPut(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        
        // Read JSON body using BufferedReader
        StringBuilder body = new StringBuilder();
        try (BufferedReader reader = req.getReader()) {
            String line;
            while ((line = reader.readLine()) != null) {
                body.append(line);
            }
        }
        
        resp.getWriter().println("Updated with: " + body);
    }
    
    // Handle DELETE request (delete data)
    @Override
    protected void doDelete(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        
        String userId = req.getParameter("id");
        resp.getWriter().println("Deleted user: " + userId);
    }
}
```

---

## 5. HttpServletRequest & HttpServletResponse

### Request Object

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
    
    // ═══ Request Line Info ═══
    String method = req.getMethod();           // GET, POST, PUT, DELETE
    String uri = req.getRequestURI();          // /app/users
    String url = req.getRequestURL().toString(); // http://localhost:8080/app/users
    String queryString = req.getQueryString();  // id=1&name=Dilip
    String protocol = req.getProtocol();        // HTTP/1.1
    
    // ═══ Parameters ═══
    String id = req.getParameter("id");                    // Single value
    String[] hobbies = req.getParameterValues("hobby");    // Multiple values
    Map<String, String[]> allParams = req.getParameterMap(); // All params
    
    // ═══ Headers ═══
    String contentType = req.getHeader("Content-Type");
    String auth = req.getHeader("Authorization");
    String userAgent = req.getHeader("User-Agent");
    Enumeration<String> headerNames = req.getHeaderNames();  // All header names
    
    // ═══ Client Info ═══
    String clientIP = req.getRemoteAddr();     // Client's IP address
    String clientHost = req.getRemoteHost();   // Client's hostname
    int clientPort = req.getRemotePort();      // Client's port
    
    // ═══ Server Info ═══
    String serverName = req.getServerName();   // localhost
    int serverPort = req.getServerPort();      // 8080
    String contextPath = req.getContextPath(); // /app
    String servletPath = req.getServletPath(); // /users
    
    // ═══ Attributes (request scope — for forwarding) ═══
    req.setAttribute("user", userObject);      // Set attribute
    Object user = req.getAttribute("user");    // Get attribute
    
    // ═══ Session ═══
    HttpSession session = req.getSession();    // Get/create session
    HttpSession session = req.getSession(false); // Get existing (null if none)
}
```

### Response Object

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
    
    // ═══ Status Code ═══
    resp.setStatus(HttpServletResponse.SC_OK);           // 200
    resp.setStatus(HttpServletResponse.SC_CREATED);      // 201
    resp.setStatus(HttpServletResponse.SC_NOT_FOUND);    // 404
    resp.sendError(HttpServletResponse.SC_FORBIDDEN, "Access denied");  // 403
    
    // ═══ Headers ═══
    resp.setContentType("text/html");                    // Content type
    resp.setContentType("application/json");
    resp.setCharacterEncoding("UTF-8");
    resp.setHeader("Cache-Control", "no-cache");
    resp.addHeader("X-Custom-Header", "value");
    resp.setIntHeader("Refresh", 30);                    // Refresh every 30s
    
    // ═══ Redirect ═══
    resp.sendRedirect("/login");                         // 302 redirect
    resp.sendRedirect("https://google.com");
    
    // ═══ Writing Response Body ═══
    // Text/HTML:
    PrintWriter out = resp.getWriter();
    out.println("<html><body>Hello!</body></html>");
    
    // Binary (images, files):
    OutputStream os = resp.getOutputStream();
    os.write(fileBytes);
    
    // ═══ Cookies ═══
    Cookie cookie = new Cookie("username", "Dilip");
    cookie.setMaxAge(3600);        // 1 hour
    cookie.setHttpOnly(true);      // Not accessible via JavaScript
    cookie.setSecure(true);        // Only send over HTTPS
    resp.addCookie(cookie);
}
```

---

## 6. Servlet Configuration

### Annotation-Based (Modern — Preferred)

```java
@WebServlet(
    name = "UserServlet",
    urlPatterns = {"/users", "/users/*"},
    loadOnStartup = 1,  // Load immediately (not on first request)
    initParams = {
        @WebInitParam(name = "maxPageSize", value = "100"),
        @WebInitParam(name = "defaultSort", value = "name")
    }
)
public class UserServlet extends HttpServlet {
    
    private int maxPageSize;
    
    @Override
    public void init() throws ServletException {
        maxPageSize = Integer.parseInt(getInitParameter("maxPageSize"));
    }
}
```

### XML-Based (Legacy — web.xml)

```xml
<!-- WEB-INF/web.xml -->
<web-app>
    <servlet>
        <servlet-name>UserServlet</servlet-name>
        <servlet-class>com.example.UserServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
        <init-param>
            <param-name>maxPageSize</param-name>
            <param-value>100</param-value>
        </init-param>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>UserServlet</servlet-name>
        <url-pattern>/users</url-pattern>
    </servlet-mapping>
</web-app>
```

---

## 7. Request Dispatching

### Forward vs Redirect

```
FORWARD (Server-side, client doesn't know):
  Client → Request → Servlet A → Forward → Servlet B → Response → Client
  URL in browser stays the same!
  Same request and response objects.

REDIRECT (Client-side, makes a NEW request):
  Client → Request → Servlet A → 302 Response → Client → NEW Request → Servlet B
  URL in browser CHANGES!
  New request and response objects.
```

```java
// FORWARD — Pass request to another servlet/JSP (server-side)
RequestDispatcher dispatcher = req.getRequestDispatcher("/WEB-INF/views/user.jsp");
req.setAttribute("user", user);  // Pass data to JSP
dispatcher.forward(req, resp);    // Forward (URL doesn't change in browser)

// REDIRECT — Tell browser to make a new request (client-side)
resp.sendRedirect("/login");      // URL changes in browser, new request
resp.sendRedirect(req.getContextPath() + "/dashboard");  // With context path

// INCLUDE — Include output of another servlet in current response
RequestDispatcher header = req.getRequestDispatcher("/header.jsp");
header.include(req, resp);        // Include header output
```

---

## 8. Session Management

### HttpSession

```java
@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        
        if (authenticate(username, password)) {
            // Create session
            HttpSession session = req.getSession();  // Creates new or gets existing
            session.setAttribute("user", username);   // Store data in session
            session.setAttribute("role", "admin");
            session.setMaxInactiveInterval(1800);     // 30 minutes timeout
            
            resp.sendRedirect("/dashboard");
        } else {
            resp.sendRedirect("/login?error=true");
        }
    }
}

@WebServlet("/dashboard")
public class DashboardServlet extends HttpServlet {
    
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        HttpSession session = req.getSession(false);  // Don't create if doesn't exist
        
        if (session == null || session.getAttribute("user") == null) {
            resp.sendRedirect("/login");  // Not logged in!
            return;
        }
        
        String username = (String) session.getAttribute("user");
        resp.getWriter().println("Welcome, " + username + "!");
    }
}

@WebServlet("/logout")
public class LogoutServlet extends HttpServlet {
    
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        HttpSession session = req.getSession(false);
        if (session != null) {
            session.invalidate();  // Destroy the session
        }
        resp.sendRedirect("/login");
    }
}
```

### Session Tracking Mechanisms

| Method | How It Works | Pros | Cons |
|--------|-------------|------|------|
| Cookies | JSESSIONID cookie | Automatic, standard | Client can disable |
| URL Rewriting | Session ID in URL | Works without cookies | Ugly URLs, security risk |
| Hidden Fields | Session ID in form fields | Works in forms | Only for forms |

---

## 9. Filters & Listeners

### Filters — Intercept Requests/Responses

```java
@WebFilter("/*")  // Apply to ALL URLs
public class LoggingFilter implements Filter {
    
    @Override
    public void init(FilterConfig config) throws ServletException {
        System.out.println("LoggingFilter initialized");
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest req = (HttpServletRequest) request;
        
        long startTime = System.currentTimeMillis();
        System.out.println("Request: " + req.getMethod() + " " + req.getRequestURI());
        
        // BEFORE the servlet processes the request
        
        chain.doFilter(request, response);  // Pass to next filter or servlet
        
        // AFTER the servlet processes the request
        
        long duration = System.currentTimeMillis() - startTime;
        System.out.println("Response time: " + duration + "ms");
    }
    
    @Override
    public void destroy() {
        System.out.println("LoggingFilter destroyed");
    }
}
```

```
Filter Chain:
  Request → Filter1 → Filter2 → Filter3 → Servlet
  Response ← Filter1 ← Filter2 ← Filter3 ← Servlet

Spring Security uses this EXACT mechanism!
Its filter chain includes:
  SecurityContextFilter → CsrfFilter → AuthenticationFilter → AuthorizationFilter → ...
```

### Authentication Filter Example

```java
@WebFilter("/api/*")
public class AuthFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response,
                         FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse resp = (HttpServletResponse) response;
        
        HttpSession session = req.getSession(false);
        
        if (session == null || session.getAttribute("user") == null) {
            resp.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Please login first");
            return;  // DON'T call chain.doFilter — stop here!
        }
        
        chain.doFilter(request, response);  // User is authenticated, continue
    }
}
```

### Listeners — React to Events

```java
// Session Listener — track active sessions
@WebListener
public class SessionListener implements HttpSessionListener {
    
    private static final AtomicInteger activeSessionCount = new AtomicInteger(0);
    
    @Override
    public void sessionCreated(HttpSessionEvent se) {
        int count = activeSessionCount.incrementAndGet();
        System.out.println("Session created. Active sessions: " + count);
    }
    
    @Override
    public void sessionDestroyed(HttpSessionEvent se) {
        int count = activeSessionCount.decrementAndGet();
        System.out.println("Session destroyed. Active sessions: " + count);
    }
    
    public static int getActiveSessionCount() {
        return activeSessionCount.get();
    }
}

// Context Listener — application startup/shutdown
@WebListener
public class AppListener implements ServletContextListener {
    
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("Application started! 🚀");
        // Initialize connection pool, load config, etc.
    }
    
    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("Application shutting down! 🛑");
        // Close connections, cleanup resources
    }
}
```

---

## 10. JSP (JavaServer Pages)

### What is JSP?

JSP is an HTML file with embedded Java code. It gets compiled into a Servlet by the container.

```jsp
<%-- This is a JSP comment (not sent to client) --%>

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="java.util.List, com.example.User" %>

<!DOCTYPE html>
<html>
<head>
    <title>User List</title>
</head>
<body>
    <h1>Welcome, <%= request.getAttribute("username") %>!</h1>
    
    <%-- Scriptlet: Java code --%>
    <%
        List<User> users = (List<User>) request.getAttribute("users");
        if (users != null) {
    %>
        <table>
            <tr><th>Name</th><th>Email</th></tr>
            <% for (User user : users) { %>
                <tr>
                    <td><%= user.getName() %></td>
                    <td><%= user.getEmail() %></td>
                </tr>
            <% } %>
        </table>
    <% } else { %>
        <p>No users found.</p>
    <% } %>
</body>
</html>
```

### JSP Elements

| Element | Syntax | Purpose |
|---------|--------|---------|
| Directive | `<%@ page ... %>` | Page configuration |
| Scriptlet | `<% Java code %>` | Embedded Java code |
| Expression | `<%= expression %>` | Output a value |
| Declaration | `<%! method/variable %>` | Declare methods/variables |
| Comment | `<%-- comment --%>` | JSP comment (not in output) |
| EL Expression | `${expression}` | Expression Language (preferred!) |

### JSP Directives

```jsp
<%-- PAGE directive --%>
<%@ page contentType="text/html;charset=UTF-8" %>
<%@ page import="java.util.*, com.example.model.*" %>
<%@ page errorPage="error.jsp" %>
<%@ page isErrorPage="true" %>
<%@ page session="true" %>

<%-- INCLUDE directive (compile-time include) --%>
<%@ include file="header.jsp" %>

<%-- TAGLIB directive (use tag libraries like JSTL) --%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
```

---

## 11. JSP Implicit Objects

```
JSP gives you 9 objects automatically (no need to declare them):

| Object        | Type                    | Description                    |
|--------------|-------------------------|--------------------------------|
| request       | HttpServletRequest      | The HTTP request               |
| response      | HttpServletResponse     | The HTTP response              |
| session       | HttpSession             | The user's session             |
| application   | ServletContext          | Application-wide data          |
| out           | JspWriter               | Write output to response       |
| config        | ServletConfig           | Servlet configuration          |
| pageContext   | PageContext             | Access to all scopes           |
| page          | Object (this)           | The servlet instance           |
| exception     | Throwable               | Exception (error pages only)   |
```

---

## 12. JSTL & Expression Language

### Expression Language (EL) — Modern Way

```jsp
<%-- Instead of scriptlets, use EL: --%>

<%-- OLD (scriptlet — AVOID): --%>
<%= request.getAttribute("username") %>

<%-- NEW (EL — PREFERRED): --%>
${username}

<%-- EL Examples: --%>
${user.name}              <%-- calls user.getName() --%>
${user.address.city}      <%-- nested property --%>
${users[0].name}          <%-- list access --%>
${param.id}               <%-- request parameter --%>
${sessionScope.user}      <%-- session attribute --%>
${header["User-Agent"]}   <%-- request header --%>
${empty users}            <%-- true if null or empty --%>
${not empty users}        <%-- true if not null and not empty --%>
${2 + 3}                  <%-- arithmetic --%>
${age >= 18 ? 'Adult' : 'Minor'}  <%-- ternary --%>
```

### JSTL Core Tags

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<%-- IF --%>
<c:if test="${not empty user}">
    <p>Welcome, ${user.name}!</p>
</c:if>

<%-- IF-ELSE (choose/when/otherwise) --%>
<c:choose>
    <c:when test="${user.role == 'ADMIN'}">
        <p>Admin Dashboard</p>
    </c:when>
    <c:when test="${user.role == 'USER'}">
        <p>User Dashboard</p>
    </c:when>
    <c:otherwise>
        <p>Guest View</p>
    </c:otherwise>
</c:choose>

<%-- FOR EACH --%>
<table>
    <c:forEach var="user" items="${users}" varStatus="status">
        <tr>
            <td>${status.index + 1}</td>
            <td>${user.name}</td>
            <td>${user.email}</td>
        </tr>
    </c:forEach>
</table>

<%-- FOR LOOP (counting) --%>
<c:forEach var="i" begin="1" end="10" step="1">
    <p>Number: ${i}</p>
</c:forEach>

<%-- SET variable --%>
<c:set var="greeting" value="Hello, ${user.name}!" scope="request" />

<%-- URL with parameters --%>
<c:url var="userUrl" value="/users">
    <c:param name="id" value="${user.id}" />
</c:url>
<a href="${userUrl}">View User</a>

<%-- OUTPUT with HTML escaping (prevents XSS) --%>
<c:out value="${user.bio}" escapeXml="true" />

<%-- REDIRECT --%>
<c:redirect url="/login" />
```

---

## 13. MVC Pattern with Servlets & JSP

```
MODEL-VIEW-CONTROLLER:

  Browser → Controller (Servlet) → Model (Java class) → View (JSP)
  
  Controller: Handles request, calls business logic, forwards to view
  Model:      Business logic, data, database operations
  View:       Displays data (JSP, HTML)

┌─────────┐    Request    ┌────────────────┐
│ Browser  │─────────────►│   Controller   │
│          │              │   (Servlet)    │
│          │    Response  │                │
│          │◄─────────────│  1. Get params │
└─────────┘              │  2. Call Model │
                          │  3. Set attrs  │
                          │  4. Forward    │
                          │     to View   │
                          └──────┬─────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
              ┌─────▼─────┐           ┌──────▼──────┐
              │   Model   │           │    View     │
              │ (Service) │           │   (JSP)     │
              │           │           │             │
              │ UserService│          │ user.jsp    │
              │ UserDAO    │          │ ${user.name}│
              └───────────┘           └─────────────┘
```

```java
// CONTROLLER (Servlet)
@WebServlet("/users")
public class UserController extends HttpServlet {
    
    private UserService userService = new UserService();
    
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        
        // 1. Get parameters
        String action = req.getParameter("action");
        
        if ("view".equals(action)) {
            // 2. Call Model (business logic)
            Long id = Long.parseLong(req.getParameter("id"));
            User user = userService.findById(id);
            
            // 3. Set attributes for the View
            req.setAttribute("user", user);
            
            // 4. Forward to JSP (View)
            req.getRequestDispatcher("/WEB-INF/views/user-detail.jsp").forward(req, resp);
        } else {
            List<User> users = userService.findAll();
            req.setAttribute("users", users);
            req.getRequestDispatcher("/WEB-INF/views/user-list.jsp").forward(req, resp);
        }
    }
}
```

---

## 14. Servlets in Spring Boot

### Spring Boot's DispatcherServlet

```
Spring Boot uses an EMBEDDED Tomcat with a DispatcherServlet:

Browser → Tomcat → DispatcherServlet → HandlerMapping → Controller → ViewResolver → Response

DispatcherServlet IS a Servlet! It's the front controller that delegates to your @Controllers.
```

### Registering Custom Servlets in Spring Boot

```java
@SpringBootApplication
public class Application {
    
    @Bean
    public ServletRegistrationBean<MyServlet> myServlet() {
        return new ServletRegistrationBean<>(new MyServlet(), "/legacy/*");
    }
    
    @Bean
    public FilterRegistrationBean<MyFilter> myFilter() {
        FilterRegistrationBean<MyFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new MyFilter());
        bean.addUrlPatterns("/api/*");
        bean.setOrder(1);
        return bean;
    }
}
```

---

## 15. Best Practices

1. **Use JSP only for views** — Never put business logic in JSP
2. **Use EL and JSTL** instead of scriptlets — `${user.name}` not `<%= user.getName() %>`
3. **Put JSPs in WEB-INF** — Prevents direct browser access
4. **Use PreparedStatement** — Never concatenate SQL in servlets
5. **Use connection pooling** — Don't create DB connections per request
6. **Servlets are singletons** — Don't use instance variables for request data!
7. **Use filters for cross-cutting concerns** — Authentication, logging, CORS
8. **Prefer Spring MVC** over raw servlets for new projects
9. **Close resources** in destroy() method
10. **Handle exceptions properly** — Use error pages

---

## 16. Common Mistakes & Pitfalls

```java
// ❌ MISTAKE 1: Instance variables in Servlets (NOT thread-safe!)
@WebServlet("/counter")
public class CounterServlet extends HttpServlet {
    private int count = 0;  // SHARED across ALL threads!
    
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
        count++;  // RACE CONDITION! Multiple threads modify simultaneously
    }
}
// ✅ FIX: Use local variables or AtomicInteger

// ❌ MISTAKE 2: Writing after forwarding
req.getRequestDispatcher("/page.jsp").forward(req, resp);
resp.getWriter().println("This causes IllegalStateException!");
// ✅ FIX: Return after forward/redirect

// ❌ MISTAKE 3: Scriptlets in JSP
<%= request.getAttribute("name") %>
// ✅ FIX: Use Expression Language
${name}

// ❌ MISTAKE 4: Not escaping output (XSS vulnerability)
<%= request.getParameter("search") %>
// User sends: <script>alert('hacked')</script>
// ✅ FIX: <c:out value="${param.search}" escapeXml="true" />
```

---

## 17. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is a Servlet?**
**A:** A Java class running on a web server that handles HTTP requests and generates responses. It extends `HttpServlet` and overrides `doGet()`, `doPost()`, etc.

**Q2: What is the Servlet lifecycle?**
**A:** `init()` → called once on creation. `service()`/`doGet()`/`doPost()` → called for every request. `destroy()` → called once on shutdown.

**Q3: What is the difference between GET and POST?**
**A:** GET: data in URL (query string), bookmarkable, idempotent, limited size. POST: data in body, not bookmarkable, non-idempotent, unlimited size.

**Q4: What is a Servlet Container?**
**A:** Software that manages Servlet lifecycle, handles HTTP, manages threads, etc. Examples: Tomcat, Jetty, WildFly. Spring Boot uses embedded Tomcat.

**Q5: What is JSP?**
**A:** JavaServer Pages — HTML with embedded Java code. Gets compiled into a Servlet by the container. Used as the View in MVC pattern.

**Q6: What is the difference between forward and redirect?**
**A:** Forward: server-side, URL doesn't change, same request. Redirect: client-side (302), URL changes, new request.

**Q7: What is HttpSession?**
**A:** Server-side storage for user-specific data across multiple requests. Identified by JSESSIONID cookie. Used for login state, shopping carts, etc.

**Q8: Are Servlets thread-safe?**
**A:** No! Servlets are singletons — one instance serves all requests in different threads. Instance variables are shared. Use local variables for thread safety.

---

### Intermediate Level

**Q9: What is a Filter?**
**A:** Intercepts requests before they reach the servlet and responses before they reach the client. Used for logging, authentication, CORS, compression.

**Q10: What is the difference between ServletConfig and ServletContext?**
**A:** ServletConfig: specific to ONE servlet (init params). ServletContext: shared across the ENTIRE application (context params, attributes).

**Q11: What is RequestDispatcher?**
**A:** Allows forwarding or including another resource. `req.getRequestDispatcher("/page.jsp").forward(req, resp)`.

**Q12: Difference between `getSession()` and `getSession(false)`?**
**A:** `getSession()` creates a new session if one doesn't exist. `getSession(false)` returns null if no session exists (doesn't create one).

**Q13: What is JSTL?**
**A:** JSP Standard Tag Library — provides tags for loops, conditionals, formatting without Java code in JSP. Use `<c:forEach>`, `<c:if>` instead of scriptlets.

**Q14: What is Expression Language (EL)?**
**A:** `${expression}` syntax for accessing data in JSP without scriptlets. Accesses request attributes, session, beans, etc.

**Q15: What is the difference between include directive and include action?**
**A:** Directive (`<%@ include file="..." %>`) includes at compile time (static). Action (`<jsp:include page="..." />`) includes at runtime (dynamic).

---

### Advanced Level

**Q16: How does Spring Boot use Servlets internally?**
**A:** Spring Boot embeds Tomcat (a servlet container). It registers a `DispatcherServlet` which receives all requests and delegates to `@Controller` methods.

**Q17: What is the front controller pattern?**
**A:** A single servlet handles ALL requests and dispatches them to appropriate handlers. Spring MVC's `DispatcherServlet` is a front controller.

**Q18: How do Servlet Filters relate to Spring Security?**
**A:** Spring Security's entire architecture is based on a chain of Servlet Filters. Each filter handles one security concern (CSRF, authentication, authorization).

**Q19: What are async servlets?**
**A:** Servlets 3.0+ support asynchronous processing. The servlet thread is released while waiting for I/O, improving scalability.

**Q20: What is a Listener in Servlets?**
**A:** Classes that react to events: session creation/destruction, context initialization, attribute changes. Used for resource management, audit logging.

---

### Rapid-Fire (Q21–Q50)

**Q21: What is `loadOnStartup`?** Controls when servlet is initialized. Positive = at startup (lower number = higher priority). Negative/unset = on first request.

**Q22: What is `web.xml`?** Deployment descriptor for configuring servlets, filters, listeners, error pages. Annotations are the modern alternative.

**Q23: What is the default scope of a JSP variable?** Page scope (available only in the current JSP page).

**Q24: Four JSP scopes?** Page (current page), Request (current request), Session (user's session), Application (entire app).

**Q25: What is `sendError()` vs `setStatus()`?** sendError sends an error page. setStatus just sets the code without an error body.

**Q26: What is MIME type?** Content type indicating the nature of the response: text/html, application/json, image/png.

**Q27: What is `getServletContext()`?** Returns the ServletContext — application-wide configuration and shared attributes.

**Q28: What is `getContextPath()`?** Returns the URL prefix of the web application. e.g., `/myapp`.

**Q29: What is URL pattern `/*` vs `/`?** `/*` matches everything. `/` is the default servlet (matches when nothing else does).

**Q30: What is a WAR file?** Web Application Archive — a ZIP file containing servlets, JSPs, web.xml, libraries.

**Q31: `PrintWriter` vs `OutputStream`?** PrintWriter for text (char). OutputStream for binary (bytes). Can't use both in same response.

**Q32: What is `SingleThreadModel`?** Deprecated interface that serialized servlet access (one thread at a time). Don't use it.

**Q33: How to handle file upload in Servlet?** Use `@MultipartConfig` annotation and `req.getPart("file")`.

**Q34: What is `HttpServletRequestWrapper`?** A wrapper class to customize request behavior. Used in filters to modify request attributes.

**Q35: What is a cookie?** Small data sent by server, stored by browser, sent back with every request. Used for session tracking, preferences.

**Q36: `sendRedirect` vs `forward`?** Redirect: new request (URL changes, can go external). Forward: same request (URL stays, internal only).

**Q37: What is `response.encodeURL()`?** Appends session ID to URL if cookies are disabled (URL rewriting).

**Q38: Can we call `doGet()` from `doPost()`?** Yes: `doGet(req, resp)`. Used when GET and POST should behave the same.

**Q39: What is the difference between GenericServlet and HttpServlet?** GenericServlet is protocol-independent. HttpServlet extends it for HTTP-specific methods.

**Q40: What is `ServletInputStream`?** Input stream for reading binary request body data (file uploads, raw bytes).

**Q41: How to make a Servlet thread-safe?** Use local variables, synchronize shared resources, use concurrent data structures.

**Q42: What is `@WebServlet`?** Annotation for configuring a servlet (URL pattern, name, init params) without web.xml.

**Q43: What is `@WebFilter`?** Annotation for configuring a filter with URL patterns, servlet names, dispatcher types.

**Q44: What is `@WebListener`?** Annotation for registering an event listener (session, context, request lifecycle).

**Q45: What is `RequestDispatcher.include()`?** Includes the output of another resource in the current response.

**Q46: What is `ServletContextListener`?** Receives notification when the web app starts (`contextInitialized`) and stops (`contextDestroyed`).

**Q47: What happens if we call `forward()` after `flush()`?** `IllegalStateException` — response is already committed.

**Q48: What is `isCommitted()`?** Returns true if the response has been flushed/sent to the client.

**Q49: Difference between `destroy()` and garbage collection?** `destroy()` is called by the container for cleanup. GC reclaims memory. `destroy()` is called first, then GC.

**Q50: What replaces JSP in modern Spring Boot?** Thymeleaf (server-side templates) or React/Angular/Vue (client-side SPA).

---

## 📚 References

- [Jakarta Servlet Specification](https://jakarta.ee/specifications/servlet/)
- [Oracle Servlet Tutorial](https://docs.oracle.com/javaee/7/tutorial/servlets.htm)
- [Baeldung Servlet Tutorials](https://www.baeldung.com/intro-to-servlets)
- [JSTL Documentation](https://jakarta.ee/specifications/tags/)
- [Spring DispatcherServlet](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet.html)

---

> **Previous Topic:** [← 07 - JDBC](../07-jdbc/README.md)  
> **Next Topic:** [09 - REST API and Web Services →](../09-rest-api-and-web-services/README.md)
