## Java 操作 JWT

JWT 是一种理念/方案，它与具体的语言无关。Java 语言中有 `jjwt` 包来简化我们创建/解析 JWT 。

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>

<!-- jjwt 依赖于 java-jwt -->
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.4.0</version>
</dependency>
```

代码：

```java
import java.util.*;
import com.auth0.jwt.*;
import com.auth0.jwt.algorithms.Algorithm;
import io.jsonwebtoken.*;
import org.apache.commons.codec.binary.Base64;

import java.util.*;


public class JwtUtil {

    // 生成签名是所使用的秘钥
    private final String base64EncodedSecretKey;

    // 生成签名的时候所使用的加密算法
    private final SignatureAlgorithm signatureAlgorithm;

    public JwtUtil(String secretKey, SignatureAlgorithm signatureAlgorithm) {
        this.base64EncodedSecretKey = Base64.encodeBase64String(secretKey.getBytes());
        this.signatureAlgorithm = signatureAlgorithm;
    }

    /**
     * 生成 JWT Token 字符串
     *
     * @param iss       签发人名称
     * @param ttlMillis jwt 过期时间
     * @param claims    额外添加到荷部分的信息。
     *                  例如可以添加用户名、用户ID、用户（加密前的）密码等信息
     */
    public String encode(String iss, long ttlMillis, Map<String, Object> claims) {
        if (claims == null) {
            claims = new HashMap<>();
        }

        // 签发时间（iat）：荷载部分的标准字段之一
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);

        // 下面就是在为payload添加各种标准声明和私有声明了
        JwtBuilder builder = Jwts.builder()
                // 荷载部分的非标准字段/附加字段，一般写在标准的字段之前。
                .setClaims(claims)
                // JWT ID（jti）：荷载部分的标准字段之一，JWT 的唯一性标识，虽不强求，但尽量确保其唯一性。
                .setId(UUID.randomUUID().toString())
                // 签发时间（iat）：荷载部分的标准字段之一，代表这个 JWT 的生成时间。
                .setIssuedAt(now)
                // 签发人（iss）：荷载部分的标准字段之一，代表这个 JWT 的所有者。通常是 username、userid 这样具有用户代表性的内容。
                .setSubject(iss)
                // 设置生成签名的算法和秘钥
                .signWith(signatureAlgorithm, base64EncodedSecretKey);

        if (ttlMillis >= 0) {
            long expMillis = nowMillis + ttlMillis;
            Date exp = new Date(expMillis);
            // 过期时间（exp）：荷载部分的标准字段之一，代表这个 JWT 的有效期。
            builder.setExpiration(exp);
        }

        return builder.compact();
    }


    /**
     * JWT Token 由 头部 荷载部 和 签名部 三部分组成。签名部分是由加密算法生成，无法反向解密。
     * 而 头部 和 荷载部分是由 Base64 编码算法生成，是可以反向反编码回原样的。
     * 这也是为什么不要在 JWT Token 中放敏感数据的原因。
     *
     * @param jwtToken 加密后的token
     * @return claims 返回荷载部分的键值对
     */
    public Claims decode(String jwtToken) {

        // 得到 DefaultJwtParser
        return Jwts.parser()
                // 设置签名的秘钥
                .setSigningKey(base64EncodedSecretKey)
                // 设置需要解析的 jwt
                .parseClaimsJws(jwtToken)
                .getBody();
    }


    /**
     * 校验 token
     * 在这里可以使用官方的校验，或，
     * 自定义校验规则，例如在 token 中携带密码，进行加密处理后和数据库中的加密密码比较。
     *
     * @param jwtToken 被校验的 jwt Token
     */
    public boolean isVerify(String jwtToken) {
        Algorithm algorithm = null;

        switch (signatureAlgorithm) {
            case HS256:
                algorithm = Algorithm.HMAC256(Base64.decodeBase64(base64EncodedSecretKey));
                break;
            default:
                throw new RuntimeException("不支持该算法");
        }

        JWTVerifier verifier = JWT.require(algorithm).build();
        verifier.verify(jwtToken);  // 校验不通过会抛出异常


        /*
            // 得到DefaultJwtParser
            Claims claims = decode(jwtToken);

            if (claims.get("password").equals(user.get("password"))) {
                return true;
            }
        */

        return true;
    }

    public static void main(String[] args) {
        JwtUtil util = new JwtUtil("tom", SignatureAlgorithm.HS256);

        Map<String, Object> map = new HashMap<>();
        map.put("username", "tom");
        map.put("password", "123456");
        map.put("age", 20);

        String jwtToken = util.encode("tom", 30000, map);

        System.out.println(jwtToken);
        /*
        util.isVerify(jwtToken);
        System.out.println("合法");
        */

        util.decode(jwtToken).entrySet().forEach((entry) -> {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        });
    }
}
```

------

## JWT Token 使用技巧

1. 可以将用户的 Http 请求的 UserAgent 信息也放入 JWT 荷载部分。

> 这样做的好处是在一定程度上 JWT Token 被盗用。一旦在另一个客户端使用同一个 token 发出 http 请求，http 请求的 UserAgent （大概率）会和前一个客户端不一样。

2. 服务端不需要考虑客户端将收到的 JWT Token 放哪。

> Cookie 还是 local storage、session storage ？这是客户端（前端开发工程师、IOS/Andriod 客户端开发工程师）考虑的事情。

3. 前端向后端传递 JWT token 是，可以放在 http header 里，也可以拼在 url 里，也可以放 cookie 里。



配置多个realm

1. 创建Modu

```

```

