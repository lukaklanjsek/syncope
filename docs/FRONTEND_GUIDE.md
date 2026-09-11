# Frontend guide: customizing Syncope's look

## 1. Overview

This is the reference for changing how the app *looks* — colors, fonts, buttons, spacing,
nav branding — without touching Python code. There's no build step: one CSS file
(`syncope/static/syncope/style.css`), self-hosted fonts, and Django templates. Edit,
save, reload the page.

The whole restyle is built around one idea: **almost every visual decision lives in a
handful of CSS custom properties (design tokens) at the top of `style.css`.** If you want
to reskin the app for your own group — different palette, different fonts, different
button shape — you should be able to do it by editing that one `:root` block and nothing
else. Sections below tell you exactly where to look for anything not covered by tokens
alone (mainly: swapping the fonts themselves, and the sidebar logo).

> Status: this guide is being written alongside the CSS overhaul itself (2026-09-11),
> section by section as each part of the app is restyled. Sections not yet filled in
> are marked below.

## 2. Design tokens

Everything here lives in the `:root { ... }` block at the top of
`syncope/static/syncope/style.css`. Change a value, save, reload — every place that uses
it updates.

| Token | Default | Used for |
|---|---|---|
| `--color-bg` | `#f4f1ea` | Page background |
| `--color-bg-subtle` | `#ebe6d9` | Secondary/alt section backgrounds |
| `--color-surface` | `#ffffff` | Card/panel/table background |
| `--color-border` | `#e5dfd6` | Default borders (cards, tables, inputs) |
| `--color-border-subtle` | `#efe9dd` | Lighter dividers |
| `--color-text` | `#2c2a28` | Body text |
| `--color-text-muted` | `#5c574f` | Secondary/helper text |
| `--color-text-on-accent` | `#ffffff` | Text on filled buttons/badges |
| `--color-primary` | `#7a6140` | Brand accent — primary buttons, links |
| `--color-primary-hover` | `#63502f` | Hover state for the above |
| `--color-danger` | `#c0392b` | Destructive actions (delete buttons) |
| `--color-danger-hover` | `#a3301f` | Hover state for the above |
| `--color-success-bg` / `-border` / `-text` | greens | Success flash messages |
| `--color-error-bg` / `-border` / `-text` | reds | Error flash messages |
| `--color-warning-bg` / `-border` / `-text` | ambers | Warning flash messages |
| `--color-info-bg` / `-border` / `-text` | blues | Info flash messages |
| `--color-attendance-0`…`-4` | see comment above them | Attendance chip colors, keyed by `AttendanceType` pk (0=TBD, 1=Present, 2=Work/School, 3=Illness, 4=Private/Vacation) — **don't reorder these**, the pk mapping is load-bearing |
| `--font-heading` / `--font-body` | see Fonts section | Typography |
| `--font-size-sm` / `-base` / `-lg` / `-xl` | 0.85rem / 1rem / 1.15rem / 1.5rem | Text sizing scale |
| `--space-1`…`-6` | 4px → 32px | Spacing scale used for padding/margin/gap throughout |
| `--radius-sm` / `-md` | 4px / 8px | Corner rounding (buttons/inputs use `sm`, cards/tables use `md`) |
| `--shadow-sm` / `-md` | subtle box-shadows | Card and hover elevation |
| `--transition-fast` | `0.2s ease` (`0s` when `prefers-reduced-motion: reduce`) | All hover/transition timing |
| `--sidebar-width` | `250px` | Sidebar column width in `.grid-container` |

**Recipe — change the whole color scheme:** edit the `--color-*` values. Every button,
alert, card, and table border is built from these, so a new palette propagates
everywhere automatically. Re-check contrast if you change `--color-primary` or
`--color-text-muted` against `--color-bg`/`--color-surface` — see the note on WCAG AA
below.

**Contrast note:** the shipped palette was picked so every text/background pairing hits
at least 4.5:1 contrast (WCAG AA for normal text), including button-label-on-fill and
muted/link text on both the page and card backgrounds. If you swap `--color-primary` or
the neutral colors, re-check the pairing you changed — a quick way is to plug the two hex
values into any online WCAG contrast checker.

