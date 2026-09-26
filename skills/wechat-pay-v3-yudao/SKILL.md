---
name: 微信支付V3芋道集成
description: 在芋道框架中集成微信支付V3(JSAPI)的完整流程，含证书部署、回调验签、Mock零容忍策略
---

# 微信支付 V3 + 芋道集成指南

## 适用场景
- 芋道 ruoyi-vue-pro + 微信小程序支付（JSAPI 模式）
- 使用 weixin-java-pay (WxJava) 库

## 1. 依赖（芋道已内置）

芋道底层已依赖 WxJava 全家桶，无需额外引入。若缺失：
```xml
<dependency>
    <groupId>com.github.binarywang</groupId>
    <artifactId>weixin-java-pay</artifactId>
</dependency>
```

## 2. 证书部署

```
项目路径/yudao-server/src/main/resources/cert/
├── apiclient_key.pem       # 商户私钥
├── apiclient_cert.pem      # 商户证书
└── pub_key.pem             # 微信平台公钥（V3新增）

# 提取证书序列号
openssl x509 -in apiclient_cert.pem -noout -serial
```

## 3. YAML 配置（推荐 @Value 而非 @ConfigurationProperties）

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

> ⚠️ 为什么用 `@Value` 而非 `@ConfigurationProperties`：芋道 YAML 嵌套极深，`@ConfigurationProperties` 绑定失败时**完全静默**不报错。`@Value` 缺失时启动立即 fail-fast。

## 4. WxPayConfiguration

```java
@Configuration
public class WxPayConfiguration {
    @Value("${yudao.pay.wechat.app-id}") private String appId;
    @Value("${yudao.pay.wechat.mch-id}") private String mchId;
    @Value("${yudao.pay.wechat.api-v3-key}") private String apiV3Key;
    // ... 其他字段

    @Bean
    public WxPayService wxPayService() throws Exception {
        WxPayConfig config = new WxPayConfig();
        config.setAppId(appId);
        config.setMchId(mchId);
        config.setApiV3Key(apiV3Key);
        config.setPrivateKeyPath(privateKeyPath);
        config.setPrivateCertPath(privateCertPath);
        config.setCertSerialNo(certSerialNo);
        // ... 公钥配置
        WxPayService service = new WxPayServiceImpl();
        service.setConfig(config);
        return service;
    }
}
```

## 5. 下单（JSAPI）

```java
public WxPayUnifiedOrderV3Result createPayOrder(Long userId, String code, Long activityId) {
    // 1. 获取 OpenID
    String openId = socialUserApi.getSocialUserByUserId(userType, userId, socialType).getOpenid();
    
    // 2. 构建请求
    WxPayUnifiedOrderV3Request request = new WxPayUnifiedOrderV3Request();
    request.setOutTradeNo(orderNo);
    request.setDescription("抽奖活动");
    request.setNotifyUrl(notifyUrl);
    request.setAmount(new WxPayUnifiedOrderV3Request.Amount().setTotal(amountInFen));
    request.setPayer(new WxPayUnifiedOrderV3Request.Payer().setOpenid(openId));
    
    // 3. 下单
    return wxPayService.createOrderV3(TradeTypeEnum.JSAPI, request);
}
```

## 6. 回调验签

```java
@PostMapping("/pay-notify")
public String handlePayNotify(HttpServletRequest request) {
    WxPayNotifyV3Result result = wxPayService.parseOrderNotifyV3Result(
        request.getHeader("xxx"), // 签名相关 header
        requestBody
    );
    // 处理业务逻辑
    return "<xml><return_code>SUCCESS</return_code></xml>";
}
```

> ⚠️ `WxPayNotifyV3Result`（回调） ≠ `WxPayUnifiedOrderV3Result`（下单），别混淆！

## 7. Mock 零容忍黄金规则 ⭐

```
❌ 永远不要：在支付 catch 块中返回假数据
❌ 永远不要：注释掉 Mock 代码（AI 会恢复）
✅ 必须做：物理删除所有 Mock 代码
✅ 必须做：@Resource 强制注入，null 直接抛异常
```

**教训**：AI 辅助开发中 Mock 回退是"定时炸弹"——每次模型切换都可能触发 AI "好心"恢复 Mock。本项目反复回退 3 次后确立此规则。

## 8. 安全配置

```yaml
# 回调接口必须匿名放行（微信服务器调用，无 token）
yudao.security.permit-all_urls:
  - /app-api/xxx/pay-notify

# mock-enable 必须为 false
yudao.security.mock-enable: false
```

## 9. 检查清单

- [ ] 证书文件在 `resources/cert/` 下
- [ ] YAML 配置 appId/mchId/apiV3Key/certSerialNo 正确
- [ ] `@Value` 注入（非 `@ConfigurationProperties`）
- [ ] `mock-enable: false`
- [ ] 回调 URL 在 `permit-all_urls` 中
- [ ] 无任何 Mock/fallback 代码残留
- [ ] 回调用 `WxPayNotifyV3Result`（非 UnifiedOrderV3Result）

## 来源
提炼自一个生产级抽奖营销小程序项目 C3 会话，经 3 次 Mock 回退教训后沉淀。
