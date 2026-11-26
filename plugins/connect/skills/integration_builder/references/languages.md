# Widget languages

> Read this only when the user asks for the integration in another language, or asks to change or remove a language.

A widget can carry its own text in other languages: one translation per language, applied when a customer's widget is set to that language (`lang` on the embed element or hosted iframe URL, v1's `locale` also accepted, `language` in the SDK). A missing key always shows the widget's own English text.

**The hub header is never translated.** Its title, subtitle and menu show the partner's own text in every language, so there is nothing to translate there and no per-language header setting.

**fastn already translates the hub's own text** (buttons, headings, dialogs, errors, the config editor and the assistant) into 22 languages: da, de, es, fi, fr, it, ja, ko, nb-no, nl, pt-br, pt-pt, sv, zh-cn, zh-hans, zh-tw, pl, tr, ar, he, hi, cs. Nothing to do for that. A translation here covers only the integration's own text:

| Key | What it is |
|---|---|
| `name`, `description`, `documentation` | The widget's name, description and How it works doc |
| `connect.label`, `connect.doneLabel`, `buttons.disconnect.label`, `buttons.configure.label` | The hub's button labels, only when the user wants their own wording instead of fastn's translation |
| `connectors.<connectorSlug>` | An app's `name`, `description`, `authFields.<field>.label` / `placeholder`, `submitLabel` |
| `triggerCategoriesTitle`, `triggerCategories.<slug>` | The sync-type choice and each type's `label` / `description` |
| `triggers.<triggerId>` | A trigger's `label` and `cronPresets.<cronExpression>` labels |
| `form` | The settings form: `title`, `description`, `submitLabel`, `fields.<key>` (`label`, `placeholder`, `help`, `options.<value>`) |
| `unified.<entityId>.<connectorSlug>.fields.<key>` | Unified per-provider form fields |
| `workflows.<workflowId>` | A workflow's `name` / `description` as customers see it listed |
| `configs.<templateConfigId>.entities.<entityId>` | Config form text (`title`, `fields.<fieldId>`) and sync text (`source.label`, `target.label`, `mappings.<mappingId>`, `decisions.<decisionId>`, `conditions.<field>`, `customFields.<customPropertyName>`) |

Keys are ids, never positions, so reordering never moves a translation onto the wrong item.

## Tools

- `fastnPlatform__listWidgetTranslations` { id }: the languages the widget has.
- `fastnPlatform__getWidgetTranslation` { id, lang }: one language's text, in the shape above. 404 when the language has none.
- `fastnPlatform__putWidgetTranslation` { id, lang, body }: **replaces the whole language.** To change one label, read the language first, change what the user asked, and send the result back. 64 KB per language.
- `fastnPlatform__deleteWidgetTranslation` { id, lang }: removes a language. Customers set to it go back to the English text.

## Rules

- Use the user's own wording. Never invent translations they did not give you or ask for.
- `en` is the widget's own text and is refused (`SOURCE_LANG`): change English with `fastnPlatform__updateWidget`.
- Delete a language only when the user asks for that language to be removed.
- After a write, read it back with `fastnPlatform__getWidgetTranslation` and confirm the keys you set are there.