## 3. Fonts

Self-hosted, no external font CDN (keeps the app free of a third-party network
dependency). Two variable-weight `.woff2` files live in
`syncope/static/syncope/fonts/`:

- `eb-garamond-latin.woff2` — heading font (`--font-heading`), a serif
- `inter-latin.woff2` — body font (`--font-body`), a sans-serif

Both are declared via `@font-face` right above the `:root` block in `style.css`, with
`font-display: swap` so text isn't invisible while the font loads. Each file is a
variable font covering weights 400–500 in the Latin subset only (this app doesn't need
Cyrillic/Greek/etc. glyphs, so those subsets were skipped to keep the files small).

**Recipe — swap to a different font pairing:**
1. Get `.woff2` file(s) for your chosen font(s) and drop them in
   `syncope/static/syncope/fonts/`.
2. Update the `@font-face` `src` path(s) and `font-family` name(s) at the top of
   `style.css`.
3. Point `--font-heading` / `--font-body` at the new family name(s).

No template changes needed — every heading and body element already references the
tokens, not a hardcoded font name.

## 4. Component classes

All defined in `style.css` under clearly labeled `/* --- Section --- */` comments, in this order top to bottom:

| Class | Lives under (`style.css` comment) | Use |
|---|---|---|
| `.btn`, `.btn-primary`, `.btn-secondary`, `.btn-danger`, `.btn-link`, `.btn-sm` | `/* --- Buttons --- */` | Every button/button-styled link in the app |
| `.form-actions` | same section, right after `.btn-sm` | Wraps a form's submit/cancel row — see **Forms** below for the layout rule |
| `.form-group`, `.form-label`, `.form-help`, `ul.errorlist` styling | `/* --- Forms --- */` | Field wrapper + label + help text + Django's built-in error list |
| `.card`, `.card-grid` | `/* --- Cards --- */` | Not yet used by any template this pass — ready for Phase 4/future list pages |
| `.table`, `.table-detail`, `.table-grouped` | `/* --- Tables --- */` | Data tables; `.table-grouped tbody tr.group-start` replaces the old inline group-border pattern (see Phase 4) |
| `.alert`, `.alert-success/-error/-warning/-info` | defined earlier, alongside `.messages` (Phase 2) | Flash messages — `class="alert alert-{{ message.tags }}"` |

**Recipe — change what a button looks like everywhere:** edit `.btn` (shared shape/spacing/font) or one of `.btn-primary`/`.btn-secondary`/`.btn-danger`/`.btn-link` (color only). Every button in the app uses these — there's no template with its own one-off button CSS left in the areas this pass covered.

