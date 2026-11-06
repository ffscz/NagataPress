# Setting up and using NagataPress

Condensed from the official documentation (nagatapress.com/docs, version 1.3.24) and checked against the plugin code. Screen, button and field names are the English UI names.

## Contents

1. How it works and what gets translated
2. Requirements, installation, screens, quick start
3. License
4. Languages, URL format, slugs, settings per language
5. Translators: API key or assistant
6. Translation runs, monitoring, repair, new content, scheduled posts
7. Visual editor, manual edits, glossary, style, exclusions
8. SEO
9. WooCommerce
10. Language switcher
11. Settings and defaults, cache
12. Managing and deleting translations
13. Migration from WPML, TranslatePress, Polylang
14. Privacy and data

## 1. How it works and what gets translated

NagataPress works on the finished page: WordPress, the theme and the builder render the page in the source language, NagataPress swaps each text for its stored translation, adjusts links and SEO tags, and sends the page. Where text comes from does not matter. Translations are created once (by an AI provider or an assistant), stored in the site's database, and only read on page views. They stay when the plugin is deactivated or deleted.

Translated: titles, content, excerpts (also the cut excerpts of search results and post lists, found in the translation of the paragraph they come from, 1.3.20), menus, widgets, headers and footers as rendered; Elementor, Bricks, Breakdance, Oxygen, Beaver Builder, Divi, WPBakery, Cornerstone, Zion Builder, SiteOrigin, Gutenberg and ACF text/textarea/editor/URL fields; categories, tags and WooCommerce taxonomies and attributes; form placeholders, button captions, `alt`, `title`, `aria-label`; the title tag, meta description, Open Graph, Twitter and JSON-LD texts; URL slugs (optional); WooCommerce product pages, cart, checkout and customer emails; AJAX fragments and REST answers from stored translations; short labels drawn by JavaScript (whole texts, also those with prices and numbers a script fills in), also in inline scripts delayed by ZiziCache; Image Map Pro tooltips, text objects, buttons and links (maps of version 6 and older version 5 maps; object titles stay, 1.3.21). Internal links lead to the same page in the visitor's language with the translated slug, also links of Divi sections, rows, columns and modules (Divi 4 and 5, 1.3.21) and links with an anchor such as `/features/#calculator`, which keep the anchor (1.3.23). A decimal number followed by a unit gets the decimal separator of the target language; codes, versions and dates stay as written (1.3.20).

Left alone on purpose: scripts, styles, `code`, `pre`, `textarea`, SVG; visitor comments; the admin bar (texts and attributes); cookie consent banners; excluded terms and sections; `notranslate` elements; texts under three characters, numbers, symbols, three capital letters alone (`EUR`, `FAQ`), the currency words Kč, zł, Ft, lei and kr alone, texts starting with a URL or email; pages with more than 1,200 segments (served untranslated, never empty).

Page builder editors are left alone too: the editor, canvas and live preview of Bricks, Elementor, Divi, Beaver Builder, Oxygen, Breakdance, Brizy and other builders, and their AJAX and REST calls, get no translation, no rewritten links, hreflang, switcher or front editor. An editor opened on a language address goes to the source address, and "Edit with Bricks" and similar links keep no language prefix.

Fit: one-to-one sites where every page exists in every language with the same structure. Not for language-specific pages or different content per language.

Terms: a segment is one translated piece of text (texts over 280 characters are split into sentences); a stored translation is one segment for one language pair, reused on every page (texts that differ only in a price, in a number of the same plural category or in a product or category name share one); a bulk run translates all pages into one language in the background.

## 2. Requirements, installation, screens, quick start

Requirements: WordPress 5.3+ (tested to 7.1), PHP 7.4+ with DOM (libxml), mbstring and OpenSSL; an active license (14-day trial); an API key from Anthropic, OpenAI, Google (Gemini), Mistral AI or OpenRouter, or an MCP assistant; WP-Cron or a system cron; the server must be able to request its own pages; Apache, LiteSpeed or nginx. For the assistant: HTTPS, pretty permalinks (OAuth), WordPress 6.9+ only for the Abilities option.

