---
title: JWT浅析
date: 2020-05-25 13:59:24
tags: [Spring Boot, JWT, 后端]
---

通常情况下，把API直接暴露出去是风险很大的，不说别的，直接被机器攻击就喝一壶的。那么一般来说，对API要划分出一定的权限级别，然后做一个用户的鉴权，依据鉴权结果给予用户开放对应的API。目前，比较主流的方案有几种:

1. 用户名和密码鉴权，使用Session保存用户鉴权结果。
2. 使用OAuth进行鉴权（其实OAuth也是一种基于Token的鉴权，只是没有规定Token的生成方式）
3. 自行采用Token进行鉴权

<!-- more -->

> 在了解JWT之前先来回顾一下传统session认证和基于token认证。

## 传统session认证

http协议是一种无状态协议，即浏览器发送请求到服务器，服务器是不知道这个请求是哪个用户发来的。为了让服务器知道请求是哪个用户发来的，需要让用户提供用户名和密码来进行认证。当浏览器第一次访问服务器(假设是登录接口),服务器验证用户名和密码之后，服务器会生成一个sessionid(只有第一次会生成，其它会使用同一个sessionid),并将该session和用户信息关联起来，然后将sessionid返回给浏览器，浏览器收到sessionid保存到Cookie中，当用户第二次访问服务器是就会携带Cookie值，服务器获取到Cookie值，进而获取到sessionid，根据sessionid获取关联的用户信息。

```java
public User login(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
    String username = request.getParameter("username");
    String password = request.getParameter("password");

    User user = userService.login(username, password);
    if (user == null) {
        throw new AuthenticationException("用户名或密码错误");
    }

    // 返回这次请求关联的当前会话，如果没有会话则创建一个新的
    // 需要在服务器端记录该session
    HttpSession session = request.getSession();
    session.setAttribute("user", user);

    // 让浏览器保存sessionid到cookie中
    // Cookie cookie = new Cookie("sessionid", session.getId());
    // cookie.setPath("/");
    // response.addCookie(cookie);
    return user;
}

public Object getUserInfo(HttpServletRequest request){
    // 从request中获取Cookie
    // 从Cookie中获取sessionid
    // 根据sessionid获取对应的Session对象
    // 从session中获取关联的用户信息
    HttpSession session = request.getSession();
    Object user = session.getAttribute("user");

    return user;
}

```

session的缺点：

- Session: 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大。
- 扩展性: 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。即不能满足单点登陆
- CSRF: 因为是基于cookie来进行用户识别的, cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。
  

## 基于token认证

token原理：

1. 使用用户名和密码请求登录接口
2. 登录接口验证用户名和密码
3. 登录接口生成一个uuid作为token，将用户信息作为值，然后保存到redis缓存中jedis.set(token, user);
4. 登录接口返回用户信息和token
5. 浏览器将token保存到本地
6. 当请求其它接口时就携带token值
7. 接口根据token去缓存中查，如果找到了就调用接口，如果找不到报token错误(一般通过拦截器来实现检查)

```java
public String auth(String username, String password) throws AuthenticationException {
    User user = userService.login(username, password);
    if (user == null) {
        throw new AuthenticationException("用户名或密码错误");
    }

    String token = UUID.randomUUID().toString();
    redisClient.set(token, user);

    return token;
}

public Object getUserInfo(@RequestHeader("token") String token) throws AuthenticationException {
    User user = redisClient.get(token);
    if (user == null) {
        throw new AuthenticationException("token不可用");
    }

    return user;
}
```



**session和token的区别：**

​	此种方式原理上和session方式差不多，都是客户端调用接口时携带一个值，服务器通过该值来获取用户的信息。
​	不同的是session是将信息保存到本机内存中，对负载均衡有限制(只能负载到同一台机器)，token是保存到缓存服务器(redis)中，对负载均衡没有限制，如果使用同一个redis服务器还可以保证单点登录。session一般用在PC上，token即可用在PC上也可以用在APP上。

> 接下来，我们开始学习JWT啦

## JWT

### 简介

JSON Web Token（JWT）是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（(RFC 7519)，它定义了一种紧凑（Compact）且自包含（Self-contained）的方式，用于在各方之间以JSON对象安全传输信息。 这些信息可以通过数字签名进行验证和信任。 可以使用秘密（使用HMAC算法）或使用RSA的公钥/私钥对对JWT进行签名。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。是目前最流行的跨域认证解决方案。

