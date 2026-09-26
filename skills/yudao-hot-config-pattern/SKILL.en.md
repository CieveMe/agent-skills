---
name: hot-configuration-pattern
description: Change operational parameters without a redeploy, a release, or waiting for a status change — an independent endpoint, a lightweight modal and a JSON column
---

# Operational hot-configuration pattern (yudao stack)

> Source: real delivery session (2026-04-09)
> Problem: a **running** campaign needed its rule text and pickup address changed, but the existing update endpoint validated campaign status (draft-only editing).

## The idea

Split *core business parameters* (prizes, probability, schedule) from *operational content* (rule text, addresses, disclaimers) at both the API and UI level:

| Dimension | Core parameters | Operational parameters |
|---|---|---|
| Editable when | draft only | any time |
| API endpoint | `/update` (existing) | `/update-rule` (new) |
| UI entry point | "Edit" button | "Configure rules" button |
| Frontend component | `ActivityForm.vue` (heavy) | `RuleUpdateForm.vue` (light) |
| Status validation | `validateActivityIsDraft()` | `validateActivityExists()` only |

The point is not to weaken validation. A campaign's probability table *should* be frozen while it runs. Its disclaimer text should not require a release.

## Backend

**1. A request VO that only contains mutable fields**

```java
@Data
public class ActivityRuleUpdateReqVO {
    @NotNull(message = "activity id is required")
    private Long id;
    private String ruleDesc;
}
```

**2. A new service method — existing code untouched**

```java
@Override
public void updateActivityRule(ActivityRuleUpdateReqVO updateReqVO) {
    validateActivityExists(updateReqVO.getId());
    // note: no validateActivityIsDraft() — running and finished campaigns can still be edited
    ActivityDO updateObj = new ActivityDO();
    updateObj.setId(updateReqVO.getId());
    updateObj.setRuleDesc(updateReqVO.getRuleDesc());
    activityMapper.updateById(updateObj);
}
```

**3. A new endpoint that reuses the existing permission**

```java
@PutMapping("/update-rule")
@PreAuthorize("@ss.hasPermission('lottery:activity:update')")
public CommonResult<Boolean> updateActivityRule(
        @Valid @RequestBody ActivityRuleUpdateReqVO updateReqVO) {
    activityService.updateActivityRule(updateReqVO);
    return success(true);
}
```

## Frontend

**4. API wrapper**

```typescript
export const updateActivityRule = async (data: any) => {
  return await request.put({ url: '/lottery/activity/update-rule', data })
}
```

**5. A lightweight modal (`RuleUpdateForm.vue`)**

```vue
<template>
  <Dialog v-model="dialogVisible" title="Campaign rule text" width="650px">
    <el-alert type="warning" :closable="false"
      description="Shown on the mini-program's rules page. Editable at any time, effective immediately." />
    <el-form ref="formRef" :model="formData" label-width="90px">
      <el-form-item label="Pickup address"><el-input v-model="formData.pickupAddress" /></el-form-item>
      <el-form-item label="Campaign notes"><el-input v-model="formData.activityDesc" /></el-form-item>
      <el-form-item label="Winning terms"><el-input v-model="formData.winRule" type="textarea" :rows="5" /></el-form-item>
      <el-form-item label="Losing terms"><el-input v-model="formData.loseRule" /></el-form-item>
      <el-form-item label="Invoice notes"><el-input v-model="formData.invoiceRule" /></el-form-item>
      <el-form-item label="Voucher terms"><el-input v-model="formData.voucherRule" type="textarea" :rows="8" /></el-form-item>
      <el-form-item label="Disclaimer"><el-input v-model="formData.disclaimer" /></el-form-item>
    </el-form>
    <template #footer>
      <el-button type="primary" @click="submitForm">Save</el-button>
      <el-button @click="dialogVisible = false">Cancel</el-button>
    </template>
  </Dialog>
</template>
```

`open(row)` parses `row.ruleDesc` (a JSON string) into the form model; `submitForm()` serialises it back and PUTs it.

**6. Four lines in the list page**

```vue
<el-button link type="warning" @click="openRuleForm(scope.row)">Configure rules</el-button>
<RuleUpdateForm ref="ruleFormRef" @success="getList" />
```

## Data model

`rule_desc` is a `TEXT` column holding JSON:

```json
{
  "pickupAddress": "Winner pickup is available at the downtown pickup point.",
  "activityDesc": "This campaign is run directly by the merchant.",
  "winRule": "1. Results are announced immediately after the draw\n2. ...",
  "loseRule": "Participants who do not win receive platform points.",
  "invoiceRule": "For an invoice, contact the merchant through support.",
  "voucherRule": "Prizes include vouchers of several denominations...",
  "disclaimer": "This is a standard commercial promotion."
}
```

The client parses it with `JSON.parse(activity.ruleDesc)` and renders it; any backend edit is live immediately.

## Reusing the pattern

It applies to any case where *core* parameters are status-protected but *operational* content must stay editable:

1. Add an `XxxUpdateReqVO` containing only the mutable fields.
2. Add a service method that **skips** the status check.
3. Add a controller endpoint that **reuses** the existing permission.
4. Add a lightweight modal that **only** exposes those fields.
5. Add one button and one `ref` to the list page.

**Blast radius**: one VO + one method + one endpoint + one component + four lines = **zero intrusion** into existing code paths.

*Extracted from real delivery work and sanitised (the original example address and campaign text were replaced).*
