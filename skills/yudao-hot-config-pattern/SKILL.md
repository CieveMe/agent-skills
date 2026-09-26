---
name: 芋道框架运营级热配置模式
description: 不停服、不发版、不受活动状态限制的配置热更新模式，含后端独立端点+前端轻量弹窗+数据库JSON字段设计
---

# 芋道框架运营级热配置模式

> 来源：该项目 C8 会话（2026-04-09）
> 场景：进行中的活动需要修改规则文案、自提地址等运营参数，但原有编辑接口校验了活动状态（草稿期才可编辑）

## 设计理念

将"核心业务参数"（奖品、概率、时间）与"运营文案参数"（规则说明、地址、免责声明）在 API 和 UI 层面彻底解耦：

| 维度 | 核心参数 | 运营参数 |
|------|---------|---------|
| 修改条件 | 仅草稿期 | 任意时期 |
| API 端点 | `/update`（原有） | `/update-rule`（新增） |
| UI 入口 | 【编辑】按钮 | 【配置规则】按钮 |
| 前端组件 | ActivityForm.vue（重量级） | RuleUpdateForm.vue（轻量级） |
| 状态校验 | validateActivityIsDraft() | 仅 validateActivityExists() |

---

## 后端实现

### 1. Request VO（新增文件）

```java
// ActivityRuleUpdateReqVO.java
@Data
public class ActivityRuleUpdateReqVO {
    @NotNull(message = "活动编号不能为空")
    private Long id;
    private String ruleDesc;
}
```

### 2. Service 方法（新增方法，不改已有代码）

```java
// ActivityServiceImpl.java - 新增方法
@Override
public void updateActivityRule(ActivityRuleUpdateReqVO updateReqVO) {
    validateActivityExists(updateReqVO.getId());
    // 关键：不调用 validateActivityIsDraft()，进行中/已结束也能改
    ActivityDO updateObj = new ActivityDO();
    updateObj.setId(updateReqVO.getId());
    updateObj.setRuleDesc(updateReqVO.getRuleDesc());
    activityMapper.updateById(updateObj);
}
```

### 3. Controller 端点（新增端点）

```java
@PutMapping("/update-rule")
@Operation(summary = "更新活动规则（独立）")
@PreAuthorize("@ss.hasPermission('lottery:activity:update')") // 复用已有权限
public CommonResult<Boolean> updateActivityRule(
        @Valid @RequestBody ActivityRuleUpdateReqVO updateReqVO) {
    activityService.updateActivityRule(updateReqVO);
    return success(true);
}
```

---

## 前端实现

### 4. API 封装

```typescript
// api/lottery/activity/index.ts
export const updateActivityRule = async (data: any) => {
  return await request.put({ url: '/lottery/activity/update-rule', data })
}
```

### 5. 轻量弹窗组件 RuleUpdateForm.vue

核心结构（118行）：
```vue
<template>
  <Dialog v-model="dialogVisible" title="配置活动规则文本" width="650px">
    <el-alert type="warning" :closable="false"
      description="以下内容将展示在小程序「活动规则」页面。不论活动是否已经开始，都可以随时修改并实时生效。" />
    <el-form ref="formRef" :model="formData" label-width="90px">
      <el-form-item label="自提地址"> <el-input v-model="formData.pickupAddress" /> </el-form-item>
      <el-form-item label="活动说明"> <el-input v-model="formData.activityDesc" /> </el-form-item>
      <el-form-item label="中奖说明"> <el-input v-model="formData.winRule" type="textarea" :rows="5" /> </el-form-item>
      <el-form-item label="未中奖说明"> <el-input v-model="formData.loseRule" /> </el-form-item>
      <el-form-item label="发票说明"> <el-input v-model="formData.invoiceRule" /> </el-form-item>
      <el-form-item label="代金券规则"> <el-input v-model="formData.voucherRule" type="textarea" :rows="8" /> </el-form-item>
      <el-form-item label="免责声明"> <el-input v-model="formData.disclaimer" /> </el-form-item>
    </el-form>
    <template #footer>
      <el-button type="primary" @click="submitForm">保存</el-button>
      <el-button @click="dialogVisible = false">取消</el-button>
    </template>
  </Dialog>
</template>
```

关键逻辑：`open(row)` 时解析 `row.ruleDesc`（JSON 字符串）到表单对象，`submitForm()` 时序列化回 JSON 提交。

### 6. 列表页集成（index.vue 改动 4 行）

```vue
<!-- 操作列新增按钮 -->
<el-button link type="warning" @click="openRuleForm(scope.row)">配置规则</el-button>

<!-- 组件引入 -->
<RuleUpdateForm ref="ruleFormRef" @success="getList" />
```

---

## 数据库设计

`rule_desc` 字段为 `TEXT` 类型，存储 JSON：

```json
{
  "pickupAddress": "中奖用户可到指定自提点领取实物奖品",
  "activityDesc": "本活动由商家正规发起。",
  "winRule": "1. 抽奖结果在中奖后即时公布\n2. ...",
  "loseRule": "未中奖用户将获得平台积分奖励",
  "invoiceRule": "如需开具发票，请通过客服渠道联系商家。",
  "voucherRule": "本次活动奖品包含不同额度的代金券...",
  "disclaimer": "本活动为正常商业促销活动。"
}
```

小程序端通过 `JSON.parse(activity.ruleDesc)` 动态渲染，后端修改即刻生效。

---

## 复用模式提取

此模式可应用于任何"核心参数受状态保护，但运营参数需要随时修改"的场景：

1. 新增一个只包含可修改字段的 `XxxUpdateReqVO`
2. Service 层新增方法，**跳过**状态校验
3. Controller 新增端点，**复用**已有权限
4. 前端新增轻量弹窗，**仅展示**可修改字段
5. 列表页操作列加一个按钮 + `ref` 引用

**变更范围**：1 个新 VO + 1 个新方法 + 1 个新端点 + 1 个新组件 + 列表页 4 行改动 = **零侵入**。
