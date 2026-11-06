# NagataPress for developers

WP-CLI, filters and the action, the shortcode, REST routes, capabilities and stored data. Add filters from a small plugin (or a must-use plugin) rather than a theme's `functions.php` when the site may change themes.

## Contents

1. WP-CLI
2. Filters and the action
3. The `[nagatapress]` shortcode and switcher markup
4. REST routes
5. Capabilities
6. Stored data and uninstall

## 1. WP-CLI

Available when WP-CLI is installed. No command asks for confirmation.

| Command | What it does |
|---|---|
| `wp nagatapress translate --lang=<code>` | Translates all published content into one target language. The code must match a configured target exactly (lowercase); otherwise the command lists the configured ones. |
| `wp nagatapress translate --lang=all` | Every target language, one after another. |
| `--batch=<n>` | Posts per batch, default 25. |
| `--resume` | Continues an interrupted run: completed languages are skipped, the current one continues after the last saved post. |
| `wp nagatapress status` | Table per language: Language, Total, Translated, Errors, Percent, Words; says when a run was interrupted and how to resume. |
| `wp nagatapress cancel` | Removes the bulk job data, the saved command-line progress and the locks. For cleaning up after an interrupted or stalled run; it does not stop a command that is still running. |
| `wp nagatapress license` | What the stored license allows: key (stored or none), status, type, valid until, last verified, last attempt, failed checks in a row, how long the last answer holds, AI translation (available or paused with a reason code), a warning when ending or in grace. |
| `wp nagatapress privacy-cleanup` | Finds stored translations that hold customer data (texts of customer e-mails stored by versions before 1.3.17) and only counts them with masked samples. `--delete` deletes the rows found; `--lang=<code>` one language; `--extended` also checks short translations made by a provider or an assistant; `--include-common` also deletes rows matched only by values many customers share; `--include-manual` also deletes hand edits. |
| `wp nagatapress license --check` | Asks the license server first and prints `Check: answered`, `unreachable`, `no_key` or `legacy`, then the table. |

How `translate` behaves:

- It stops with an error when the site is not configured, has no target language, the license does not allow AI work, or no AI provider with a saved key exists. An assistant-only site gets "This site translates with your own assistant (MCP), so a bulk run has nothing to call. ..."; a site with nothing set up gets "Set up an AI provider, or switch on translation with your own assistant (MCP) on the AI Provider page, before you translate."
- It walks published posts of the public post types (minus system types such as attachments, menu items, product variations, Elementor library, Contact Form 7 forms, coupons, ACF field groups, fonts and global styles) in ID order, and sends only segments without a stored translation to the provider. One line per batch: percentage, posts, newly translated and cached segments, errors, speed, time left, first title.
- It translates what is stored in the posts (title, excerpt, content, page builder data, ACF fields). It does not fetch rendered pages or archives, so theme and WooCommerce texts (menus, header, footer, widgets, Add to cart, related products) and archive pages (blog, shop, categories) need an admin run or Translate this page in the visual editor.
- Progress is saved every three batches. Run long jobs inside `screen` or `tmux`; after a dropped connection run the same command with `--resume`.
- A lock marks an active run. Starting again without `--resume` only warns ("Another CLI process may be running ..."). `wp nagatapress cancel` clears it.
- When the provider reports used-up credit, or the license stops AI work, it saves progress and exits; continue with `--resume` after fixing the cause, which starts with the batch that stopped.
- A rate-limiting provider is waited for and the same batch asked again, at most 15 minutes per batch (filter `nagatapress_cli_max_wait`, seconds). If the provider does not recover, or the database refuses the translations, the command stops and `--resume` continues there.
- It processes posts even if the admin marks them translated; already stored segments are not sent again, so this costs little. Its progress shows on the Languages screen.

Examples:

```bash
wp nagatapress translate --lang=de
wp nagatapress translate --lang=all --batch=50
wp nagatapress translate --lang=all --resume
wp nagatapress status
```

## 2. Filters and the action

