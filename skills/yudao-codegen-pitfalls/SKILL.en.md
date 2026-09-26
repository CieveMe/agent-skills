---
name: yudao-codegen-pitfalls
description: Six traps in the yudao code generator — output paths, package names, menu conflicts, dictionaries, empty modules
---

# yudao CodeGen — the six traps

## When this applies

- Generating CRUD with the `ruoyi-vue-pro` code generator
- The target module uses the two-level structure (parent POM + `biz` child module)

## Trap 1 — output lands in the parent POM directory

**Symptom**: the newly generated classes cannot be found at compile time.
**Cause**: CodeGen defaults to `yudao-module-xxx/src/`, but the parent POM declares `<packaging>pom</packaging>` and never compiles Java.

```bash
# move the generated tree into the biz child module
mv yudao-module-xxx/src/* yudao-module-xxx-biz/src/
```

**Prevention**: point CodeGen's output path at the `biz` module *before* generating.

## Trap 2 — a hyphen in the package name

**Symptom**: `package cn.iocoder.xxx.point-goods` fails to compile.
**Cause**: the generator derives package names from table names; `point_goods` can produce a hyphen.
**Fix**: rename to `pointgoods` — no hyphens, no underscores.

## Trap 3 — duplicate `component_name` in menus

**Symptom**: a route resolves to the wrong page.
**Cause**: the new menu's `component_name` collides with an existing one (e.g. `PointRecord`).

```sql
UPDATE system_menu SET component_name = 'LotteryPointRecord'
WHERE component_name = 'PointRecord' AND path LIKE '%your-module%';
```

**Prevention**: after generating, query `system_menu` for duplicate `component_name` values.

## Trap 4 — frontend API path out of sync

**Symptom**: the frontend calls return 404.
**Cause**: the backend package/path was renamed but `api/xxx.ts` still points at the old route.
**Fix**: search the whole frontend for the old path and replace it.

## Trap 5 — dictionary data never created

**Symptom**: dropdowns are empty and status columns show no colour tag.
**Cause**: `<dict-tag>` depends on rows in `system_dict_type` + `system_dict_data`.

```sql
-- example: product status dictionary
INSERT INTO system_dict_type (name, type, status) VALUES ('Points product status', 'points_product_status', 0);
INSERT INTO system_dict_data (dict_type, label, value, color_type, sort)
VALUES ('points_product_status', 'On sale', '0', 'primary', 1),
       ('points_product_status', 'Off sale', '1', 'danger', 2);
```

Generated code happily references a dictionary that does not exist yet.

## Trap 6 — an empty module breaks the build

**Symptom**: the `xxx-api` child module has no Java sources and `maven-jar-plugin` fails.
**Fix**: add a placeholder:

```java
// src/main/java/cn/iocoder/.../module/xxx/package-info.java
/**
 * API definitions for the xxx module.
 */
package cn.iocoder.yudao.module.xxx;
```

## Post-generation checklist

- [ ] Generated files are under the `biz` module's `src/`, not the parent POM
- [ ] No hyphens in package names
- [ ] No duplicate `system_menu.component_name`
- [ ] Dictionary rows inserted
- [ ] Frontend API paths match the backend
- [ ] Empty child modules contain a `package-info.java`

*Extracted from real delivery work — all six were hit in production.*
