<p align="center"><img src="assets/banner.png" alt="Mohami Plus Web" width="100%"></p>
<h1 align="center">Mohami Plus — Web</h1>
<p align="center"><b>A bilingual marketing site plus a full Flutter web platform with a desktop shell, built from the same codebase as the mobile app.</b></p>
<p align="center">
  <img src="https://img.shields.io/badge/Platform-Web%20(desktop%20%2B%20mobile)-13284A?style=flat-square" alt="Platform">
  <img src="https://img.shields.io/badge/Flutter-Web-02569B?style=flat-square&logo=flutter&logoColor=white" alt="Flutter Web">
  <img src="https://img.shields.io/badge/Landing-HTML%20%2F%20CSS%20%2F%20JS-E34F26?style=flat-square&logo=html5&logoColor=white" alt="HTML CSS JS">
  <img src="https://img.shields.io/badge/i18n-Arabic%20RTL%20%7C%20English%20LTR-0F766E?style=flat-square" alt="i18n">
  <img src="https://img.shields.io/badge/Theme-Light%20%7C%20Dark-D4AF37?style=flat-square" alt="Themes">
  <img src="https://img.shields.io/badge/Live-mohami.plus-111827?style=flat-square" alt="Live">
  <img src="https://img.shields.io/badge/Case%20Study-Web-111827?style=flat-square" alt="Case Study">
</p>

> ℹ️ This is a case study. The source code belongs to the client and is not included.

## Overview