| Filter | What it changes | Arguments | Default |
|---|---|---|---|
| `nagatapress_inline_grouping` | Group sentences across inline tags (links, bold) into one segment with markers. | `$on` | `true` |
| `nagatapress_max_runtime_segments` | Most segments a page may have; above it the page is served untranslated (`SKIP segment-limit`). `0` removes the limit. | `$limit`, `$lang`, `$html` | `1200` |
| `nagatapress_translate_structured_data` | Translate JSON-LD. | `$on`, `$lang` or `null` | `true` |
| `nagatapress_localized_script_handles` | Script handles whose localized data (texts passed to JavaScript) is translated. | `$handles` | WooCommerce scripts |
| `nagatapress_builder_json_attributes` | Builder elements that keep readable text in a JSON attribute. | `$config` (CSS class => `attribute`, `keys`) | Bricks animated typing |
| `nagatapress_js_map_max_segment_length`, `nagatapress_js_map_max_bytes`, `nagatapress_js_map_max_pairs` | Limits of the script that swaps texts drawn by JavaScript. | the value | `160`, `40960`, `500` |
| `nagatapress_ajax_translation_skip_actions`, `nagatapress_ajax_translation_skip_prefixes`, `nagatapress_ajax_translation_skip_wc_ajax` | `admin-ajax.php` actions, action prefixes and `wc-ajax` values whose answers are not translated by the general AJAX translation (mini cart and checkout review fragments have their own filters). | the list | checkout, coupons, form submissions (WPForms, CF7, Forminator, Gravity Forms, Fluent Forms, Ninja Forms, Elementor Pro forms), NagataPress' own actions |
| `nagatapress_sslverify_url` | Whether a request the site makes to itself verifies TLS. | `$verify`, `$url`, `$host` | `true`, except local hosts (`localhost`, `127.*`, `::1`, `.test`, `.local`, `.dev`) and local/development environments |
| `nagatapress_client_ip` | The address the OAuth rate limits count by (1.3.22). | `$ip`, `$remote` (REMOTE_ADDR) | `CF-Connecting-IP` when the request comes from a Cloudflare address, else REMOTE_ADDR; X-Forwarded-For is never used |
| `nagatapress_cloudflare_ranges` | Address ranges trusted as Cloudflare (1.3.22). | `$ranges` (CIDR list) | Cloudflare's published IPv4 and IPv6 ranges |
| `nagatapress_script_json_sources` | Inline scripts whose JSON holds readable text (1.3.21). Entry: `needle` (a string the script contains), `var` (the variable the JSON object is assigned to), `keys` (keys whose string values are text) and optionally `url_keys` (site addresses pointed to the language version). | `$sources` (name => definition) | Image Map Pro |
| `nagatapress_link_script_vars` | Script variables holding link data whose addresses lead to the visitor's language (1.3.21). | `$vars` | `et_link_options_data`, `diviElementLinkData` (Divi) |
| `nagatapress_cors_allowed_origins` | Origins that may read fonts, styles and AJAX answers (separate domains). | `$origins` (`https://example.com`) | the site's domains and language domains |
| `nagatapress_auto_translate_delay`, `nagatapress_auto_translate_spacing` | Seconds before the first language of a published post is translated, and between languages. | the value | `30`, `60` |
| `nagatapress_bulk_max_load_per_cpu` | Load per CPU above which a run sleeps (`0` disables). | the value | `1.5` |
| `nagatapress_bulk_max_db_ping_ms` | Milliseconds for a trivial query (`0` disables). | the value | `250` |
| `nagatapress_bulk_max_options_write_ms` | Milliseconds for an options write (`0` disables). | the value | `1000` |
| `nagatapress_bulk_max_page_fetch_ms` | Milliseconds for one of the site's own pages. Keep it above zero: the page crawl also uses it as a cut-off. | the value | `10000` |
| `nagatapress_bulk_busy_pause` | Seconds a run sleeps when busy (at least 60). | the value | `180` |
| `nagatapress_batch_token_budget` | Input tokens per request. | `$budget`, `$provider` | `12000` Gemini and OpenAI, `8000` others |
| `nagatapress_throttle_settings` | Pace, batch size and parallel requests per provider. | `$settings` (`pace`, `batch`, `parallel`), `$provider` | per provider |
| `nagatapress_token_profile` | Tokens per source word used by estimates. | `$profile` (`in`, `out`, `basis`, `segments`), `$provider`, `$model` | typical figures, then measured ones |
| `nagatapress_price_currency` | `USD` or `EUR` for displayed prices. | `$currency` | `USD` |
| `nagatapress_usd_to_eur` | Conversion rate. | `$rate` | `0.92` |
| `nagatapress_license_renew_url` | Address of the Renew or upgrade link. | `$url`, `$state` | pricing section of nagatapress.com |
| `nagatapress_sitemap_post_types` | Post types in the translated sitemap. | `$types` | public viewable types without builder templates and noindex types |
| `nagatapress_sitemap_exclude_post` | Leave one post out of the sitemap (can add exclusions, not remove them). | `$excluded`, `$post_id` | `false` |
| `nagatapress_hreflang_query_args` | Query args kept in hreflang and `x-default` URLs of pages without a usable canonical. | `$args` | `p`, `page_id`, `attachment_id`, `name`, `pagename`, `post_type`, `cat`, `category_name`, `tag`, `taxonomy`, `term`, `author`, `author_name`, `m`, `year`, `monthnum`, `day`, `s`, `page`, `paged`, `cpage`, plus public taxonomy and post type query vars |
| `nagatapress_payment_locale` | Locale a payment gateway sees while setting up an order's payment. | `$locale`, `$order` (WC_Order), `$gateway_id`, `$lang` | locale of the order's language |
| `nagatapress_lang_options_language` | Language whose Settings per language values are used. | `$lang` | language of the request |
| `nagatapress_known_name_post_types`, `nagatapress_known_name_taxonomies` | Where the known names come from that are masked as `{{N1}}` inside longer texts (1.3.19). | `$post_types` or `$taxonomies` | `product`, `product_variation`; the product taxonomies (categories, tags, brands, `pa_*` attributes) |
| `nagatapress_known_names`, `nagatapress_known_names_max` | The list of known names itself, and how many are kept. | `$names`; `$max` | published titles and terms; 50,000 |
| `nagatapress_is_builder_request` | Whether a request is a page builder's editor, canvas or live preview, which NagataPress leaves alone (1.3.14). | `$is` | true for the builders NagataPress knows |
| `nagatapress_js_masked_max_segment_length`, `nagatapress_js_masked_max_bytes`, `nagatapress_js_masked_max_pairs` | Limits of the texts with prices and numbers in the in-browser translation map (1.3.18). | the value | `160`, `8192`, `100` |
| `nagatapress_mcp_archive_taxonomies` | Taxonomies whose term archive pages an assistant's pass visits (1.3.17). | `$taxonomies` | public taxonomies with archives (categories, tags, `pa_*`, brands...), not post formats |
| `nagatapress_mcp_read_second_page` | Whether an assistant's pass also reads the second page of an archive (1.3.25). Turn off for a host that limits requests from the site to itself. | `$read` | `true` |
| `nagatapress_mcp_fetch_parallel` | Pages an assistant's pass fetches at once (1 to 10). | `$count` | `4` |
| `nagatapress_mcp_page_index` | Whether an assistant's pass keeps the texts of fetched pages and takes an unchanged page from there next time (1.3.18). | `$on` | `true` |
| `nagatapress_mcp_page_index_options`, `nagatapress_mcp_page_index_layout_types` | Options and post types (templates, menus) the page index watches: a change fetches every page again. Add the option of a plugin that prints a text on every page. | `$names`; `$types` | NagataPress' own list |
| `nagatapress_mcp_page_index_listing` | Whether a post lists other posts, so any published change fetches it again. | `$listing`, `$id`, `$type` | true for pages |
| `nagatapress_mcp_page_index_max_age`, `nagatapress_mcp_page_index_max_bytes` | How long the index trusts an unchanged page (seconds, `0` turns it off in effect), and its size cap (`0` = no limit). | the value | one week; 128 MB |
| `nagatapress_shop_test_cart` | Whether the shop collection may use a test cart (1.3.18). | `$allowed` (the setting) | the Allow a test cart setting |
| `nagatapress_wc_email_sample_order` | The order the e-mail previews, Send test and the warm-up render with (1.3.17). | `$order` (WC_Order or null) | a sample built in memory, never stored |
| `nagatapress_privacy_cleanup_common`, `nagatapress_privacy_cleanup_providers` | For the cleanup of translations with customer data: how many customers must share a value for it to count as common (`0` = off), and the origins of the rows it reads. | the value | `40`; `bulk_fix` |
| `nagatapress_cli_max_wait` | Most seconds WP-CLI waits for a rate-limiting provider per batch. | `$seconds`, `$provider` | `900` |
| `nagatapress_cleanup_post_type`, `nagatapress_cleanup_taxonomy` | Whether edits are compared with the earlier wording (old-wording cleanup). | `$tracked`, `$post_type` or `$taxonomy` | public post types (not attachments) plus reusable blocks, templates, product variations; viewable taxonomies plus product attributes |

