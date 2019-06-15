# Shiro 会话固定问题及其解决方案

## 会话固定

[会话固定](https://www.owasp.org/index.php/Session_fixation)问题是指一个系统在登录前后没有对其 sessionId 更新，导致攻击者可以利用 sessionId 登录系统，其流程如下：

1. 攻击者发送带有 sessioinId 信息的链接，这里设 sessionId = a；
2. 用户通过该链接登录系统，但是此时 sessionId 仍旧为 a，没有变化；
3. 攻击者不断尝试登录，假如用户登录成功，那么攻击者即马上可以获取该用户的信息。

## 问题复现

shiro 为目前比较主流且容易学习的安全框架，但 shiro 框架内对会话固定问题没有解决。[这里是一个简单的例子](https://github.com/zeahoo/shirodis/)来复现该问题。


## 解决方案

会话固定问题的关键在于用户登录系统没有更新 sessionId，那么可以通过每次登录修改 sessionId 来解决这个问题。shiro执行登录的方法在 FormAuthenticationFilter 类的 executeLogin 方法中，可以通过重写该方法来解决这个问题，代码的方案参考[这个邮件](http://mail-archives.apache.org/mod_mbox/shiro-dev/201507.mbox/%3CJIRA.12465307.1274733244000.151603.1436511965601@Atlassian.JIRA%3E)，可以使用[github比较](https://github.com/zeahoo/shirodis/commit/3aec453bab7475ab100325ddcd08560f034c6523)来查看修复前后代码的差异，其中，重写的代码如下：

```java
package org.sonny.auth;

import java.util.Collection;
import java.util.LinkedHashMap;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.session.Session;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.web.filter.authc.FormAuthenticationFilter;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

@Component
public class MyFormAuthenticationFilter extends FormAuthenticationFilter {


  private static final Logger log = LoggerFactory.getLogger(MyFormAuthenticationFilter.class);

  @Override
  protected boolean executeLogin(final ServletRequest request, final ServletResponse response)
      throws Exception {
    final AuthenticationToken token = createToken(request, response);
    if (token == null) {
      String msg =
          "createToken method implementation returned null. A valid non-null AuthenticationToken"
              + "must be created in order to execute a login attempt.";
      throw new IllegalStateException(msg);
    }
    try {
      // Stop session fixation issues.
      // https://issues.apache.org/jira/browse/SHIRO-170
      final Subject subject = getSubject(request, response);
      Session session = subject.getSession();
      String old_id = (String) session.getId();
      // Store the attributes so we can copy them to the new session after auth.
      final LinkedHashMap<Object, Object> attributes = new LinkedHashMap<Object, Object>();
      final Collection<Object> keys = session.getAttributeKeys();
      for (Object key : keys) {
        final Object value = session.getAttribute(key);
        if (value != null) {
          attributes.put(key, value);
        }
      }
      session.stop();

      subject.login(token);
      // Restore the attributes.
      session = subject.getSession();
      log.debug("OWASP session fixation  from " + old_id + " to " + session.getId());
      for (final Object key : attributes.keySet()) {
        session.setAttribute(key, attributes.get(key));
      }
      return onLoginSuccess(token, subject, request, response);
    } catch (AuthenticationException e) {
      return onLoginFailure(token, e, request, response);
    }
  }
}

```