Install: Plugins > Add New > Upload Plugin, choose the ZIP, Install Now, Activate. Activation creates the tables, flushes rewrites and, on Apache/LiteSpeed in Subdirectory format, writes a block between `# BEGIN NagataPress` and `# END NagataPress` in `.htaccess`. Updates appear on the Plugins screen once a license is active.

Screens (NagataPress menu; plugin navigation groups Workspace, Configuration, Data): Overview (status tiles, languages, license, Translations for visitors, auto-translate, cache, translation style, word count and estimate), Bulk translation, Languages, Scheduled posts, AI Provider, WooCommerce emails (only with WooCommerce), Settings, Glossary, SEO, Translations, Migration, License. A status bar on every screen shows plan, translator, bulk state, stored translations and Purge cache. Most controls save at once; the switcher cards, Scheduled posts and WooCommerce emails have their own Save buttons (an Unsaved changes bar reminds).

Quick start: activate the license; connect a translator; add target languages and choose the URL format; Count words on the Overview; Translate one language on Languages; check a translated page in the visual editor and place the switcher.

## 3. License

Activate: NagataPress > License (or the Overview's License card), paste the key (`XXXX-XXXX-XXXX-XXXX`), Activate License or Enter, wait for Valid license. The key is stored encrypted (AES-256-GCM) and shown with only its last four characters. The first activation creates the language settings (source = the WordPress site language, Subdirectory, no targets). Until then Languages shows License required.

The plugin checks the license daily, when the License screen is opened after an hour, on Check again, and before AI work when the stored answer is negative or older than 36 hours (at most every ten minutes). An unreachable server never counts as invalid: the last answer holds for 7 days, with a warning after a day.

Staging sites (1.3.21): a staging copy of the site (a local address, a staging subdomain such as `staging.` or `dev.`, or a staging host of the hosting provider) is activated as a staging site and does not count toward the activation limit, up to two per license. The License screen shows the row Activation as staging site.

When a license ends or cannot be verified, visitors keep stored translations, hand editing, glossary, exclusions, language and switcher settings, deleting and Purge cache keep working. Paused: bulk runs, auto-translate on publish, pre-translation of scheduled posts, the WooCommerce email queue, new slug words, the editor's AI buttons, the MCP tools that store translations (`get_batch`, `submit_batch`, `fix_translations`; the status, reading and setup tools still answer), WP-CLI translation, the Translate row actions and bulk action, Fix identical, Refresh archives, Reset and Retry errors. A warning appears seven days before a one-time license ends.

Move a license: Deactivate on the old site (releases the slot, removes key and license data, keeps translations and settings; visitors then see the original language), activate on the new one.

## 4. Languages, URL format, slugs, settings per language

Languages screen cards: Source Language, Target Languages, URL Format, URL Slugs, Content Exclusions, Settings per language, Widget Style, Widget Position, URL Exclusions. A card No translator connected yet appears while neither an API key nor the MCP endpoint is set up.

- **Source language**: saved at once; a target that becomes the source is removed from the targets. Translations are stored per language pair, so changing the source later means translating again.
- **Target languages**: 75 languages and 10 regional variants (en-US, en-GB, en-AU, es-ES, es-MX, es-AR, pt-BR, pt-PT, fr-FR, fr-CA), no limit. Arabic, Hebrew, Persian and Urdu get right-to-left automatically. + Add Language, tick, click outside. Removing a language (×) keeps its translations.
- **Language table**: URLs checkbox (start a slug scan with the run), Translated (pages, segments, words, progress), Remaining (Translate with reset, Cancel, Translate N pages for failures, or a check mark with Reset), ×. Under it: Translate All, Translate Selected (N), Remove Selected (N), Scan & Fix, Sync translation status.

URL formats:

| Format | Example | Notes |
|---|---|---|
| Subdirectory (recommended) | `example.com/de/` | Best for SEO; needs the `.htaccess` block on Apache/LiteSpeed. |
| Query parameter | `example.com/?lang=de` | Works everywhere, less SEO-friendly. |
| Separate domains | `de.example.com/` | Domain Mapping table, Save Domain Mapping. Needs DNS, a server alias and SSL per domain. Cookies (cart, login) are per domain; log in on a language domain to use the editor there; cross-origin fonts need `Access-Control-Allow-Origin`. Purge the page cache after saving. |

Automatic 301 redirects: in Subdirectory, `?lang=de` to `/de/...`, a source-language prefix to the plain address, a source slug under a prefix to the translated slug, an old `de.example.com` to `example.com/de/`; in Separate domains, `?lang=de` and `/de/` to the language domain; a changed translated slug keeps redirecting. Choose the format before the translated pages are indexed.

**URL slugs** (`/fr/a-propos/` instead of `/fr/about-us/`): pick the language, Scan & Translate Slugs (background, Cancel available). The scan covers published posts of public types and term archives with content; skips query-string and non-ASCII addresses. Slugs reuse translated titles or names, the rest goes to the AI provider, which needs a saved API key (not available over MCP). Numbers, short words, codes and brand taxonomy names stay. New scans keep existing translations; run again after new content. Not available for Japanese, Chinese, Korean, Thai, Arabic, Hebrew, Hindi, Bengali, Tamil, Telugu, Georgian, Burmese. There is no switch to turn slug translation off again (indexed addresses would 404). Delete Slugs reverts a language. Translated slugs by segment: type a slug and Save to pin it (redirect kept, never overwritten), Release to return it to automatic, Rebuild paths.

**Settings per language**: give another plugin's setting (Meta Pixel, GA, Ads IDs...) a value per language. Find tracking IDs, Set per language, type values (empty = same as source), Save settings per language. Add a setting by name for others (`wp_options` name plus `/`-separated keys). Up to 100 settings, 2,000 characters each; core settings are protected. Send the page language to Google Tag Manager adds `dataLayer.push({language: "de"})`.

## 5. Translators: API key or assistant

| | API key (Option A) | Own assistant over MCP (Option B, experimental, off by default) |
|---|---|---|
| Who translates | A provider model called by the server | The assistant in the user's own app |
| Needs | Provider account and key | Assistant subscription, HTTPS, pretty permalinks for OAuth |
| Who starts work | The plugin: runs, auto-translate, scheduled posts, editor AI buttons, WooCommerce emails, slugs, WP-CLI | The user, by telling the assistant |
| Cost | Provider bills tokens; estimates and monthly report | No price shown; the assistant's plan pays |

Both can be set up together: with a saved key the plugin uses it for its own runs and the assistant can still add translations. On an assistant-only site every button that starts a run, and WP-CLI, refuse with an explanation; row actions are hidden, auto-translate does nothing, scheduled pre-translation skips silently. Hand editing always works.

**Option A**: AI Provider > choose a tab (Claude, OpenAI, Google Gemini, OpenRouter, Mistral AI; each shows an estimated price per 1,000 words), pick a Model (read live from the key once saved), paste the API Key (`sk-ant-` Claude, `sk-` OpenAI and Mistral, `AIza` Gemini, `sk-or-v1-` OpenRouter), Test & Save. Results: "Key tested and saved. {model} answered the test translation." / "Saved, but {model} did not answer the test translation: {reason}." (a fallback model is used until another is picked) / a provider error. OpenRouter's first entry, Automatic: the best free model of the moment, picks a free model per request (rate limited). One provider and one model serve every language; there is no per-language choice. Defaults in 1.3.24 (they change as providers retire models): Claude Haiku 4.5, OpenAI GPT-4.1 mini, Gemini 3.6 Flash, Mistral Small, OpenRouter Automatic; when the preferred default is not available to the key, the cheapest available model is preselected. The most expensive models show a warning. Delete Key removes provider, model and key. The key is stored encrypted, sent only to the chosen provider in a header (Gemini `x-goog-api-key`), and masked in all messages. Translation costs: a table per month (Tokens IN/OUT, Segments, Cost USD, or EUR via filter), priced at list rates, tracking from the moment it was introduced.

**Option B**: see `connect-clients.md` and `mcp-translation.md`. Since 1.3.15 an assistant connected as an administrator can also read and change the setup (languages, translation style, Never Translate, glossary, the visitor switch, the shop's test cart), and since 1.3.16 read and correct stored translations. An assistant pass covers posts, archive pages, the shared texts and, on a shop, the side cart, cart, checkout, thank-you page, 404, search, My account and the customer e-mails.

## 6. Translation runs, monitoring, repair, new content, scheduled posts

A run translates one language at a time in the background (WP-Cron; WP-CLI for large sites). It covers published content of public post types (pages, posts, products, custom types), without system types. Phase Translating...: posts are read directly, only new segments go to the provider, and the live pages are requested after each batch to catch theme texts. Phase Verification: archive pages (blog, home, shop, popular categories and tags). If the server cannot request its own pages, those steps are skipped and theme-only text may be missing (fill it with Translate this page in the editor).

Already translated posts are skipped by later runs; stored texts are never sent again, except a stored translation identical to its source that was not saved by hand, which counts as untranslated and is asked again (and billed). Save such text by hand to stop that. Prices become placeholders (one placeholder for a price in any currency). Since 1.3.19 every number does too, with its plural category in the target language, and so do known product, category and attribute names inside longer texts: "Subtotal (7 items)" and "(8 items)" share one translation, "(1 item)" has its own, and "Add “X” to your cart" is translated once for every product, the name being filled in from its own translation. Digits are never translated or spelled out.

Estimate: Overview > Site Word Count > Count words: words, pages, estimated cost per language for the chosen model (switches to the site's own measured figures after 200 segments). An assistant shows no price; a free model shows free.

Start: Languages > Translate on a row > Start translation; or tick rows and Translate Selected (N); or Translate All (queues incomplete languages, one at a time).

Bulk translation screen: status badge, progress circle and queue bars, tabs Live log (last 120 lines), Errors (up to 100 failed pages with reasons), History (pages per language and last translated), Last answers from the AI provider (newest eight failures, keys masked), Pace (Normal or Gentle (small hosting)), Cancel (keeps stored translations, empties the queue), Retry errors, Start new bulk job. Runs sleep when the server is busy, wait out rate limits (pause after 20 waits in a row), adapt batch size, and stop on used-up credit, persistent refusals or a license that no longer allows AI work. Fix the cause and click Translate again.

On a site that translates only through an assistant, Bulk translation starts nothing ("This site translates over MCP"): it shows the assistant's pass per language (texts stored, pass progress), a Live log of the assistant's activity, Errors and History, and Shop texts (Step, Last collected, Found, Missing, Note) for the cart, checkout, e-mails and other shop pages the pass collects.

Scan & Fix opens Translation Health per language: Total, OK, Missing (posts), Identical (stored translations equal to the source, excluding hand edits and assistant `keep` texts), Failed. Buttons: Translate missing (needs a key), Fix identical (deletes identical automatic translations and resets progress; no confirmation), Refresh archives (needs a key), Deep scan (segment coverage per page). Sync translation status marks a post translated when at least 80 percent of its texts are stored (use after imports or restores; Deep scan finds the 80 to 99 percent pages). Reset forgets which posts are done and reruns cheaply from the stored cache.

New content is not translated on the fly. Ways in: Auto-translate on publish (Overview; each language a separate job, first after about 30 s, then 60 s apart; also on updates of published posts); post list column with language badges ("outdated" in amber after edits) and the row actions Translate (all languages) and Fix translation (re-asks every text of the page for one language, replacing automatic translations as answers arrive, keeping hand edits) and the bulk action Translate with NagataPress; the editor's Translate this page. On assistant-only sites: a new assistant pass.

Scheduled posts (NagataPress > Scheduled posts): Pre-translate scheduled posts (off), Translate this far ahead (15 min to 3 days, default 1 hour), Post types (default Post), Posts per check (1 to 50, default 5), Save, Run a check now; a check every 15 minutes renders the post locally. Needs an API key; assistant-only sites are skipped ("Nothing to do: no AI provider is set up on this site.").

## 7. Visual editor, manual edits, glossary, style, exclusions

**Visual editor**: as an administrator on the front end, click NagataPress Editor in the admin bar (or add `?nagatapress_edit=1`). On for an hour across pages. Sidebar: header with a price estimate for missing words (MCP, free, or a dash), Translate to: language picker, Translate this page (20 texts at a time, needs a key), tabs All, Meta, Titles, Menu, Content, Search text, and the list with Enter translation… fields, Save Translation (check mark, or Ctrl/Cmd+Enter) and Translate via AI. Badges: Missing (red), AI (pink), Manual (dark). Click a text on the page to jump to its field. An edit changes that text on every page (stored once). Keep the same numbers and prices in the same order, write product and category names exactly as they are written in that language (such a text is shared by every product and the name is filled in from its own translation; the editor refuses an edit that drops or changes the name), and keep every `<t1>...</t1>`/`<t2/>` marker (shown as pills with insert buttons). A text with a number is stored per plural category, so an edit made on "7 items" applies to every count of that category (in English to every count but 1). Translate via AI deletes the stored translation (manual too) and asks the provider again; on assistant-only sites it refuses and deletes nothing.

**Manual edits are protected**: never overwritten by runs, auto-translate, scheduled pre-translation, WP-CLI, MCP tools (unless an administrator lets the assistant replace one with `overwrite_manual`), Translate this page, Fix translation, Delete page translations, Fix identical, or adding an excluded term. Replaced only by Translate via AI on that text; removed by deleting a whole language or everything, and with the old wording when the source text is reworded (cleanup), or by Clean up now with the box for hand-edited unused translations ticked.

**Glossary** (NagataPress > Glossary): Source term, Translation, language pair, optional Notes, Add term. One entry per pair (same term replaces). Sent with every provider request and assistant batch. Affects only new translations ("N existing segments affected"); apply to existing pages with Fix translation or by deleting page translations.

**Translation Style** (Overview): Neutral / Corporate (default, faithful with adapted idioms and units) or Contextual (adds Brand name and What does your company do? (both required), Tone: Formal (default), Relaxed, Friendly, Professional, Creative, and Special instructions such as how to address the reader). Applies to new translations and to assistant batches.

**Content Exclusions** (Languages): Never Translate, up to 100 exact terms kept as written (a pasted list is split into terms; adding one reopens only the automatic translations that changed it, and a translation that drops or changes a term is refused); Exclude Sections, up to 50 CSS selectors (`.class`, `#id`, `[attribute]`, tag). Text inside excluded sections is also remembered as excluded terms (up to 500). `notranslate` is always excluded.

## 8. SEO

Automatic on translated pages: `lang` attribute (and `dir="rtl"`); `canonical`, `og:url`, `twitter:url` rewritten to the language version of the printed canonical, never with campaign arguments, never added where missing; `og:locale` as `language_REGION` when present; `hreflang` alternates for the source and every target language with stored translations plus `x-default` (source); title and meta texts translated (`description`, `og:title`, `og:description`, `og:image:alt`, `twitter:title`, `twitter:description`, `twitter:image:alt`); `alt`, `title`, `aria-label`, `placeholder`, button captions; JSON-LD text values (`name`, `description`, `headline` and similar) translated and site URLs localized, `inLanguage` follows; translated slugs and 301s; the sitemap `/nagatapress-sitemap.xml` and a `Sitemap:` line in a WordPress-generated `robots.txt`.

SEO screen: Sitemap Preview (pages, languages, URLs) and Active Redirect Rules for the URL format (Query to subdirectory, Default language prefix, Slug translation, Subdomain migration, WPML slug cleanup; or Query to domain, Prefix to domain, Slug translation; none for Query parameter).

Sitemap: lists pages of every language with at least one translation (404 until then), up to 5,000 source pages per file (index with `nagatapress-sitemap-1.xml`... beyond), lastmod and `xhtml:link` alternates; source-language URLs stay in the SEO plugin's sitemap. Leaves out attachments, templates, builder libraries, password-protected posts and noindex content from The SEO Framework, Yoast SEO, Rank Math and SEOPress (terms are not checked). Served with `X-Robots-Tag: noindex`. Yoast and Rank Math SEO fields are read directly for scheduled posts.

## 9. WooCommerce

Built in. Products, attributes, taxonomies, shop, archives, cart and checkout are translated with the site; redrawn fragments and block cart/checkout REST answers use stored translations, and missing ones join the email queue. Orders record the language they were placed in (classic and block checkout); thank-you and payment pages follow it; payment gateways get the order's locale (not for source-language orders or offline methods), with Comgate's page language set too (filter `nagatapress_payment_locale`).

WooCommerce emails screen: Translate customer emails (off) sends customer emails in the order's language (never emails to the shop owner); Show a language selector at checkout and in the account area (off; classic checkout only). Language order: the order's email language, the order's language, the customer's saved language. Sending never waits for the provider: missing texts go out in the source language and join a queue (up to 2,000), translated hourly 60 at a time; Translate email templates now queues all template texts and translates the first 200 at once. The editor (pick E-mail and Language) previews with a sample order built in memory (never a real order), lists lines with filters, Save (Manual), Reset (asks the provider again), Translate missing with AI, Send test (built from the same sample order; send it to your own mailbox). Wording overrides set an exact subject or heading per email and language. On a site that translates only through an assistant, the queued texts are translated by the assistant in its next pass, and Collect e-mail texts now takes the place of Translate email templates now.

Customer data: since 1.3.17 texts with customer names, addresses or notes never reach the queue, and since 1.3.18 a greeting with the customer's name is stored once with a placeholder for the name. Old translations with customer data (WooCommerce emails screen) finds rows that versions before 1.3.17 stored: Scan lists them with the personal parts masked, Delete removes only those rows (WP-CLI: `wp nagatapress privacy-cleanup`).

Shop texts (WooCommerce emails screen): Allow a test cart when collecting shop texts (on). To collect the texts of the cart and checkout as a customer sees them (for an assistant's pass), an anonymous cart with one product is created and deleted right away, without an order or e-mail; analytics plugins may count it as an add to cart.

## 10. Language switcher

Floating widget added automatically, or `[nagatapress]` shortcode. Lists all configured languages (source included) and links to the same page with translated slugs. No browser-language redirects ever.

Widget Style (ten): Globe + Name (default), Globe + Code, Globe, Flag + Name, Flag + Code, Flag, Name, Code, Inline Flags, Inline Codes. Names in their own language, bundled flags, two-column panel with search above six languages. Widget Position: Bottom Right (default), Bottom Left, Top Right, Top Left, Shortcode. Save switcher settings, then purge the page cache. URL Exclusions: one path per line starting with `/`, `*` wildcard (`/my-account/*` excludes children, not the parent), compared without the language prefix (write translated slugs), non-ASCII paths not matched. Shortcode styles and markup: `developers.md`.

A language appears in the switcher as soon as it is configured, even before its translations exist.

## 11. Settings and defaults, cache

| Setting | Default |
|---|---|
| Auto-translate on publish | Off |
| Translation Style | Neutral / Corporate (Tone Formal) |
| Source Language | WordPress site language |
| Target Languages | None |
| URL Format | Subdirectory |
| URLs per language row | Ticked where supported |
| Translate Slugs | Enabled on a new site |
| Never Translate, Exclude Sections, URL Exclusions | Empty |
| Send the page language to Google Tag Manager | Off |
| Widget Style / Position | Globe + Name / Bottom Right |
| Provider, Model, API Key | None (model: provider default; OpenRouter: Automatic) |
| Enable the MCP endpoint | Off |
| Allow sign-in with OAuth | On (works while the endpoint is on) |
| Offer the tools as WordPress Abilities | On |
| Pace | Normal |
| Pre-translate scheduled posts | Off, 1 hour ahead, Post, 5 per check |
| Translate customer emails / language selector | Off / Off |
| Allow a test cart when collecting shop texts | On |
| Show translations to visitors | Off until target languages are saved with a license key; stays off on a site that translates only through an assistant |
| Remove unused after | 1 year |
| Show the mascot on the Overview | On (per user) |

Show translations to visitors (Overview, card Translations for visitors) switches translation on or off for visitors; saving the rest of the Overview does not change it. Saving target languages with a license key stored also switches it on, except on a site that translates only through an assistant: there it stays off while the assistant translates, until someone switches it on.

Translation cache: translated pages are cached (up to 12 hours, one hour while texts are missing) and refreshed automatically when translations change through the plugin. Purge translation cache (Overview, status bar, admin bar) clears it, asks known caches to clear (WP Rocket, LiteSpeed Cache, W3 Total Cache, WP Super Cache, WP Fastest Cache, Cache Enabler, SiteGround Optimizer, Autoptimize, Swift Performance, Nginx Helper, Breeze, WP-Optimize, Hummingbird, Comet Cache, FlyingPress, ZiziCache, WP Engine, Kinsta, Cloudflare plugin, Varnish), fires `nagatapress_after_purge_cache`, and deletes stored translations identical to their source unless edited by hand. Other caches are not cleared after bulk runs, Translate on a post, editor edits or assistant translations: purge them once afterwards.

## 12. Managing and deleting translations

NagataPress > Translations (Settings has the same delete tools). Deleting is permanent and re-translating is billed again.

- Delete the translations of one page: Page URL (`/de/my-page/` fills the language), Language, Delete page translations, confirm "Delete M translation(s)". Hand edits stay; shared texts are deleted for every page; the page is unmarked so the next run redoes it.
- Delete a language: pick it, Delete all translations for this language, type the language code, Delete all. Removes hand edits too; resets progress; the language stays configured.
- Delete everything: type `DELETE`. Glossary, excluded terms and slugs stay.
- Old and unused translations: edits are compared with the earlier wording; replaced wording (close successors, not under 12 characters) is removed 14 days after the edit unless it shows up again. Unused translations are removed after Remove unused after (90 days, 180 days, 1 year default, Never); hand edits only with the extra checkbox on Clean up now. Clean up now does it at once.

## 13. Migration from WPML, TranslatePress, Polylang

Back up first; keep the old plugin and its data until the import is done; activate the license and connect a translator for what is missing. NagataPress > Migration: I am migrating from another translation plugin..., tell the old URL format (Detect from a pasted address, or pick Subdirectory or Query parameter), Detect Competitor Plugins, then Import per language ("1204 translations imported (EN -> DE)"). WPML: translated posts (up to 5,000 pairs per language pair) and complete String Translation strings (up to 50,000). TranslatePress: the dictionary (automatic or reviewed entries, up to 50,000). Polylang: published posts with published translations. Posts are matched paragraph by paragraph (same structure works best); identical texts are skipped. Weglot and GTranslate are recognised but have nothing to import. Then run each language normally; imported texts are reused.

Redirects (Subdirectory): `?lang=xx` to `/xx/page/`, `en.example.com/page/` to `example.com/en/page/` (while the subdomain still reaches the site), untranslated slug to translated, source prefix removed. WPML Database Cleanup (after import): Scan WPML Duplicates, type `DELETE`, Clean Up Duplicates (batches of 50, Resume Cleanup), redirects for differing slugs are kept. It deletes translated copies of every WPML post type in every WPML language.

## 14. Privacy and data

Stays on the server: translations, glossary, slugs and redirects (tables), settings with encrypted API and license keys and license details, token usage per call (no text), work logs (keys masked), temporary copies (translated pages up to 12 hours, MCP batches 24 hours, the texts of pages an assistant's pass fetched for up to a week so unchanged pages are not fetched again), MCP app sign-ins (secrets hashed), per-user preferences, order languages. No NagataPress cloud.

Leaves the server: texts and instructions (language pair, glossary, brand context; excluded terms replaced by placeholders) to the chosen AI provider only during translation work (OpenRouter also gets the site address); batches to the user's assistant only when it asks; license key, site address and version to the license server (activation, daily, checks); license key, instance, site and version to the update server; a metadata request to an OAuth app's https address when it signs in. Flags, fonts and switcher assets are bundled. Visitors: no cookies, no tracking, no browser-language redirect; only `nagatapress_edit` for administrators using the editor.

Who can do what: settings, translating, deleting and the editor need `manage_options`; the MCP translation tools need `nagatapress_translate` or `manage_options`; the MCP setup tools, and replacing hand edits over MCP, need an administrator (over OAuth also a connection an administrator allowed to change settings).
