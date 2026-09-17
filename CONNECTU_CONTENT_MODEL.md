# ConnectU SA — Content model

Two sources feed this doc: fields **verified directly from working code**
(`src/lib/drupal.ts`, `src/components/SectionRenderer.astro` — already
built and build-tested), and fields **from the content architecture spec
you shared** (screenshots) that haven't been implemented yet. Each section
below says which is which — don't take the second kind as already working.

---

## Content type: Landing page

Verified — this is what the current frontend code actually expects.

| Field | Machine name | Type | Notes |
|---|---|---|---|
| Title | `title` | Text | |
| Page key | `field_page_key` | Text | Used to look up a page by slug — e.g. `home`, `careers`, `investor-relations` |
| Hero title | `field_hero_title` | Text | Falls back to `title` if empty |
| Hero subtitle | `field_hero_subtitle` | Text | |
| Hero CTA label | `field_hero_cta_label` | Text | |
| Hero CTA link | `field_hero_cta_url` | Link | |
| Hero image | `field_hero_image` | Media (image) | |
| SEO description | `field_seo_description` | Text | |
| Sections | `field_sections` | Entity reference revisions (Paragraphs), unlimited | See below — this is the page-builder field |
| Moderation state | *(built-in via Content Moderation)* | — | See workflow section below |

This is built with **Paragraphs**, not Layout Builder — see the earlier
discussion on why. An editor adds one or more sections, in order, from
the 4 types below.

## Paragraph types (page sections)

Verified — these 4 map exactly to `SectionRenderer.astro`'s 4 render
branches. The `type` values below are the actual Paragraph bundle machine
names the code checks for.

### `section_rich` — general text block

| Field | Machine name |
|---|---|
| Eyebrow (small label above title) | `field_section_eyebrow` |
| Title | `field_section_title` (supports basic HTML, e.g. a `<span>` for partial-color titles) |
| Body | `field_section_body` (long text, rich text editor) |

### `section_stats` — stat/number grid

| Field | Machine name |
|---|---|
| Title | `field_section_title` |
| Stats | `field_stats` — entity reference revisions → **`stat_item`** (nested Paragraph, below), unlimited |

### `section_cards` — card grid

| Field | Machine name |
|---|---|
| Eyebrow | `field_section_eyebrow` |
| Title | `field_section_title` |
| Body | `field_section_body` |
| Cards | `field_cards` — entity reference revisions → **`card_item`** (nested Paragraph, below), unlimited |

### `section_cta` — call-to-action band

| Field | Machine name |
|---|---|
| Title | `field_section_title` |
| Body | `field_section_body` |
| CTA label | `field_cta_label` |
| CTA link | `field_cta_url` |

## Nested Paragraph types

Verified — these are what make cards/stats fillable by a non-technical
editor (a form with labeled fields) instead of hand-typed JSON. Same
reasoning as Vodacom's `link_card`/`link_item` pattern: a repeating group
of *different* field types can only be modeled this way in Drupal.

### `card_item`

| Field | Machine name |
|---|---|
| Title | `field_card_title` |
| Body | `field_card_body` |
| Link | `field_card_href` |

### `stat_item`

| Field | Machine name |
|---|---|
| Label | `field_stat_label` — e.g. "Customers" |
| Value | `field_stat_value` — e.g. "237M" |
| Note | `field_stat_note` — optional, e.g. "+6.2% YoY" |

---

## From your spec, not yet implemented in code

Everything below is transcribed from the screenshots you shared. I
haven't built any of this yet — field lists marked "cut off in photo"
are genuinely incomplete and need confirming before building, not guessed.

### Content type: Basic pages

| Field | Notes |
|---|---|
| Title | Required |
| Body | Rich text editor |
| Publication date | |
| Language selection | |

Features: revisioning enabled, translation support, URL alias
configuration, basic access control. This maps cleanly onto Drupal's
standard Page content type + core Content Translation — no unusual setup
needed.

### Content type: Zones

Geographic zone definitions, for organizing regional content.

| Field | Notes |
|---|---|
| Zone name | Required |
| Description | |
| Active status | Boolean |
| Language selection | |

### Content type: Regions

| Field | Notes |
|---|---|
| Region name | Required |
| Description | |
| Active status | Boolean |
| Language selection | |

### Content type: Stores

Physical store locations — pairs with Regions above, likely for a store
locator feature (a map for this belongs in a lazy-loaded Astro island,
per your own frontend README's own performance note).

| Field | Notes |
|---|---|
| Store name | Required |
| Address | Structured: street, city, postal code |
| Geographic coordinates | Latitude/longitude — for map placement |
| *(cut off in photo — screenshot ended here)* | **Needs confirming**: likely candidates are phone number, opening hours, and a reference field back to its Region/Zone, but don't build against a guess — send the rest of this field list or a fresh screenshot before this content type gets built |

### Content status workflow

A 5-state moderation workflow, more granular than Vodacom's simpler
Draft → Published:

```
Draft → Ready for Review → Approved → Published → Archived
```

This implies an actual approval chain (author drafts, a separate reviewer
approves, a third step publishes) rather than one editor self-publishing.
Set up via Drupal core's **Content Moderation** module with a custom
workflow (the default "Editorial" workflow only has Draft/Published/
Archived — this needs 2 extra states added). Where the **rebuild webhook**
fires matters here: it should trigger on the transition into
**Published** specifically, not on every state change, same principle as
Vodacom's setup.

### Translation strategy

| Piece | Module |
|---|---|
| Content translation | Content Translation (Drupal core) |
| Interface translation | Core |
| Menu translation | Core |
| Taxonomy term translation | Core |
| Block translation | Core |
| URL path translation | Language-prefixed paths (`/fr/...` etc.) |

All core, no contrib modules needed — matches the multilingual approach
already proven on the Vodacom project.

---

## Open questions before building the "not yet implemented" section

1. The full Stores field list (see the cut-off note above)
2. Does a Store need a reference to both a Zone *and* a Region, or just one?
3. Should Zones/Regions/Stores be translatable per-field, or are they
   largely language-neutral (a store's coordinates don't change by
   language, but its description might)?
