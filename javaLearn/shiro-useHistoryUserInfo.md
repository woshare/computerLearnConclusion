# security shiro获取的是前一个用户的数据

## 一，现象
>1，用户1登录，做了一些其他业务请求，退出      
>2，用户2登录，发现返回的是用户1的数据    

## 二，猜测
>1，客户端有用keepalive和服务器端shiro ThreadLocal导致，可能导致用户1登录和用户2登录，两次http请求使用的一个客户端tcp链接，对接到服务器端同一个链接同一个线程上，又shiro有基于ThreadLocal保存用户信息，故而有如上现象     
>2，客户端有使用cookie和服务器端有session导致，可能用户1登录保存了用户1的cookie，用户2登录的时候带上了用户1的cookie，故而读取的用户1的信息          

## 三，定位问题

### 客户端有用keepalive和服务器端shiro ThreadLocal问题
>1，客户端keepalive，需要客户端支持联调      
>2，服务器端打印了threadId，ThreadLocal，shiro subject中的数据       
>3，测试发现，用户1登录和用户2登录使用的是两个不同的线程，但是shiro subject的数据是一样的，如此可以暂时排除服务器同一个线程中threadLocal问题，经过本地debug发现，可能有服务器会话的可能性较大         
>4，结论，排除这个可能          

### 客户端有使用cookie和服务器端有session问题
>1，本地debug要求，idea开启debug模式，能支持两次以上的请求，以方便模拟用户1登录请求和用户2登录请求，关键设置是：设置debug配置，允许Leave enabled，并且用F9调试        
>2，因为怀疑客户端有cookie，则打印所有http header中的参数，发现确实有cookie，如下         
```
Set-Cookie: JSESSIONID=76CD05659A8AFC4A5E025D9213FFADEC; Path=/; HttpOnly
Set-Cookie: rememberMe=deleteMe; Path=/; Max-Age=0; Expires=Tue, 23-Nov-2021 06:58:19 GMT; SameSite=lax
```
>3，在本地测试，获取到了用户1登录返回的cookie数据，并在用户2登录请求上带上cookie数据，用F9一步步调试，看idea形成的调试栈空间，并一步步定位何时处理cookie并转为服务器session的。   
>4，一句话总结就是：**有一个全局的session CurrentHashmap存储了session，通过JSESSIONID（就是sessionId）获取session，通过session的attribute，获取subject 的principal的数据，该数据就存储的是用户信息**。    
>5，补充一点的是，shiro还支持 **记住我（rememberMe）**，如上cookie所示，基于cookie来实现的，不过当前我们是没有用到这个功能，故而，**Set-Cookie: rememberMe=deleteMe**    

#### 定位如下，关键代码