[Mohami Plus](https://github.com/ANaSHaker/mohami-plus-app-case-study) is an AI legal assistant for Iraq. This case study covers its two web surfaces, both of which I built in October 2026:

1. **The landing site** at [mohami.plus](https://mohami.plus). It is hand-written HTML, CSS and vanilla JS with no framework, in Arabic and English, with light and dark themes. It reads live content counters from the API.
2. **The web platform** at [mohami.plus/home](https://mohami.plus/home/). This is the Flutter app compiled for the web. It wraps every mobile screen in a desktop shell with a sidebar, a top bar and a command search. Visitors can browse public content as guests.

Both share the brand, the copy and the backend with the mobile app.

## The challenge

- **One codebase, two form factors.** The Flutter app was designed for phones. On a 1440 px monitor it had to feel like a real desktop product, not a stretched phone, without forking 70 screens.
- **Visitors who don't have an account.** On mobile the app asks you to sign in first. On the web, people arrive from search and links, so they need to read content first and be asked to sign in only when they act.
- **Deep links on a static host.** The web app lives under `/home/` on shared hosting. Short links such as `/home/faq` and `/home/privacy` had to work, and so did refreshes on any route.
- **A landing page that never lies.** The counters on the page (AI services, contracts, courts, dictionary terms) had to come from the real database, not be hard-coded.

## Highlights

**🖥️ Flutter web platform**
- A **desktop shell** that wraps the whole `Navigator`. It has a collapsible sidebar, a top bar with search (Ctrl/⌘ K), a language toggle, a theme toggle and notifications. Content sits in a floating card at most 1280 px wide.
- Below 1000 px the shell steps aside, and the user gets the exact mobile UI.
- **Guest mode.** Public screens open freely. Protected routes are swapped for a "sign in to continue" sheet, both in `onGenerateRoute` and in the network interceptor.
- A split-screen sign-in page and an app-download card in the sidebar.
- Path URLs, browser back and forward support, and short aliases for the landing footer: `/faq`, `/privacy`, `/terms`.

**🌐 Landing site**
- Arabic RTL and English LTR from a small `i18n.js` dictionary. Light and dark themes, with the user's choice remembered.
- Live counters from the public API, fetched with an abort timeout and a graceful fallback.
- Sections: features, audience (citizens, lawyers, companies), how it works, the AI assistant, app screens in light and dark, FAQ, download and contact.
- In-page phone mockups drawn in HTML and CSS (`screens.js`), so the hero stays sharp at any size.
- SEO basics: a meta description, `sitemap.xml`, `robots.txt` and a web manifest.

## Screens

<p align="center"><img src="assets/web-landing.png" alt="Landing site" width="100%"></p>
<p align="center"><sub>Landing site, desktop and phone.</sub></p>

<p align="center"><img src="assets/web-landing-sections.png" alt="Landing sections" width="100%"></p>

<p align="center"><img src="assets/web-mobile.png" alt="Landing on a phone" width="100%"></p>

<p align="center"><img src="assets/web-platform-screens.png" alt="Flutter web screens" width="100%"></p>
<p align="center"><sub>Flutter web platform: emergency help, procedure guides, Iraqi legal Q&amp;A and the split-screen sign-in.</sub></p>

> All images are real captures of the site and the Flutter web build, rendered with headless Chrome at 1440 × 900 (desktop) and 390 × 844 (phone).

## Signature feature: one Flutter app, a real desktop shell

<p align="center"><img src="assets/web-platform.png" alt="Flutter web desktop shell" width="100%"></p>

The shell is installed once, in `MaterialApp.builder`. Every route the app already had renders inside its content area, so no screen was rewritten for the web:

- The content card gets its own `MediaQuery`. Screens built for phones lay out against the card's width, not the window's.
- The sidebar groups the app's sections (main, tools, account) and highlights the current route.
- Keyboard shortcuts work: Ctrl/⌘ K focuses the global search, which looks up legal terms or the library.
- On narrow windows (< 1000 px) or mobile browsers, the shell returns its child untouched, so phones get the native mobile layout.

## Architecture & stack

```mermaid
flowchart LR
  Visitor((Visitor)) --> Landing["Landing site<br/>HTML · CSS · JS<br/>ar/en · light/dark"]
  Landing -- "Open the platform" --> Shell
  Landing -- "live counters" --> API
  subgraph Web["Flutter web build at /home/ (same code as mobile)"]
    Shell["WebShell<br/>sidebar · top bar · Ctrl K"] --> Nav["Navigator + AppRouter"]
    Nav --> Gate{"GuestGate"}
    Gate -- public --> Screens["Shared screens"]
    Gate -- protected --> Sheet["Sign-in sheet"]
  end
  Screens --> API["Laravel REST API"]
  Host["Static hosting<br/>SPA rewrite to index.html"] -.serves.-> Web
```

| Layer | Technology |
|---|---|
| Landing | Semantic HTML5, a single CSS file (about 880 lines), vanilla JS (`main.js`, `i18n.js`, `screens.js`) |
| Web app | Flutter web (CanvasKit), built with `--base-href /home/`, path URL strategy |
| Web-only code | `WebShell`, `WebNav`, `WebAuthFrame`, `GuestGate`, conditional imports (`web_bridge_web.dart` / `_io.dart`) |
| Hosting | Static files with an SPA rewrite rule and forced HTTPS |
| Data | The same REST API as the mobile app; public endpoints for counters, FAQ and legal pages |

## Engineering notes

- **Conditional imports for web-only APIs.** Browser features such as file download and opening URLs sit behind a `web_bridge` facade with `_web` and `_io` implementations. The mobile build never pulls in `dart:html`.
- **A deep-link trap, documented.** The SPA rewrite file is copied into every build. Once, a stale copy pointed at an old base path and broke deep links. The base path now lives in one place and is checked on every deploy.
- **Stale caches after a rename.** Translation files moved from `assets/languages/` to `assets/i18n/` to bust aggressive browser caches after a big i18n update. Users got fresh strings without hard-refreshing.
- **Honest numbers.** The landing page only shows counters returned by the API. If the request fails or times out, the section degrades quietly instead of showing invented numbers.
- **Light by default, dark on request.** Every surface (landing, web app, dashboard) starts in light mode and remembers the user's choice. The dark palette is derived from the same brand tokens.

## Results

- Both surfaces are live: [mohami.plus](https://mohami.plus) (landing) and [mohami.plus/home](https://mohami.plus/home/) (web platform).
- At the time of writing, the landing page showed **18 AI services, 28 contract templates, 59 courts and 233 dictionary terms**, all read live from the API.

## بالعربي

واجهتا الويب لمنصة **محامي بلس**: موقع تعريفي بالعربية والإنجليزية بوضعين فاتح وداكن، وأرقامه تُقرأ مباشرة من قاعدة البيانات.
ومنصة ويب كاملة مبنية من كود تطبيق Flutter نفسه، بواجهة سطح مكتب فيها قائمة جانبية وشريط علوي وبحث سريع (Ctrl K).
يتصفح الزائر المحتوى العام بدون حساب، ويُطلب منه تسجيل الدخول فقط عند تنفيذ إجراء.
روابط مباشرة لكل صفحة، ودعم كامل لزر الرجوع في المتصفح، والعربية والإنجليزية.

## Links

- 🌐 Landing: [mohami.plus](https://mohami.plus)
- 🖥️ Web platform: [mohami.plus/home](https://mohami.plus/home/)
- 📱 Mobile app case study: [mohami-plus-app-case-study](https://github.com/ANaSHaker/mohami-plus-app-case-study)

---
<h3 align="center">Built by Mahmoud Shaker — Senior Flutter Engineer</h3>
<p align="center">
  <a href="https://wa.me/201140279111"><img src="https://img.shields.io/badge/WhatsApp-Chat-25D366?style=for-the-badge&logo=whatsapp&logoColor=white" alt="WhatsApp"></a>
  <a href="https://www.linkedin.com/in/anashaker/"><img src="https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn"></a>
  <a href="https://anashaker.github.io/shaker_portfolio/"><img src="https://img.shields.io/badge/Portfolio-Visit-111827?style=for-the-badge&logo=googlechrome&logoColor=white" alt="Portfolio"></a>
  <a href="mailto:mahmoudshaker382001@gmail.com"><img src="https://img.shields.io/badge/Email-Hire%20me-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"></a>
  <a href="https://khamsat.com/user/sheko201566"><img src="https://img.shields.io/badge/Khamsat-Services-F39C12?style=for-the-badge" alt="Khamsat"></a>
  <a href="https://mostaql.com/u/ShekoShaker"><img src="https://img.shields.io/badge/Mostaql-Freelance-2386C8?style=for-the-badge" alt="Mostaql"></a>
  <a href="https://github.com/ANaSHaker"><img src="https://img.shields.io/badge/GitHub-More%20work-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub"></a>
</p>
