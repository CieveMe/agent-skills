---
name: wechat-pay-v3-integration
description: End-to-end WeChat Pay V3 (JSAPI) integration in the yudao/ruoyi-vue-pro stack — certificate deployment, callback verification, and a zero-tolerance rule about mock payment code
---

# WeChat Pay V3 integration (yudao stack)

## When this applies

- `ruoyi-vue-pro` / yudao backend + WeChat mini-program payments (JSAPI mode)
- Using the `weixin-java-pay` (WxJava) library

## 1. Dependency

yudao already pulls in the WxJava family; no extra dependency is usually needed. If it is missing:

```xml
<dependency>
    <groupId>com.github.binarywang</groupId>
    <artifactId>weixin-java-pay</artifactId>
</dependency>
```

## 2. Certificates

```
<project>/yudao-server/src/main/resources/cert/
├── apiclient_key.pem       # merchant private key
├── apiclient_cert.pem      # merchant certificate
└── pub_key.pem             # WeChat platform public key (V3)

# read the certificate serial number
openssl x509 -in apiclient_cert.pem -noout -serial
```

## 3. Configuration — prefer `@Value` over `@ConfigurationProperties`

```yaml
yudao.pay.wechat:
  app-id: wx_your_app_id
  mch-id: your_mch_id
  api-v3-key: your_v3_key
  cert-serial-no: YOUR_CERT_SERIAL
  private-key-path: classpath:cert/apiclient_key.pem
  private-cert-path: classpath:cert/apiclient_cert.pem
  public-key-id: "PUB_KEY_ID_xxx"
  public-key-path: classpath:cert/pub_key.pem
  notify-url: https://your-domain.com/app-api/xxx/pay-notify
```

> **Why `@Value` and not `@ConfigurationProperties`**: yudao's YAML is deeply nested, and a failed `@ConfigurationProperties` binding is **completely silent** — every field just becomes `null`. With `@Value`, a missing key fails at startup instead of at the moment a customer tries to pay.

## 4. `WxPayConfiguration`

```java
@Configuration
public class WxPayConfiguration {
    @Value("${yudao.pay.wechat.app-id}") private String appId;
    @Value("${yudao.pay.wechat.mch-id}") private String mchId;
    @Value("${yudao.pay.wechat.api-v3-key}") private String apiV3Key;
    // ... remaining fields

    @Bean
    public WxPayService wxPayService() throws Exception {
        WxPayConfig config = new WxPayConfig();
        config.setAppId(appId);
        config.setMchId(mchId);
        config.setApiV3Key(apiV3Key);
        config.setPrivateKeyPath(privateKeyPath);
        config.setPrivateCertPath(privateCertPath);
        config.setCertSerialNo(certSerialNo);
        // ... public key configuration
        WxPayService service = new WxPayServiceImpl();
        service.setConfig(config);
        return service;
    }
}
```

## 5. Placing an order (JSAPI)

```java
public WxPayUnifiedOrderV3Result createPayOrder(Long userId, String code, Long activityId) {
    // 1. openid
    String openId = socialUserApi.getSocialUserByUserId(userType, userId, socialType).getOpenid();

    // 2. request
    WxPayUnifiedOrderV3Request request = new WxPayUnifiedOrderV3Request();
    request.setOutTradeNo(orderNo);
    request.setDescription("campaign entry");
    request.setNotifyUrl(notifyUrl);
    request.setAmount(new WxPayUnifiedOrderV3Request.Amount().setTotal(amountInFen));
    request.setPayer(new WxPayUnifiedOrderV3Request.Payer().setOpenid(openId));

    // 3. create
    return wxPayService.createOrderV3(TradeTypeEnum.JSAPI, request);
}
```

## 6. Callback verification

```java
@PostMapping("/pay-notify")
public String handlePayNotify(HttpServletRequest request) {
    WxPayNotifyV3Result result = wxPayService.parseOrderNotifyV3Result(
        request.getHeader("..."),   // signature headers
        requestBody
    );
    // business logic — must be idempotent: WeChat retries this callback
    return "<xml><return_code>SUCCESS</return_code></xml>";
}
```

> `WxPayNotifyV3Result` (callback) is a **different type** from `WxPayUnifiedOrderV3Result` (order creation). Mixing them up compiles fine and fails in production.

## 7. Zero tolerance for mock payment code ⭐

```
Never: return fabricated data from a payment catch block
Never: comment out mock code (the agent will restore it)
Always: physically delete mock code
Always: force injection (@Resource) so a missing bean fails at startup
```

**Why this is a rule and not a preference**: in AI-assisted development, mock fallbacks are a time bomb. Every model switch, every compile error, every over-long context is an opportunity for the agent to "helpfully" restore the mock path. This project rolled back to real payments **three times** before the rule was written down.

## 8. Security configuration

```yaml
# the callback must be anonymous: WeChat calls it without a token
yudao.security.permit-all_urls:
  - /app-api/xxx/pay-notify

# mock enable must be false
yudao.security.mock-enable: false
```

## 9. Pre-flight checklist

- [ ] Certificate files present under `resources/cert/`
- [ ] `appId` / `mchId` / `apiV3Key` / `certSerialNo` correct in YAML
- [ ] Injected with `@Value` (not `@ConfigurationProperties`)
- [ ] `mock-enable: false`
- [ ] Callback URL listed in `permit-all_urls`
- [ ] Zero mock/fallback code left anywhere
- [ ] Callback uses `WxPayNotifyV3Result` (not the order-creation result type)
- [ ] Callback handler is idempotent

*Extracted from a production mini-program delivery, after three mock-code rollbacks.*