The action `nagatapress_after_purge_cache` (no arguments) runs after Purge translation cache and after deleting translations, saving Exclude Sections or Settings per language, Enable slug translation, Reset of a language, Fix translation on a post, and once after a plugin update. Hook a cache NagataPress does not know into it. It does not run after bulk runs, Translate on a post, editor edits or assistant (MCP) translations.

Example: raise the segment limit for one long landing page only.

```php
add_filter( 'nagatapress_max_runtime_segments', function ( $limit, $lang, $html ) {
    return ( false !== strpos( $html, 'id="long-landing"' ) ) ? 3000 : $limit;
}, 10, 3 );
```

## 3. The `[nagatapress]` shortcode and switcher markup

`[nagatapress]` draws the language switcher where it is placed (page, widget, builder header or footer, template). The only attribute is `style`: `classic` (Globe + Name), `globe-code`, `globe-only`, `flags-name`, `flags-code`, `flags-only`, `inline-flags`, `name-only`, `code-only`, `inline`. Without it, or with an unknown value, the style from Widget Style is used. With Widget Position = Shortcode there is no floating widget; with a corner position the shortcode adds a second switcher.

Markup: the floating widget's container has the ID `nagatapress-switcher`; shortcode switchers have `nagatapress-inline`, `nagatapress-inline-2`, ... Every switcher container has the class `notranslate`. URL Exclusions hide all of them on the listed paths.

