# Fields

**Status:** Draft · **Prefix:** FLD

## Purpose

Field types, the shape of their values, and how values are checked and normalised on every write.

## Requirements

| ID | Requirement | Verified by |
|---|---|---|
| FLD-1 | Types: `text`, `number` (settings `kind`: `int`\|`float`, `decimals` 0–8, `display`: `number`\|`percent`), `boolean`, `date`, `date_time`, `select`, `multi_select`, `rating` (1–5), `currency`, `full_name`, `emails`, `phones`, `links` (setting `form`: `url`\|`domain`), `address`, `rich_text`, `files`, `array` (strings), `json`, `uuid`, `relation`, `actor`. Display settings never change stored values. | percent field 0.25 → stored 0.25 |
| FLD-2 | Composite values: currency `{amount_micros (integer, 10⁻⁶ units), currency_code (ISO 4217)}`; full name `{first, last}`; emails `{primary, additional []}`; phones `{primary {number, country (ISO 3166-1 alpha-2), calling_code}, additional [same]}`; links `{primary {url, label}, additional [same]}`; address `{street1, street2, city, postcode, state, country, lat, lng}`; rich text `{markdown, blocks}`; actor `{source, member_id?, name, context?}`. Unknown sub-fields → `unknown_field`. | currency `{amount: 5}` → `unknown_field` |
| FLD-3 | Empty means null: `""` in text and `[]` in lists are stored as null; a single string sent to an array is wrapped. Unknown or inactive field → `unknown_field`; null for a required field with no default → `required`. | create opportunity with `stage: null` → `required` |
| FLD-4 | Emails: every address is an addr-spec with a dotted domain (else `invalid_email`), lower-cased, trimmed; `additional` loses duplicates and the primary. | `Ann@X.com` → `ann@x.com` |
| FLD-5 | Links: blank entries dropped; an empty primary takes the first additional; invalid URL → `invalid_url`. `url` form: scheme and host lower-cased, trailing `/` dropped when there is no query or fragment. `domain` form: host only (no scheme, path, user, port, leading `www.`, trailing dot), lower-cased, IDN to punycode. | `https://WWW.Acme.com/about` in domain form → `acme.com` |
| FLD-6 | Phones: parsed per E.164; `country` and `calling_code` filled from the number when missing; a number, country or calling code that is invalid or contradicts the others → `invalid_phone`. | `+33 6 12 34 56 78` → country `FR`, calling code `+33` |
| FLD-7 | Rich text: a write may give `markdown` or `blocks`; the other is derived and both stored. | write markdown only → blocks returned |
| FLD-8 | Select and rating values must be an option value; multi-select values a set of option values (duplicates removed). Else `invalid_option`. | select `maybe` with no such option → `invalid_option` |
| FLD-9 | Currency: amounts are integer micros; a non-null amount needs a `currency_code` (default: field default, else workspace `default_currency`). | amount 1.5 → `amount_micros` 1500000 |
| FLD-10 | Files: value `[{file_id, name}]`, at most the field's `max_files` (1–60). Uploads: > 0 bytes and ≤ 1 GB (instance setting), images for avatars ≤ 10 MB; file names non-empty without `/` or `\`; SVGs sanitised and ≤ 3 MB; only image, PDF, plain-text, audio and video types are served inline, the rest as downloads. Stored files are removed only when their record is destroyed. | upload `a/b.pdf` → refused |
| FLD-11 | Actor sources: `manual`, `api`, `import`, `workflow`, `webhook`, `email`, `calendar`, `system`. The server sets `updated_by` on every write and `created_by` on create from the caller: member → `manual` + member name and id; API key → `api` + key note; workflow → `workflow` + workflow name ([WF-12](workflows.md)); mail sync → `email`\|`calendar` + account owner. A client may only mark its own creates as `import`. | API key creates person → `created_by.source` = `api` |

## Out of scope

- Formula and computed fields, field-level validation rules, encrypted fields.

## Decisions

- Raw `numeric` (arbitrary precision) dropped; `number` with `kind` covers it.
- Clients can't forge actor sources other than `import`; the source kept any client-sent source.