- Compact（紧凑）：
  由于它们尺寸较小，JWT可以通过URL，POST参数或HTTP标头内发送。 另外，尺寸越小意味着传输速度越快。
- Self-contained（自包含）:
  有效载荷（Playload）包含有关用户的所有必需信息，避免了多次查询数据库。

###应用场景

- Authentication（鉴权）：
  这是使用JWT最常见的情况。 一旦用户登录，每个后续请求都将包含JWT，允许用户访问该令牌允许的路由，服务和资源。 单点登录是当今广泛使用JWT的一项功能，因为它的开销很小，并且能够轻松地跨不同域使用。
- 分布式站点的单点登录（SSO）
- Information Exchange（信息交换）：
  JSON Web Tokens是在各方之间安全传输信息的好方式。 因为JWT可以签名：例如使用公钥/私钥对，所以可以确定发件人是他们自称的人。 此外，由于使用标头和有效载荷计算签名，因此您还可以验证内容是否未被篡改。

### JWT的工作流程

下面是一个JWT的工作流程图。模拟一下实际的流程是这样的（假设受保护的API在`/protected`中）

1. 用户导航到登录页，输入用户名、密码，进行登录
2. 服务器验证登录鉴权，如果改用户合法，根据用户的信息和服务器的规则生成JWT Token
3. 服务器将该token以json形式返回（不一定要json形式，这里说的是一种常见的做法）
4. 用户得到token，存在localStorage、cookie或其它数据存储形式中。
5. 以后用户请求`/protected`中的API时，在请求的header中加入 `Authorization: Bearer xxxx(token)`。此处注意token之前有一个7字符长度的 `Bearer`
6. 服务器端对此token进行检验，如果合法就解析其中内容，根据其拥有的权限和自己的业务逻辑给出对应的响应结果。
7. 用户取得结果

