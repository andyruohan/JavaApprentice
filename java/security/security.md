### XSS（跨站脚本攻击，Cross-Site Scripting）
问题：注入恶意脚本到网页，用户浏览时脚本在浏览器执行，可窃取 Cookie、篡改页面内容，核心因前后端未过滤输入、未编码输出导致。
防护：后端严格过滤 / 转义恶意字符；前端渲染时做对应编码；配置 CSP 安全头；为 Cookie 设置 HttpOnly（防脚本窃取）和 Secure（仅 HTTPS 传输）属性。

### CSRF（跨站请求伪造，Cross-Site Request Forgery，又称 XSRF）
问题：借助浏览器自动携带 Cookie 的特性，伪造用户已认证的恶意请求，诱导用户触发，进而执行非本意操作（如转账、改密码），核心因后端未校验请求来源导致。
防护：使用 CSRF Token 随机校验；后端验证请求 Referer/Origin 字段；采用双重 Cookie 验证；敏感操作（如支付、修改信息）增加短信 / 验证码二次验证。