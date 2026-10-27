# Changelog

## 1.2.0 — 2026-10-27

- Added header theme selector: Auto / Light / Dark, with compact icon-only buttons on small screens.
- Auto mode follows system color scheme, including live changes. Manual preference is persisted in localStorage when available.
- Added an accessible `aria-pressed` state and matching browser theme-color metadata.
- Added dedicated coral-accented dark palette for page background, typography, navigation cards, hover states, and footer.
- Added tightly scoped CSP SHA-256 authorization for the inline initial theme script.
- Simplified the Telegram/X/Facebook social preview: removed the extra outer frame and white panel around the right-hand icon.
- Kept the original DocOfCard wordmark, square browser favicon, user-edited copy, navigation URLs, and foldable screen CSS.
- Expanded tests and exported reference screenshots in light/dark desktop/mobile sizes.

## 1.1.1 — 2026-10-27

- Separated original wordmark logo.svg from favicon.svg.
- Added static 1200 x 630 Open Graph social image, responsive layout and Cloudflare Worker distribution.
