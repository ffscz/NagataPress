# Troubleshooting NagataPress

Find the symptom, then go through the causes in order. Ask the user for what they see (the page source comment, the Live log line, the Errors tab, the message under a button) before guessing. MCP connection problems are in `connect-clients.md`, section 4.

## Contents

1. A language address does not open
2. A translated page still shows the original text
3. A run does not finish, pauses or reports errors
4. Error codes of the provider
5. A translation is wrong
6. The provider rejects the key or the model
7. Items rejected when an assistant submits a batch
8. License messages
9. Frequently asked questions

## 1. A language address does not open

`example.com/de/` gives a web server error page or "not found".

| Cause | Fix |
|---|---|
| Subdirectory format on Apache or LiteSpeed, and the rewrite block is missing from `.htaccess` (written only when WordPress can write the file, silently skipped otherwise). | Make `.htaccess` writable, then switch URL Format to another format and back to Subdirectory (clicking the selected format does nothing), or add the block below by hand. |
| Subdirectory format on nginx. | Keep the standard WordPress routing `try_files $uri $uri/ /index.php?$args;`. No extra block is needed. |
| Separate domains: the domain is not a server alias of the site, or lacks DNS or a certificate. | Set up DNS (A or CNAME), the server alias (ServerAlias, server_name, Domain Alias) and SSL for every domain. An unknown domain gets the server's own 404 before WordPress runs. |
| WordPress rewrite rules are stale. | Settings > Permalinks > Save Changes once. |

The block for the Subdirectory format:

```
# BEGIN NagataPress
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^([a-z]{2}(-[a-z]{2,4})?)(/.*)?$ /index.php [L,QSA]
</IfModule>
# END NagataPress
```

## 2. A translated page still shows the original text

In this order:

1. **Is the language translated?** The Translated column on Languages. A language without stored translations shows the original.
2. **Is a license key stored and translation on?** Without a stored key visitors get the original language, the floating switcher and hreflang disappear (a shortcode switcher stays and links to original pages). Check Show translations to visitors on the Overview (card Translations for visitors). Saving the Overview does not switch it on; saving target languages with a key stored does, except on a site that translates only through an assistant, where it stays off until someone switches it on (or the assistant does, with `nagatapress_set_translation_enabled`).
3. **Is the page cached?** Clear the caching plugin's cache (and Purge translation cache if needed; note that it also deletes stored translations identical to their source, except hand edits).
4. **Read the page source.** Search for `NagataPress`. The comment `<!-- NagataPress Translation: cache=... lang=de segments=... cached=... api=... skip=... -->` gives the number of texts (`segments`), found in the database (`cached`), missing (`api`) and stored identical to the original (`skip`). Missing texts stay in the original until translated. Other comments:
   - `NagataPress Translate: SKIP segment-limit`: more than 1,200 segments (filter `nagatapress_max_runtime_segments`), served untranslated.
   - `NagataPress Translate: SKIP no-lang`: not a target-language address. Normal on every source-language page.
   - `NagataPress Translate: SKIP no-segments`: nothing translatable.
5. **Is the text translatable at all?** Never translated: text nodes under three characters, numbers, symbols, exactly three capital letters (`EUR`, `FAQ`), a text starting with a URL or email address, the contents of scripts, styles, `noscript`, `code`, `pre`, `textarea` and `svg`, the admin bar, cookie consent banners (Complianz, CookieYes, Cookie Notice, Cookiebot, Borlabs, Cookie Law Info, iubenda, OneTrust, Usercentrics and similar), visitor comments, anything inside `notranslate` or an excluded section, Never Translate terms. Attribute values such as `alt` and `placeholder` are translated from two characters.
6. **Is the text drawn by JavaScript after load?** A small script swaps short whole texts it knows, not everything. Open the page in the visual editor and use Translate this page, or translate the text in the list.
7. **Is the text new?** New text needs auto-translate on publish, Translate on the post, the visual editor, or (on assistant-only sites) a new assistant pass with `restart: true`.

