---
name: magento2-implementation-guard
description: >
  Guards every Magento 2 development against mandatory
  implementation rules.
  Use this skill on every Magento 2 task — before writing code, to know
  the rules, and again before considering the task done, to verify the
  implementation respects them.
---

# Goal

Make sure any Magento 2 implementation respects the rules collected
here. These are not suggestions: a change that breaks one of them is
incomplete, even when the feature works.

This skill complements (it does not replace) `coding-style` and
`implementation-guard`. Apply all of them on a Magento 2 task.

## Translations

Every string passed through a Magento translate feature **must** have
a corresponding entry in the `i18n` folder of the module that declares
it.

A string that is translatable in code but missing from `i18n` looks
correct in `en_US` and silently stays untranslated in every other
locale, so this is checked on every change.

### Step 1: Detect every translatable string

Translation is not only `__()` in PHP. Look for all of these:

| Where            | What makes the string translatable         |
|------------------|--------------------------------------------|
| PHP              | `__('...')`                                |
| `.phtml`         | `<?= $block->escapeHtml(__('...')) ?>`     |
| `system.xml`     | `<label>` / `<comment>` under `translate=` |
| UI component XML | `translate="true"` items, `<label>` nodes  |
| Layout XML       | `<argument ... translate="true">`          |
| Menu / ACL XML   | `title` attributes                         |
| `.js` / `.html`  | `$.mage.__('...')`, `<!-- ko i18n: -->`    |
| Email templates  | `{{trans "..."}}`                          |

The rule applies to strings produced anywhere in the module: admin
labels, frontend messages, exception messages shown to a user,
validation errors, source model option labels, and column headers.

Placeholders stay part of the string. `__('Removed tag %1', $tag)`
is the single translatable string `Removed tag %1` — never split it
into concatenated fragments, because a translator cannot reorder
pieces that were already glued together.

### Step 2: Determine the project languages

Resolve the locale list from the project, in this order, and stop at
the first source that answers:

1. The locales already maintained by the project — list them with
   `find app/code -path '*/i18n/*.csv'` and use the set of file
   names found (e.g. `en_US.csv`, `es_ES.csv`).
2. The configured store locales — `general/locale/code` in
   `app/etc/config.php`, in a module `config.xml`, or in the
   `core_config_data` table.
3. Language packs required in the root `composer.json`.

If no language is specified anywhere, translate using **`en_US.csv`
as the default**, and create nothing else.

Prefer these static inspections over Magento CLI calls. If the locale
list still cannot be resolved, try 
the command `bin/magento info:language:list`,
but never block an implementation on that: fall back to `en_US.csv`.

### Step 3: Add the entry to every locale file

**File:** `app/code/Vendor/Module/i18n/<locale>.csv`

The file lives in the module that *declares* the string.

```csv
"A total of %1 order(s) have been updated.","A total of %1 order(s) have been updated."
```

Rules for the CSV:

* Two columns: the source string, then its translation. Both are
  always double-quoted, with no space after the comma.
* In `en_US.csv` the translation repeats the source string verbatim.
  This looks redundant and is still required: it is what makes the
  string appear in the dictionary and survive
  `bin/magento i18n:collect-phrases`.
* One row per string, no duplicates. Search the file before adding.
* A double quote inside a string is escaped by doubling it (`""`).
* Keep placeholders (`%1`, `%2`) identical in both columns.
* Create the `i18n` folder and the CSV when the module has none yet.

For every locale beyond `en_US`, add the same source string with the
actual translation for that language. Never leave a locale file
behind with fewer rows than `en_US.csv`.

### Step 4: Verify before finishing

Before considering the task done:

* Every new or modified translatable string in the diff has a row in
  `i18n/en_US.csv` of its own module.
* Every other locale file of that module got the same row, translated.
* Strings whose source text was **edited** had their old row updated,
  not duplicated — a renamed label leaves a stale key behind
  otherwise.
* Rows for strings removed by the change were deleted.