#### DefaultSecurityManager.java
```
public Subject createSubject(SubjectContext subjectContext) {
        //create a copy so we don't modify the argument's backing map:
        SubjectContext context = copy(subjectContext);

        //ensure that the context has a SecurityManager instance, and if not, add one:
        context = ensureSecurityManager(context);

        //Resolve an associated Session (usually based on a referenced session ID), and place it in the context before
        //sending to the SubjectFactory.  The SubjectFactory should not need to know how to acquire sessions as the
        //process is often environment specific - better to shield the SF from these details:
        context = resolveSession(context);

        //Similarly, the SubjectFactory should not require any concept of RememberMe - translate that here first
        //if possible before handing off to the SubjectFactory:
        context = resolvePrincipals(context);

        Subject subject = doCreateSubject(context);

        //save this subject for future reference if necessary:
        //(this is needed here in case rememberMe principals were resolved and they need to be stored in the
        //session, so we don't constantly rehydrate the rememberMe PrincipalCollection on every operation).
        //Added in 1.2:
        save(subject);

        return subject;
    }
```
#### Request.java
```
 protected Session doGetSession(boolean create) {

        // There cannot be a session if no context has been assigned yet
        Context context = getContext();
        if (context == null) {
            return null;
        }

        // Return the current session if it exists and is valid
        if ((session != null) && !session.isValid()) {
            session = null;
        }
        if (session != null) {
            return session;
        }

        // Return the requested session if it exists and is valid
        Manager manager = context.getManager();
        if (manager == null) {
            return null;      // Sessions are not supported
        }
        if (requestedSessionId != null) {
            try {
                session = manager.findSession(requestedSessionId);
            } catch (IOException e) {
                session = null;
            }
            if ((session != null) && !session.isValid()) {
                session = null;
            }
            if (session != null) {
                session.access();
                return session;
            }
        }

        // Create a new session if requested and the response is not committed
        if (!create) {
            return null;
        }
        boolean trackModesIncludesCookie =
                context.getServletContext().getEffectiveSessionTrackingModes().contains(SessionTrackingMode.COOKIE);
        if (trackModesIncludesCookie && response.getResponse().isCommitted()) {
            throw new IllegalStateException(sm.getString("coyoteRequest.sessionCreateCommitted"));
        }

        // Re-use session IDs provided by the client in very limited
        // circumstances.
        String sessionId = getRequestedSessionId();
        if (requestedSessionSSL) {
            // If the session ID has been obtained from the SSL handshake then
            // use it.
        } else if (("/".equals(context.getSessionCookiePath())
                && isRequestedSessionIdFromCookie())) {
            /* This is the common(ish) use case: using the same session ID with
             * multiple web applications on the same host. Typically this is
             * used by Portlet implementations. It only works if sessions are
             * tracked via cookies. The cookie must have a path of "/" else it
             * won't be provided for requests to all web applications.
             *
             * Any session ID provided by the client should be for a session
             * that already exists somewhere on the host. Check if the context
             * is configured for this to be confirmed.
             */
            if (context.getValidateClientProvidedNewSessionId()) {
                boolean found = false;
                for (Container container : getHost().findChildren()) {
                    Manager m = ((Context) container).getManager();
                    if (m != null) {
                        try {
                            if (m.findSession(sessionId) != null) {
                                found = true;
                                break;
                            }
                        } catch (IOException e) {
                            // Ignore. Problems with this manager will be
                            // handled elsewhere.
                        }
                    }
                }
                if (!found) {
                    sessionId = null;
                }
            }
        } else {
            sessionId = null;
        }
        session = manager.createSession(sessionId);

        // Creating a new session cookie based on that session
        if (session != null && trackModesIncludesCookie) {
            Cookie cookie = ApplicationSessionCookieConfig.createSessionCookie(
                    context, session.getIdInternal(), isSecure());

            response.addSessionCookieInternal(cookie);
        }

        if (session == null) {
            return null;
        }

        session.access();
        return session;
    }
		
 public PrincipalCollection resolvePrincipals() {
        PrincipalCollection principals = getPrincipals();

        if (isEmpty(principals)) {
            //check to see if they were just authenticated:
            AuthenticationInfo info = getAuthenticationInfo();
            if (info != null) {
                principals = info.getPrincipals();
            }
        }

        if (isEmpty(principals)) {
            Subject subject = getSubject();
            if (subject != null) {
                principals = subject.getPrincipals();
            }
        }

        if (isEmpty(principals)) {
            //try the session:
            Session session = resolveSession();
            if (session != null) {
                principals = (PrincipalCollection) session.getAttribute(PRINCIPALS_SESSION_KEY);
            }
        }

        return principals;
    }
```
##### CookieRememberMeManager	：
```
protected byte[] getRememberedSerializedIdentity(SubjectContext subjectContext) {

        if (!WebUtils.isHttp(subjectContext)) {
            if (log.isDebugEnabled()) {
                String msg = "SubjectContext argument is not an HTTP-aware instance.  This is required to obtain a " +
                        "servlet request and response in order to retrieve the rememberMe cookie. Returning " +
                        "immediately and ignoring rememberMe operation.";
                log.debug(msg);
            }
            return null;
        }

        WebSubjectContext wsc = (WebSubjectContext) subjectContext;
        if (isIdentityRemoved(wsc)) {
            return null;
        }

        HttpServletRequest request = WebUtils.getHttpRequest(wsc);
        HttpServletResponse response = WebUtils.getHttpResponse(wsc);

        String base64 = getCookie().readValue(request, response);
        // Browsers do not always remove cookies immediately (SHIRO-183)
        // ignore cookies that are scheduled for removal
        if (Cookie.DELETED_COOKIE_VALUE.equals(base64)) return null;

        if (base64 != null) {
            base64 = ensurePadding(base64);
            if (log.isTraceEnabled()) {
                log.trace("Acquired Base64 encoded identity [" + base64 + "]");
            }
            byte[] decoded;
            try {
                decoded = Base64.decode(base64);
            } catch (RuntimeException rtEx) {
                /*
                 * https://issues.apache.org/jira/browse/SHIRO-766:
                 * If the base64 string cannot be decoded, just assume there is no valid cookie value.
                 * */
                getCookie().removeFrom(request, response);
                log.warn("Unable to decode existing base64 encoded entity: [" + base64 + "].", rtEx);
                return null;
            }

            if (log.isTraceEnabled()) {
                log.trace("Base64 decoded byte array length: " + decoded.length + " bytes.");
            }
            return decoded;
        } else {
            //no cookie set - new site visitor?
            return null;
        }
    }
```

## 四，解决方案
>1，主要配置不创建session： context.setSessionCreationEnabled(false)等。如下：

```
 @Bean
    protected ShiroFilterFactoryBean shiroFilterFactoryBean(ShiroFilterChainDefinition shiroFilterChainDefinition, SecurityManager securityManager) {
        ShiroFilterFactoryBean filterFactoryBean = new ShiroFilterFactoryBean();
        DefaultWebSecurityManager webSecurityManager = (DefaultWebSecurityManager) securityManager;

        DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
        DefaultSessionStorageEvaluator defaultSessionStorageEvaluator = new DefaultSessionStorageEvaluator();
        
        //设置不存储session
        defaultSessionStorageEvaluator.setSessionStorageEnabled(false);

        subjectDAO.setSessionStorageEvaluator(defaultSessionStorageEvaluator);
        webSecurityManager.setSubjectDAO(subjectDAO);

        //设置不生成session的subjectContext
        webSecurityManager.setSubjectFactory(subjectFactory());
        
        //设置 关闭轮询校验session的 会话管理器，因为会话也是有有效期的，需要有额外的开销去维护
        webSecurityManager.setSessionManager(sessionManager());
        
        
        filterFactoryBean.setSecurityManager(securityManager);
        filterFactoryBean.setFilterChainDefinitionMap(shiroFilterChainDefinition.getFilterChainMap());
        Map<String, Filter> filterMap = new HashMap<>();
        //自定义拦截器
        filterMap.put("auth", new AuthFilter());
        filterFactoryBean.setFilters(filterMap);
        return filterFactoryBean;
    }


    public DefaultSubjectFactory subjectFactory() {
        return new StatelessSubjectFactory();
    }

    public SessionManager sessionManager() {
        DefaultSessionManager sessionManager = new DefaultSessionManager();
        // 关闭session校验轮询
        sessionManager.setSessionValidationSchedulerEnabled(false);
        return sessionManager;
    }

    public class StatelessSubjectFactory extends DefaultWebSubjectFactory {

    @Override
    public Subject createSubject(SubjectContext context) {
        //不创建session
        context.setSessionCreationEnabled(false);
        return super.createSubject(context);
    }
}
```