### The run says 100 % but parts of a page (footer, header, form, widget) are still in the source language

The menu or the content being translated proves that translation works. The percentage counts posts, not texts, so it can say 100 % while some texts of a page are missing. Clicking Translate again does not help: a run skips every post that is already marked as translated.

| Cause | How to tell | Fix |
|---|---|---|
| A page cache shows an old copy. | Logged out (private window) the page differs from logged in. | Purge the caching plugin's cache. |
| Texts that only the theme or builder adds (footer and header templates, forms, widgets, related products) are caught only when the run requests the live pages. If the server could not request its own pages (firewall or bot protection, HTTP authentication on staging, a `localhost` setup), they were skipped. | The page source comment shows `api=` above 0; the Errors tab or the Live log shows page errors. | On the post list, use the row action Fix translation for that page and language, or open the page with NagataPress Editor and click Translate this page. Fix the blocked self-requests for future runs. |
| The post was marked translated while some texts were missing (Sync translation status marks a post at 80 % coverage). | Scan & Fix > Deep scan lists pages with incomplete coverage. | Fix translation on those posts, or Reset on the language (reruns everything, stored texts are reused, only the gaps are billed). |
| The text is shown only after an interaction (a form's error or success message) or drawn by JavaScript. | Not in the page source. | Translate it in the visual editor after triggering it, or translate the string in the list. |
| Stored, but identical to the source. | `skip=` above 0 in the comment. | Scan & Fix > Fix identical, then Fix translation or Reset; save texts that must stay by hand. |
| On an assistant-only site, the texts were added after the last pass. | The status of the assistant tools says `pass_done`. | Ask the assistant for a new pass with `restart: true`. |
| Right after a plugin update, a few texts with numbers or product names show the original again. | The update re-keyed the stored translations (one stored text now serves every count of a plural category and every product name). A translation it could not convert safely, such as a number written out as a word or a product name translated differently from the product's own translation, was set aside, so that text needs one new translation. | A new assistant pass with `restart: true` (the update reopens the passes), or Fix translation / Reset for API runs. |

## 3. A run does not finish, pauses or reports errors

Most messages appear in the Live log of the Bulk translation screen.

| What you see | Meaning and fix |
|---|---|
| Partial progress with the Translate button again; the Dashboard says NagataPress could not translate. | The provider refused the requests. Read Last answers from the AI provider on Bulk translation, fix the key, credit or model, click Translate. |
| "Insufficient credits or expired subscription. Add credits and resume." | Add credit at the provider, click Translate. |
| "The AI provider refused every attempt for a long time (rate limit, key or model). Check the AI settings and resume." | 20 waits in a row paused the run. Check the provider account and the Model. |
| "Waiting for the AI provider (rate limit); next attempt in N s." | Normal; it continues by itself. |
| "The server is busy (...); continuing in N s." | Load, database or the site is slow. Set Pace to Gentle (small hosting) or use WP-CLI at a quiet hour. Thresholds: load per CPU 1.5, DB ping 250 ms, options write 1 s, own page 10 s; pause 3 minutes (filters in `developers.md`). |
| "Translations could not be written to the database; trying again in N s. See the PHP error log." | Check the PHP error log and the database's free space. |
| Errors tab: "Page returned an error" (`HTTP 403`, `HTTP 500`...), "Page took too long to load" (`cURL error 28`), "Page returned empty content", "No URL found for this page" | The run could not read the page. Check password protection, maintenance mode, a firewall or bot protection blocking the server's requests to itself, HTTP auth on staging. Then Retry errors. 404 and 410 do not count as failures. |
| Errors tab with a short code | See section 4. |
| "AI translation is paused: ..." | License, see section 8. |
| Progress does not move | Background work needs WP-Cron (or a system cron calling it) and a site that can request its own pages. Keep Languages or Bulk translation open (both restart a stalled job), or run `wp nagatapress translate --lang=all --resume`. |
| "This site translates with your own assistant (MCP), so a bulk run has nothing to call. ..." | No API key is saved; the site translates only through an assistant. Run the assistant, or add a provider key. |
| "Set up an AI provider, or switch on translation with your own assistant (MCP) on the AI Provider page, before you translate." | Nothing is set up. |

To continue after fixing the cause, click Translate on the language again (on the Languages screen; Bulk translation only follows the job): translated pages are skipped, so only the rest is processed. Pages that were marked translated with gaps need Fix translation or Reset (see section 2).

## 4. Error codes of the provider

Shown on the Errors tab and in Last answers from the AI provider:

| Code | Meaning |
|---|---|
| `unconfigured` | No AI provider with a saved key. |
| `auth` | The provider rejected the key. |
| `quota` | Credit or quota used up. |
| `model` | The provider does not serve the model; pick another. |
| `transport` | The provider could not be reached or answered with an error. |
| `parse` | The answer could not be read. |
| `blocked` | The provider blocked the request or returned nothing usable. |
| `misaligned` | The answer had a different number of items than the request; refused. |
| `truncated` | The answer was cut off (too long for the model). |
| `translation_failed` | No usable translation for part of the page. |
| `failed` | Failed without a more precise reason; read Last answers. |
| `empty` | Google Gemini returned nothing usable. |
| `malformed` | Google Gemini's answer could not be read. |

A page whose provider answered nothing at all (no key, refused key, no credit, no connection) is recorded as failed, not done, and nothing stored is deleted before the provider answers.

## 5. A translation is wrong

| Problem | Fix |
|---|---|
| One text is wrong. | Correct it in the visual editor; the edit is protected. |
| A term varies from page to page. | Add it to the Glossary, then Fix translation on pages that already have the old wording. |
| A brand or product name gets translated. | Add it under Never Translate (Languages > Content Exclusions). |
| The tone does not fit. | Translation Style > Contextual with the brand context (Overview). |
| A text is identical to the original. | Scan & Fix > Fix identical, then run the language again. If it must stay as it is, save it by hand in the visual editor so it is not sent again (and billed again). |
| Numbers or prices look wrong. | They are placeholders refilled from each occurrence's source in their order; edit in the visual editor keeping the same values in the same order. |
| The word next to a number has the wrong form ("1 items", "5 položky"). | One translation serves every number of the same plural category, so the translation of that category is wrong. Correct it in the visual editor on a page that shows such a number, or let the assistant correct it (`nagatapress_get_translations`, `nagatapress_fix_translations`). |
| A product or category name inside a sentence is wrong. | The name is filled in from the name's own translation. Correct the name (its product title or term), not every sentence that contains it. |
| A page mixes two languages. | Fix translation on the post list, or Delete page translations and translate again. |

## 6. The provider rejects the key or the model

- Key refused: it must belong to the selected provider tab, be copied whole, and the account needs credit. The message under Test & Save quotes the provider.
- "Saved, but {model} did not answer the test translation: ... Your setting is kept as you chose it; translation falls back to another model until you pick a different one.": the key works but the model does not. Pick another Model. Until then NagataPress substitutes a reachable model per request and says so; it rechecks the model list daily.
- A model the provider withdraws while translations run: the Model setting is left alone. The run (also a WP-CLI run) goes on with a model the key can reach and shows "Your provider no longer offers {model} to this API key. Your setting has been left alone; translations are running on {model} until you pick a new model in the AI settings." Pick another model on AI Provider.
- After a move to a site with different WordPress security keys, the stored API key (and license key) cannot be decrypted: enter them again. Keep `AUTH_KEY` and `AUTH_SALT` the same on a copy to avoid this.
- Costs higher than expected: AI Provider > Translation costs (tokens and cost per month and model); the warning "This is one of the most expensive models available" marks the top price band; use Count words before large runs.

## 7. Items rejected when an assistant submits a batch

The same reasons apply to `nagatapress_submit_batch` and `nagatapress_fix_translations`. A reason may carry a detail after a colon.

| Reason | Meaning |
|---|---|
| `empty` | Empty translation. |
| `untranslated` | Identical to the source; send `keep: true` for text that must stay. |
| `tokens` | A `{{#}}`, `{{$}}` or `{{@}}` token was missing or added, typically a number written out ("two" for `{{#}}`), a price with its value, or a customer's name instead of `{{@}}`. |
| `names` | A name token `{{N1}}`, `{{N2}}`... was missing, repeated or renumbered, or the name was written out. Each name token of the text exactly once; the name is filled in from its own translation. |
| `placeholders` | An inline marker such as `<t1>` was missing, changed or added. |
| `html` | A script or `javascript:` link was added. |
| `damaged` | Looks like broken output. |
| `odd_character: <char> U+XXXX` | A character of another script or alphabet that the source does not have (a Cyrillic letter in a Latin word, a full-width space). |
| `excluded_term: keep X exactly as written` | A Never Translate term was dropped or changed. |
| `rekeyed` | The batch was handed out before an update changed how the text is keyed; call `nagatapress_get_batch` again. |
| `unknown_id` | Not part of the batch (or already accepted). With `fix_translations`: no stored translation with that id in this language. |
| `manual_edit` | `fix_translations` only: a person edited the translation by hand; an administrator can replace it with `overwrite_manual`. |
| `bad_item`, `store_failed` | `fix_translations` only: an item without `id` or `tr`; the database refused the write. |

Fix and resubmit with the same batch ID. A batch older than 24 hours is unknown: call `nagatapress_get_batch` again. Details: `mcp-translation.md`.

## 8. License messages

New AI work stops with "AI translation is paused:" plus a reason; visitors keep getting stored translations.

| Reason | Fix |
|---|---|
| the NagataPress license has expired | Renew (Renew or upgrade link). |
| the NagataPress license has been disabled | Contact support. |
| the NagataPress license is not active on this site | License screen, activate a valid key. |
| the (plan name) period has ended | A one-time license ended: switch to a subscription or lifetime license. |
| the NagataPress license could not be verified for more than 7 days | Allow outgoing HTTPS from the server, then Check again. |
| no NagataPress license is activated on this site | Activate a key. |

Activation messages: "License key not found." (check the key), "Invalid license key format.", "This license has reached its activation limit. Deactivate it on another site first." (lists the sites), "License service temporarily unavailable...", "Connection error: ..." (outgoing HTTPS blocked).

A site copied without deactivating and with different security keys shows an empty key field, Deactivate answers "No license to deactivate", and AI work pauses after 7 days.

## 9. Frequently asked questions

- **Deactivating the plugin?** Visitors see the original only; language addresses show the original or 404, switcher, hreflang and sitemap disappear, a `[nagatapress]` shortcode shows as text. Nothing is deleted; reactivating restores everything. Deleting the plugin keeps the translations too (it removes settings and MCP sign-ins).
- **License expired?** Visitors notice nothing; new AI work pauses.
- **Without an API key?** Yes, with an assistant over MCP (experimental), or OpenRouter's free models (rate limited).
- **Speed impact?** A page view reads stored translations and its own cache; no AI call. Runs pause when the server is busy.
- **Replace WPML or Polylang?** Yes when every language has the same structure (one-to-one translation). Not for language-specific pages or different content per language.
- **Page builders and ACF?** Yes, the rendered page is translated, so Bricks, Elementor, Breakdance, Oxygen, Beaver Builder, Divi, WPBakery, Cornerstone, Zion, SiteOrigin, Gutenberg and ACF text fields all work.
- **Leave a page out?** No per-page switch; use Never Translate, Exclude Sections or the `notranslate` class.
- **Change provider later?** Yes; stored translations stay.
- **Moving or staging to production?** Copy database and files; translations travel with the database and addresses follow the new domain. Deactivate the license on the old site first, re-enter the API key if security keys differ.
- **Right-to-left?** Arabic, Hebrew, Persian and Urdu switch automatically.
- **Limits?** No limit on languages or pages; a page over 1,200 segments is served untranslated unless the limit is raised.
