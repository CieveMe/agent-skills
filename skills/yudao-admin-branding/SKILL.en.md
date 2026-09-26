---
name: yudao-admin-branding
description: White-labelling the yudao-ui-admin-vue3 console before handover — trimming the login page, emptying the dashboard, replacing logos and titles
---

# White-labelling the yudao admin console

## When this applies

- Delivering `yudao-ui-admin-vue3` to a client and removing the framework's own branding
- The client does not need the framework's default marketing-style dashboard

## 1. Titles

```env
# .env
VITE_APP_TITLE=Example Company
```
```html
<!-- index.html -->
<title>Example Company</title>
```

## 2. Logos — three files

```
src/assets/imgs/logo.png      → sidebar
src/assets/imgs/logo.gif      → login page
src/assets/imgs/avatar.gif    → default avatar
```

> **Lesson**: when the client hands you a JPG, just copy it over the PNG/GIF filenames — browsers render by content, not by extension. Do not spend iterations asking an AI to redesign a logo; in one project seven attempts were worse than the client's original asset.

## 3. Trimming the login page

Keep:
```
tenant name · username · password · login button
```
Remove:
```
mobile-number login tab · QR login · captcha · "remember me"
third-party login icons · "no account? register"
top-right language/theme switcher · decorative footer image
```

## 4. Emptying the dashboard

`Home/Index.vue` — remove the ECharts widgets, project notices, shortcuts, GitHub link and contributor list; keep a single line:

```
"Welcome to the Example Company management console"
```

A dashboard full of framework statistics is worse than no dashboard: the client will ask what the numbers mean.

## 5. Removing dangerous buttons from business pages

```vue
❌ export button (the client does not need it)
❌ bulk delete (mis-click risk in a small team)
❌ reset button (clearing a search box is enough)
```

## 6. Build and deploy

```bash
npm run build:prod            # build locally: the server usually lacks RAM
tar -czf admin-dist.tar.gz dist/
scp admin-dist.tar.gz user@server:/opt/app/
```

## Checklist

- [ ] `VITE_APP_TITLE` changed in `.env`
- [ ] `<title>` changed in `index.html`
- [ ] All three logo files replaced
- [ ] Login page shows only tenant + username + password + button
- [ ] Dashboard shows only the welcome line
- [ ] No export / bulk-delete buttons left in the delivered modules

*Extracted from real delivery work (including seven logo iterations and three login-page trims).*
