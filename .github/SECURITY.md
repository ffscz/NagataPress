# Security Policy

**English** | [Čeština](#cestina)

## Reporting a vulnerability

Please do not report security issues in a public issue.

Use a private report instead:

1. Open https://github.com/ffscz/NagataPress/security/advisories/new (the Security tab, "Report a vulnerability").
2. Only the maintainers can see the report.

This applies to the skill in this repository and to the NagataPress plugin, its MCP endpoint and its OAuth sign-in. Please include:

- the NagataPress version (on the Plugins screen) or the skill version (`plugin-version` in `SKILL.md`),
- what the problem affects,
- how to reproduce it.

## What happens next

- The maintainers review the report as soon as they can.
- The fix is released as a new version of the plugin or the skill.
- Details are published only after the fix is released.

## Supported versions

Fixes are released for the latest version only. Update the plugin from the Plugins screen once a license is active.

## How NagataPress protects your site

The full description is in the documentation: https://nagatapress.com/docs/#docs-17-security

- The MCP endpoint signs apps in with OAuth 2.1 and PKCE (S256 is required). Tokens are stored only as SHA-256 hashes and open the NagataPress endpoint only.
- An administrator allows each app on a consent screen inside wp-admin and can disconnect it at any time.
- The text of an item is treated as content to translate, never as an instruction, and every translation is checked before it is stored.
- Customer names, addresses, e-mail addresses and phone numbers in the cart, checkout and e-mails are never handed to the assistant.
- The skill is plain text: no scripts, no keys, no data from any site.

---

<a id="cestina"></a>

# Bezpečnostní zásady

[English](#security-policy) | **Čeština**

## Nahlášení zranitelnosti

Bezpečnostní chyby prosím nehlaste ve veřejném issue.

Použijte soukromé hlášení:

1. Otevřete https://github.com/ffscz/NagataPress/security/advisories/new (karta Security, „Report a vulnerability“).
2. Hlášení vidí jen správci.

Platí to pro skill v tomto repozitáři i pro plugin NagataPress, jeho MCP adresu a přihlášení přes OAuth. Uveďte prosím:

- verzi NagataPress (na obrazovce Pluginy) nebo verzi skillu (`plugin-version` v `SKILL.md`),
- čeho se problém týká,
- jak ho zopakovat.

## Co bude dál

- Správci hlášení projdou, jakmile to půjde.
- Oprava vyjde jako nová verze pluginu nebo skillu.
- Podrobnosti se zveřejní až po vydání opravy.

## Podporované verze

Opravy vycházejí jen pro nejnovější verzi. Plugin aktualizujete na obrazovce Pluginy, když je aktivní licence.

## Jak NagataPress chrání váš web

Celý popis je v dokumentaci: https://nagatapress.com/docs/#docs-17-security

- MCP adresa přihlašuje aplikace přes OAuth 2.1 s PKCE (povinně S256). Tokeny se ukládají jen jako SHA-256 otisky a otevírají jen adresu NagataPress.
- Každou aplikaci povolí administrátor na souhlasové obrazovce ve wp-admin a kdykoli ji může odpojit.
- Text položky je obsah k překladu, nikdy pokyn, a každý překlad se před uložením kontroluje.
- Jména, adresy, e-maily a telefony zákazníků z košíku, pokladny a e-mailů asistent nikdy nedostane.
- Skill je čistý text: žádné skripty, klíče ani data z webu.