![image.png](https://ae01.alicdn.com/kf/Hed8e7c3f1c474acaaf43d075cef9c865w.png)

为了更好的理解这个token是什么，我们先来看一个token生成后的样子，下面那坨乱糟糟的就是了。

```css
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ3YW5nIiwiY3JlYXRlZCI6MTQ4OTA3OTk4MTM5MywiZXhwIjoxNDg5Njg0NzgxfQ.RC-BYCe_UZ2URtWddUpWXIp4NMsoeq2O6UF-8tVplqXY1-CI9u1-a-9DAAJGfNWkHE81mpnR3gXzfrBAB3WUAg
```

但仔细看到的话还是可以看到这个token分成了三部分，每部分用 `.` 分隔，每段都是用 [Base64](https://link.jianshu.com?t=https://en.wikipedia.org/wiki/Base64) 编码的。如果我们用一个Base64的解码器的话 （ [https://www.base64decode.org/](https://link.jianshu.com?t=https://www.base64decode.org/) ），可以看到第一部分 `eyJhbGciOiJIUzUxMiJ9` 被解析成了:

```json
{
    "alg":"HS512"
}
```

这是告诉我们HMAC采用HS512算法对JWT进行的签名。

第二部分 `eyJzdWIiOiJ3YW5nIiwiY3JlYXRlZCI6MTQ4OTA3OTk4MTM5MywiZXhwIjoxNDg5Njg0NzgxfQ` 被解码之后是

```json
{
    "sub":"wang",
    "created":1489079981393,
    "exp":1489684781
}
```

这段告诉我们这个Token中含有的数据声明（Claim），这个例子里面有三个声明：`sub`, `created` 和 `exp`。在我们这个例子中，分别代表着用户名、创建时间和过期时间，当然你可以把任意数据声明在这里。

看到这里，你可能会想这是个什么鬼token，所有信息都透明啊，安全怎么保障？别急，我们看看token的第三段 `RC-BYCe_UZ2URtWddUpWXIp4NMsoeq2O6UF-8tVplqXY1-CI9u1-a-9DAAJGfNWkHE81mpnR3gXzfrBAB3WUAg`。同样使用Base64解码之后，咦，这是什么东东

```bash
D X �DmYTeȧL�UZcPZ0$gZAY�_7�wY@ 
```

最后一段其实是签名，这个签名必须知道秘钥才能计算。这个也是JWT的安全保障。这里提一点注意事项，由于数据声明（Claim）是公开的，千万不要把密码等敏感字段放进去，否则就等于是公开给别人了。

也就是说JWT是由三段组成的，按官方的叫法分别是header（头）、payload（负载）和signature（签名）：

```css
header.payload.signature
```

头中的数据通常包含两部分：一个是我们刚刚看到的 `alg`，这个词是 `algorithm` 的缩写，就是指明算法。另一个可以添加的字段是token的类型(按RFC 7519实现的token机制不只JWT一种)，但如果我们采用的是JWT的话，指定这个就多余了。

```json
{
  "alg": "HS512",
  "typ": "JWT"
}
```

payload中可以放置三类数据：系统保留的、公共的和私有的：

- 系统保留的声明（Reserved claims）：这类声明不是必须的，但是是建议使用的，包括：iss (签发者), exp (过期时间),
   sub (主题), aud (目标受众)等。这里我们发现都用的缩写的三个字符，这是由于JWT的目标就是尽可能小巧。
- 公共声明：这类声明需要在 [IANA JSON Web Token Registry](https://link.jianshu.com?t=http://www.iana.org/assignments/jwt/jwt.xhtml) 中定义或者提供一个URI，因为要避免重名等冲突。
- 私有声明：这个就是你根据业务需要自己定义的数据了。

签名的过程是这样的：采用header中声明的算法，接受三个参数：base64编码的header、base64编码的payload和秘钥（secret）进行运算。签名这一部分如果你愿意的话，可以采用RSASHA256的方式进行公钥、私钥对的方式进行，如果安全性要求的高的话。

```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

> 看了这些是不是感觉有些迷惑，我们来详细的看一看语法吧

### 语法

jwt有3个组成部分，每部分通过点号来分割 header.payload.signature

- 头部（header) 是一个 JSON 对象，描述 JWT 的元数据，通常是下面的样子

- 载荷（payload) 是一个 JSON 对象，用来存放实际需要传递的数据

- 签证（signature) 对header和payload使用密钥进行签名，防止数据篡改。

  

#### ① 头部header

Jwt的头部是一个JSON,然后使用Base64URL编码，承载两部分信息：

- 声明类型typ，表示这个令牌（token）的类型（type），JWT令牌统一写为JWT
- 声明加密的算法alg，通常直接使用HMACSHA256，就是HS256了，也可以使用RSA,支持很多算法(HS256、HS384、HS512、RS256、RS384、RS512、ES256、ES384、ES512、PS256、PS384)
  


```java
var header = Base64URL({ "alg": "HS256", "typ": "JWT"})
```

**Base64URL**：Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）\。Base64 有三个字符+、/和=，在 URL 里面有特殊含义，所以要被替换掉：=被省略、+替换成-，/替换成_ 。这就是 Base64URL 算法。

#### ② 载荷payload

payload也是一个JSON字符串，是承载消息具体内容的地方，也需要使用Base64URL编码，payload中可以包含预定义的7个可用，它们不是强制性的，但推荐使用，也可以添加任意自定义的key

1. iss(issuer): jwt签发者
2. sub(subject): jwt所面向的用户
3. aud(audience): 接收jwt的一方, 受众
4. exp(expiration time): jwt的过期时间，这个过期时间必须要大于签发时间
5. nbf(Not Before): 生效时间，定义在什么时间之前.
6. iat(Issued At): jwt的签发时间
7. jti(JWT ID): jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。

```java
// 该token签发给1234567890，姓名为John Doe(自定义的字段)，签发时间为1516239022
var payload = Base64URL( {"sub": "1234567890", "name": "John Doe", "iat": 1516239022})
```

> 注意，JWT中payload是不加密的，只是Base64URL编码一下，任何人拿到都可以进行解码，所以不要把敏感信息放到里面。

#### ③ signature

Signature 部分是对前两部分的签名，防止数据篡改。

```java
var header = Base64URL({ "alg": "HS256", "typ": "JWT"});
var payload = Base64URL( {"sub": "1234567890", "name": "John Doe", "iat": 1516239022});
var secret = "私钥";
var signature = HMACSHA256(header + "." + payload, secret);
var jwt = header + "." + payload + "." + signature;
```



### jwt的特点

- 因为json的通用性，所以JWT是可以进行跨语言支持的，像JAVA,JavaScript,NodeJS,PHP等很多语言都可以使用。
- 因为有了payload部分，所以JWT可以在自身存储一些其他业务逻辑所必要的非敏感信息。
- 它不需要在服务端保存会话信息, 所以它易于应用的扩展

***JWT 的几个特点***
（1）JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。
（2）JWT 不加密的情况下，不能将敏感数据(如密码)写入 JWT，除非对payload进行加密。保护好secret私钥，该私钥非常重要。
（3）JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。
（4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
（5）JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。

（6）为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

#### JWT的优点：

- 体积小，因而传输速度更快
- 多样化的传输方式，可以通过URL传输、POST传输、请求头Header传输（常用）
- 简单方便，服务端拿到jwt后无需再次查询数据库校验token可用性，也无需进行redis缓存校验
- 在分布式系统中，很好地解决了单点登录问题
- 很方便的解决了跨域授权问题，因为跨域无法共享cookie

### JWT的缺点：

- 因为JWT是无状态的，因此服务端无法控制已经生成的Token失效，是不可控的，这一点对于是否使用jwt是需要重点考量的
- 获取到jwt也就拥有了登录权限，因此jwt是不可泄露的，网站最好使用https，防止中间攻击偷取jwt
- 在退出登录 / 修改密码时怎样实现JWT Token失效https://segmentfault.com/q/1010000010043871



## JWT的生成和解析

为了简化我们的工作，这里引入一个比较成熟的JWT类库，叫 [jjwt](https://github.com/jwtk/jjwt)。这个类库可以用于Java和Android的JWT token的生成和验证。

**如果要构建（非Android）JDK项目，则需要定义以下依赖项：**

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.1</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.1</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId> <!-- or jjwt-gson if Gson is preferred -->
    <version>0.11.1</version>
    <scope>runtime</scope>
</dependency>
```

> 到这就可以了

(听说下面这样也可以)

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.11.1</version>
</dependency>
```

application.properties

```java
jwt.secret=JO6HN3NGIU25G2FIG8V7VD6CK9B6T2Z5
jwt.expire=60000
```

**JwtToken**

```java
@Configuration
public class JwtToken {
    private static Logger logger = LoggerFactory.getLogger(JwtToken.class);

    /** 秘钥 */
    @Value("${jwt.secret}")
    private String secret;

    /** 过期时间(秒) */
    @Value("${jwt.expire}")
    private long expire;


    /**
     * 生成jwt token
     */
    public String generateToken(Long userId) {
        Date nowDate = new Date();
        Date expireDate = new Date(nowDate.getTime() + expire * 1000);
        return Jwts.builder()
                .setHeaderParam("typ", "JWT")
                .setSubject(userId + "")
                .setIssuedAt(nowDate)
                .setExpiration(expireDate)
                .signWith(SignatureAlgorithm.HS512, secret)
                .compact();
    }

    public Claims getClaimByToken(String token) {
        if (StringUtils.isEmpty(token)) {
            return null;
        }

        String[] header = token.split("Bearer");
        token = header[1];
        try {
            return Jwts.parser()
                    .setSigningKey(secret)
                    .parseClaimsJws(token)
                    .getBody();
        }catch (Exception e){
            logger.debug("validate is token error ", e);
            return null;
        }
    }

    /**
     * token是否过期
     * @return  true：过期
     */
    public static boolean isTokenExpired(Date expiration) {
        return expiration.before(new Date());
    }

    // Getter && Setter
}
```

**JwtController**

```java
@RestController
public class JwtController {

    @Autowired
    private JwtToken jwtToken;

    @PostMapping("/login")
    public String login(User user) {
        // 1. 验证用户名和密码
        // 2. 验证成功生成token
        Long userId = 666L;
        String token = jwtToken.generateToken(userId);
        return token;
    }

    @GetMapping("/getUserInfo")
    public String getUserInfo(@RequestHeader("Authorization") String authHeader) throws AuthenticationException {
        // 黑名单token
        List<String> blacklistToken = Arrays.asList("禁止访问的token");
        Claims claims = jwtToken.getClaimByToken(authHeader);
        if (claims == null || JwtToken.isTokenExpired(claims.getExpiration()) || blacklistToken.contains(authHeader)) {
            throw new AuthenticationException("token 不可用");
        }

        String userId = claims.getSubject();
        // 根据用户id获取接口数据返回接口
        return userId;
    }
}
```