Any element with the class `notranslate` is never translated, so wrap code samples, names or user content in it.

## 4. REST routes

Namespace `nagatapress/v1` (under `/wp-json/`):

| Route | Methods | Purpose |
|---|---|---|
| `/mcp` | POST (JSON-RPC), GET (405), DELETE (200) | The MCP endpoint; 404 while switched off; needs OAuth bearer or Application Password of a user allowed to translate (the setup tools: an administrator). |
| `/oauth/protected-resource` | GET | Protected resource metadata (RFC 9728). |
| `/oauth/.well-known/openid-configuration`, `/oauth/.well-known/oauth-authorization-server` | GET | Authorization server metadata. |
| `/oauth/authorize` | GET | Starts the sign-in, then the consent screen in wp-admin. |
| `/oauth/token` | POST | Code (PKCE S256) and refresh grants. |
| `/oauth/register` | POST | Dynamic Client Registration, public clients. |
| `/oauth/revoke` | POST | Token revocation. |

The OAuth routes answer 404 while Allow sign-in with OAuth is off. REST API answers requested with `?lang=` or from a translated page are translated from stored translations.

## 5. Capabilities

- Settings, translating, deleting translations and the visual editor: `manage_options`.
- MCP translation tools and abilities (status, get-batch, submit-batch, get-translations, fix-translations): `nagatapress_translate` or `manage_options`. Administrators get `nagatapress_translate` on activation and update; grant it to another role to let non-administrators translate through an assistant.
- MCP setup tools and abilities (get-settings, set-languages, set-translation-style, suggest-excluded-terms, update-excluded-terms, update-glossary, set-translation-enabled, set-shop-test-cart) and `overwrite_manual` of fix-translations: `manage_options`; over OAuth also the scope `nagatapress:settings`, which only an administrator can allow.
- An OAuth sign-in reaches only the NagataPress endpoint; an Application Password opens the account's whole REST API.

## 6. Stored data and uninstall

Tables (with the WordPress prefix): `nagatapress_translations` (one row per text and language pair: hash, languages, original and translated text, origin `bulk`, `ai`, `manual`, `assistant`, `migration_wpml`, `migration_translatepress` or `migration_polylang`, manual flag, created, changed, last used, old-wording date), `nagatapress_meta`, `nagatapress_glossary`, `nagatapress_slugs`, `nagatapress_slug_parts`, `nagatapress_slug_redirects`, `nagatapress_ai_usage` (tokens only, no text), plus the MCP sign-in table. `nagatapress_meta` also holds the page index of the assistant's passes. Texts with prices, numbers or known product and category names are stored once with tokens (`{{$}}`, `{{#}}`, `{{N1}}`); a text with numbers has one row per combination of plural categories in the target language. The option `nagatapress_mask_rules` records the masking rules the stored keys were made with (5 since 1.3.19); after an update that changes them, the rows are re-keyed once in the background.

Post meta `_nagatapress_translated_langs` marks posts translated per language; orders carry `_nagatapress_lang` and `_nagatapress_email_lang`; user meta `nagatapress_ui` holds screen preferences. The only visitor cookie is `nagatapress_edit` (administrators, one hour, visual editor).

Deactivating keeps everything. Deleting the plugin removes settings, license data, per-language values, scheduled events, logs and job records, the user and post meta above, the MCP sign-in table and the capability, but keeps the translation tables on purpose, with the options `nagatapress_db_version` and `nagatapress_mask_rules` that let a reinstall reuse them. To remove everything: back up, drop the `{prefix}nagatapress_*` tables, then delete the plugin (or delete those two options by hand afterwards). Deleting the plugin does not release the license activation: Deactivate on the License screen first. Order meta stays with the orders.
