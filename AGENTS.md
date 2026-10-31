# Agent instructions

## Layout

- `TEMPLATE.md` — base template every recipe copies.
- `foods/`, `sauces/`, `cocktails/`, `meads/` — one Markdown file per recipe, plus a `README.md`
  index and an `images/` directory when a recipe has a picture.

## Adding a recipe from a URL

1. Fetch the URL. Read only the recipe: skip the life story, ads, SEO padding, "why you'll love
   this", comments, nutrition tables and popups. If the page has JSON-LD `Recipe` markup, use it.
2. Pick the directory — `foods/`, `sauces/`, `cocktails/` or `meads/`.
3. Copy `TEMPLATE.md` to `<dir>/<kebab-case-name>.md` and fill it in:
   - Breadcrumb: replace `{Category}` with the directory's title (Foods, Sauces, Cocktails, Meads).
   - **Source**: the URL, linked with the site's name.
   - Meta table: keep only rows the source actually gives, swap in the cocktail/mead rows where they
     fit, add one of your own if the dish needs it (Equipment, Chill, Rest). Never invent a yield or
     a time.
   - Ingredients: one line each, `quantity ingredient, prep`. Keep the source's units for amounts
     and add a conversion only if the source gives one. Group with bold sub-labels only when the
     recipe has genuinely separate components (crust and filling, dough and sauce).
   - Steps: imperative, one action per step, no step numbers in the text. Merge trivial steps.
     Write them in our own words rather than copying the source's prose.
   - Notes: only substitutions, make-ahead or warnings that change the outcome — otherwise delete
     the section.
   - Delete every leftover `{placeholder}` and template comment.
4. Update the indexes — see below. This is part of adding the recipe, not a follow-up.

## Page order

Every recipe page runs in this order. Whatever a recipe doesn't have is deleted, not left blank.

1. Breadcrumb
2. `# Title`
3. Badges, if the recipe is tagged
4. One- or two-line description
5. Info table
6. Image, if one was supplied
7. `## Ingredients`, `## Steps`, `## Notes`

## Images

When the user supplies an image for a recipe, it goes **below the description and the info table**,
immediately before `## Ingredients` — never at the top of the page, where it would push the
breadcrumb and the table out of view.

- Save it as `<category>/images/<kebab-case-name>.<ext>`, matching the recipe's filename. Create the
  `images/` directory if it isn't there yet.
- Reference it relatively: `![Recipe Name](images/recipe-name.jpg)`.
- Alt text is the recipe's name, not a description of the photo.
- One image per recipe. If the user supplies several, use the best one and leave the rest out.

## Badges

Badges are the tagging system. They are shields.io static images, which render in colour on GitHub
where inline HTML styling does not. A recipe may carry several, or none at all — most carry none.

Form, on both the recipe page and its index row:

```
![Dessert](https://img.shields.io/badge/Dessert-7C3AED?style=flat)
```

- On the page: on their own line directly under the `# Title`, separated by spaces when there are
  several, before the description.
- In the index: directly after the recipe's link in the `Recipe` column of its category README.
- The two must agree. A badge on the page but not the index row (or the reverse) is a bug — the
  check below catches it.

### Registry

Every tag and its colour. Use these exactly; don't improvise a shade for a tag already listed.

| Tag | Colour | Badge |
| --- | ------ | ----- |
| Bread | `38BDF8` | ![Bread](https://img.shields.io/badge/Bread-38BDF8?style=flat) |
| Dessert | `7C3AED` | ![Dessert](https://img.shields.io/badge/Dessert-7C3AED?style=flat) |

Adding a tag: pick a colour distinct from the ones above, add the row here first, then apply it.
Keep tags single words where possible; a tag with a space is written `Two%20Words` in the URL.

## Temperatures

Always give both scales, wherever a temperature appears — meta table, steps, notes. Put the source's
own figure first and the conversion after it, separated by a slash:

- `160°C / 320°F`
- `375°F / 190°C`

Convert with `°F = °C × 9/5 + 32`, and round to the nearest 5° unless the source states something
precise (a candy or meat thermometer target, say, where 71°C / 160°F must stay exact). If the source
already prints both, keep its pairing rather than recomputing. This is the one exception to keeping
the source's units: ingredient amounts stay as written, temperatures always carry both.

## Indexes and breadcrumbs

The repo is a catalogue: every page must be reachable by clicking, and every link must resolve.
Treat these as invariants and leave them true at the end of every task.

- Each recipe opens with `[← Recipes](../README.md) · [← <Category>](README.md)` as its first line.
- Each category `README.md` opens with `[← Recipes](../README.md)`, then its title, one-line blurb,
  and an index table of `| Recipe | Notes |` rows.
- The root `README.md` index lists every category directory.
- Every recipe file appears exactly once in its category's index, linked by filename, sorted
  alphabetically by recipe name, with a few words in the Notes column.
- Every index row points at a file that exists. No orphan recipes, no rows pointing at nothing.

So whenever you add, rename, move or delete a recipe, update the affected index in the same change.
Adding the first recipe to a directory replaces its `_nothing here yet_` row; emptying a directory
restores it. A new category directory needs its own `README.md` with the breadcrumb and table, plus
a row in the root index.

Before you call a task done, verify it. All four checks below should print nothing but their
headings — they skip `TEMPLATE.md` and `AGENTS.md`, whose links, badges and placeholders are
examples by design.

```bash
# every relative link resolves
find . -name '*.md' ! -name TEMPLATE.md ! -name AGENTS.md -not -path './.git/*' | while read -r f; do
  d=$(dirname "$f")
  grep -o '](\([^)#]*\))' "$f" | sed 's/](//;s/)$//' | grep -v '^http' | while read -r l; do
    [ -e "$d/$l" ] || echo "BROKEN  $f -> $l"
  done
done

# every recipe is indexed, and every index row has a file
for d in foods sauces cocktails meads; do
  find "$d" -name '*.md' ! -name README.md | while read -r r; do
    grep -q "($(basename "$r"))" "$d/README.md" || echo "UNINDEXED  $r"
  done
done

# every temperature carries both scales
grep -rn '°[CF]' --include='*.md' . | grep -v TEMPLATE.md | grep -v AGENTS.md \
  | grep -vE '°C / [0-9]+°F|°F / [0-9]+°C' | sed 's/^/UNCONVERTED  /'

# a recipe's badges match its index row
tags() { grep -o 'badge/[^-]*-[0-9A-Fa-f]\{6\}' | sed 's|badge/||;s|-[0-9A-Fa-f]\{6\}$||' | sort -u | tr '\n' ' '; }
for d in foods sauces cocktails meads; do
  find "$d" -name '*.md' ! -name README.md | while read -r r; do
    page=$(tags < "$r")
    row=$(grep -F "($(basename "$r"))" "$d/README.md" | tags)
    [ "$page" = "$row" ] || echo "BADGE MISMATCH  $r  page:[$page] index:[$row]"
  done
done
```

## Style

Brevity is the point. If a line doesn't change what the cook does, drop it. No preamble, no
narrative, no filler adjectives, no emoji. A simple recipe should land well under 40 lines; a long
one earns its length with steps and ingredients, never with prose.