**Button hierarchy rule (apply this, don't reinvent per page):** one `.btn-primary` per screen (the actual commit action — Save/Update/Send/etc.), everything secondary/navigational is `.btn-link` (plain, e.g. Cancel) or `.btn-secondary` (outlined, e.g. an alternate "Return to X" action), and `.btn-danger` is reserved for destructive actions only, kept on its own confirm-delete page or behind an existing `confirm()` dialog — never placed next to a primary Save button in the same `.form-actions` row.

## 5. Utility classes

- **`.inline-form`** — replaces `style="display:inline"` on the small one-button forms scattered through detail/list pages (delete/unlink forms sitting inline with links). Purely `display: inline`.
- **`.visually-hidden`** — a hard `display: none`, *not* the accessible-but-present clip technique some codebases use under this name. Only ever put it on wrapper `<div>`s that hold nothing but a CSRF token — there's no content to keep available to assistive tech there, so the stronger hide is fine and matches what those wrappers already did via inline `style`.
- **Generic `input[type="checkbox"]`/`input[type="radio"]`** spacing (not a class, an element selector) — covers Django's default `CheckboxSelectMultiple`/`RadioSelect` output (skills/roles, attendance type) without a custom widget template. Slightly less polished alignment than a bespoke layout would give; a real widget-template override is the natural next step if this needs more polish later.
- **`.form-error`** — plain `color: var(--color-error-text)`, for the couple of hand-rolled error messages that were previously inline `style="color:red"` (formset non-field errors).

## 6. Forms

**The pattern:** `{% for field in form %}` + a `.form-group` wrapper, using `{{ field.errors }}` directly (Django already renders that as `<ul class="errorlist">`, which `.form-group` already styles — don't hand-loop `field.errors` yourself). See `song_form.html` and `event_meta_edit.html` for the full pattern, including how `song_form.html` branches on `field.name` to insert its "+" add-related-person buttons inline without breaking the loop. Plain `{{ form.as_p }}` forms (e.g. `event_form.html`, `signup.html`, `login.html`) needed no template changes at all — `style.css` styles Django's default `<p>`/`<label>`/`.errorlist`/`.helptext` output directly via `form p ...` selectors, so `.form-group` and `as_p` both render consistently without picking one convention app-wide.

**Button row convention — `.form-actions`:** always a single `<div class="form-actions">` containing, in this DOM order: Cancel/Back link (`.btn-link`) → optional secondary link (`.btn-secondary`, e.g. a "Return to X" navigation) → the primary submit button (`.btn-primary`) last. Flexbox (`justify-content: flex-end`) renders that as cancel-left/primary-right on desktop, clustered to the right edge; on the existing 768px breakpoint `justify-content` switches to `space-between` so the same cancel-left/primary-right order spreads across the full width instead. This is deliberately the modern web convention (secondary-left/primary-right, e.g. Bootstrap/Material), not the older Windows-dialog "primary-left" pattern.

**Delete button placement:** a page-level `.btn-danger` "Delete" for the record being edited lives only on that record's own edit page (never on its detail/dashboard/list view), and sits in its own `<p>` placed *after* (below) the `.form-actions` row — the opposite side of the screen from the primary Save button, since `.form-actions` clusters right while a bare `<p>` sits left. See `org_form.html`/`event_meta_edit.html` for the pattern.

**Button labels are single words, app-wide** — not just inside `.form-group`/`.form-actions`. "Save", "Cancel", "Delete", "Update", "New", "Add", "Send", "Signup", "Unlink" etc. Where a one-word label alone would be ambiguous (e.g. multiple "New" buttons on one page), the disambiguating context goes in a nearby heading or a `title="..."` tooltip instead of stretching the button text — see the composer/arranger/poet/translator "+" buttons in `song_form.html`, or the "New"/"Clear" search buttons in `event_songs_edit.html`/`event_attendance_edit.html`, both of which carry a `title` attribute. Plain in-page navigation links (not buttons) — e.g. "Sign up form" on the login page — are outside this rule; it only applies to actual buttons/button-styled actions.

A "return to X" navigation action (`return_url`/`return_label` in the view context) is always a plain `<a class="btn btn-secondary">`, never a `<button onclick="window.location.href=...">` — that JS was pure navigation with nothing else attached, so the link does the same job with less code. Anywhere a button still uses `onclick`/JS in this app, it's because it genuinely does something beyond navigation (a `confirm()` guard, a formset row action, the save-bar's dirty-state tracking) — don't collapse those to links.

## 7. Navigation & branding

The sidebar (`syncope/templates/syncope/base.html`) is one `<nav class="sidebar">` with
four parts, top to bottom:

- `.sidebar-header` — logo (`<img class="sidebar-logo">`) + `.sidebar-brand` wordmark
- `.sidebar-user` — the logged-in username
- `.menu-links` — the nav list, plus the nested `.org-list`/`.org-submenu` per organization
- `.sidebar-footer` — Logout (or Sign Up/Login when logged out)

**Recipe — change the logo:** replace
`syncope/static/syncope/images/sitelogo.png` with your own image (any size — it's
displayed at a fixed 48px via `.sidebar-logo` in `style.css`), or point the `<img src>`
at a different static path.

**Recipe — change the brand name/wordmark:** edit the text inside `<p
class="sidebar-brand">` in `base.html`.

**Recipe — add a new nav item:** copy an existing `<li><a href="...">` line and give it
its own `url_in` check so it highlights when active, e.g.:

```html
<li><a href="{% url 'syncope:my_view' org.user.username %}"
   {% if url_username == org.user.username and request.resolver_match.url_name|url_in:'my_view,my_view_detail' %}class="active"{% endif %}>
   My Feature
</a></li>
```

`url_in` (from `{% load url_tags %}`, already loaded at the top of the sidebar) takes a
comma-separated list of URL names and returns true if the current page's URL name is one
of them — that's what drives the `.active` highlight class. `url_username` is set by
`syncope/middleware.py` from the URL's `username` path segment, so multi-org users only
see the active state on the org they're currently viewing.

Invitations badges (`.nav-badge`) use the same pattern: the count comes from
`pending_invitations` (personal) or `membership.pending_invitations` (per-org), both
already computed in the view context — only shown when non-zero.

## 8. Responsive behavior

Everything lives in the one existing `@media (max-width: 768px)` block in `style.css` —
there's no separate mobile stylesheet or JS-based breakpoint detection. Below 769px:

- The sidebar becomes a fixed, off-canvas drawer (`transform: translateX(-100%)`),
  toggled by `.grid-container.menu-visible` — driven by the `toggleMenu()` function in
  `base.html`, unchanged by this pass.
- `.form-actions` switches from clustering right (`justify-content: flex-end`) to
  spreading full-width (`justify-content: space-between`) — cancel stays pinned to the
  left edge, primary to the right, same as desktop, just spread out instead of clustered.
- Text inputs and `.btn-primary`/`.btn-danger` get a `min-height: 44px` touch target.
- The save-bar becomes `position: fixed` to the viewport bottom.

**Recipe — change the breakpoint:** there's one number to edit —
`@media (max-width: 768px)` near the bottom of `style.css`. Everything inside that block
applies below whatever width you set.

**Accessibility, addressed once, globally:**
- `:focus-visible` gets a visible `outline: 2px solid var(--color-primary)` on every
  `.btn`, link, and form field (one rule, near the Buttons section of `style.css`) —
  keyboard users always see where focus is, without adding a visible ring on mouse clicks.
- `@media (prefers-reduced-motion: reduce)` forces `--transition-fast` to `0s` — since
  every hover/transition in the app is built on that one token, this one override turns
  all of them off at once for users who've asked for reduced motion at the OS level.

## 9. File map

- `syncope/static/syncope/style.css` — all styling: tokens, components, utilities, responsive rules
- `syncope/static/syncope/fonts/` — self-hosted font files
- `syncope/templates/syncope/base.html` — page shell: sidebar, branding, flash messages
- `syncope/templates/syncope/_breadcrumbs.html` — breadcrumb markup (unchanged by this restyle)
- `syncope/templates/syncope/_save_bar.html` — shared draft/dirty-tracking save bar (used by `event_songs_edit.html`, `event_attendance_edit.html`)

**Pages with their own bespoke `<style>` block** (page-specific table layouts — sticky
columns, rotated date headers — that weren't worth generalizing into `style.css` this
round): `poll_detail.html`, `poll_attendance.html`, `attendance_dashboard.html`,
`event_meta_edit.html`. Each already reuses the global color/spacing tokens inside its
local block rather than hardcoding new values.

**Not covered in this pass** — inline `style="..."` attributes remain as they were,
follow-up work if picked back up later:
- The ~19 lower-traffic templates below the Phase 4 threshold (list/detail pages with
  fewer inline-style occurrences than the 6 worst offenders that were fixed:
  `project_update.html`, `poll_event.html`, `poll_person.html`, `poll_detail.html`,
  `poll_attendance.html`, `song_list_results.html`).
- The two checkbox/radio custom widget-template overrides (skills/roles,
  `AddAttendanceForm.attendance_type`) — currently styled via generic
  `input[type=checkbox]`/`input[type=radio]` selectors, not a bespoke aligned layout.
- A full WCAG contrast spreadsheet — only the highest-risk pairings (muted text on
  card/page backgrounds, button-label-on-fill) were actually checked against the shipped
  palette.
- Dark mode — the token names are semantic (`--color-bg`, not `--color-cream`) so it's a
  reasonable follow-up, but no dark values exist yet.
