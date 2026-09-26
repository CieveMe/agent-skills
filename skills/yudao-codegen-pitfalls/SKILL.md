---
name: 芋道CodeGen避坑指南
description: 芋道框架代码生成器的6个常见坑及解决方案，涵盖路径、包名、菜单、字典等问题
---

# 芋道 CodeGen 避坑指南

## 适用场景
- 使用芋道 ruoyi-vue-pro 的代码生成器（CodeGen）生成 CRUD 代码
- 目标模块采用双层结构（父 POM + biz 子模块）

## 坑 1：生成代码输出到父 POM 目录

**现象**: 编译找不到新生成的类  
**原因**: CodeGen 默认输出到 `yudao-module-xxx/src/`，但父 POM 是 `<packaging>pom`，不编译 Java  
**解决**:
```bash
# 生成后批量移动到 biz 子模块
mv yudao-module-xxx/src/* yudao-module-xxx-biz/src/
```
**预防**: 生成前在 CodeGen 配置中指定输出路径为 biz 子模块

## 坑 2：包名含连字符

**现象**: `package cn.iocoder.xxx.point-goods` 编译语法错误  
**原因**: CodeGen 按表名 `point_goods` 生成包名时可能产生连字符  
**解决**: 手动改为 `pointgoods`（无连字符无下划线）

## 坑 3：菜单 component_name 冲突

**现象**: 页面路由到错误组件  
**原因**: 新模块菜单的 `component_name` 与系统已有菜单重名（如 `PointRecord`）  
**解决**:
```sql
UPDATE system_menu SET component_name = 'LotteryPointRecord'
WHERE component_name = 'PointRecord' AND path LIKE '%your-module%';
```
**预防**: 生成后检查 `system_menu` 表是否有重名

## 坑 4：前端 API 路径与后端不一致

**现象**: 前端调用 404  
**原因**: 改了后端包名/路径后，前端 `api/xxx.ts` 的请求路径未同步更新  
**解决**: 全局搜索旧路径，批量替换

## 坑 5：字典数据未配置

**现象**: 管理后台下拉框为空、状态列不显示颜色标签  
**原因**: `<dict-tag>` 组件依赖 `system_dict_type` + `system_dict_data` 表数据  
**解决**:
```sql
-- 示例：积分商品状态字典
INSERT INTO system_dict_type (name, type, status) VALUES ('积分商品状态', 'lottery_point_status', 0);
INSERT INTO system_dict_data (dict_type, label, value, color_type, sort)
VALUES ('lottery_point_status', '上架', '0', 'primary', 1),
       ('lottery_point_status', '下架', '1', 'danger', 2);
```

## 坑 6：空模块打包失败

**现象**: `lottery-api` 子模块无 Java 源文件，`maven-jar-plugin` 报错  
**解决**: 创建占位文件
```java
// src/main/java/cn/iocoder/.../lottery/package-info.java
/**
 * 抽奖模块 API 定义
 */
package cn.iocoder.yudao.module.lottery;
```

## CodeGen 后必做检查清单

- [ ] 生成文件在 `biz` 子模块的 `src/` 下（非父 POM）
- [ ] 包名无连字符
- [ ] `system_menu.component_name` 无重名
- [ ] 字典数据已插入
- [ ] 前端 API 请求路径与后端一致
- [ ] 空子模块有 `package-info.java`

## 来源
提炼自一个生产级抽奖营销小程序项目 C2 会话，6 个坑全部实际踩过。
