# Agent instructions

## Layout

- `TEMPLATE.md` — base template every recipe copies.
- `foods/`, `sauces/`, `cocktails/`, `meads/` — one Markdown file per recipe, plus a `README.md` index.

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
   - Ingredients: one line each, `quantity ingredient, prep`. Keep the source's units; add a
     conversion only if the source gives one. Group with bold sub-labels only when the recipe has
     genuinely separate components (crust and filling, dough and sauce).
   - Steps: imperative, one action per step, no step numbers in the text. Merge trivial steps.
     Write them in our own words rather than copying the source's prose.
   - Notes: only substitutions, make-ahead or warnings that change the outcome — otherwise delete
     the section.
   - Delete every leftover `{placeholder}` and template comment.
4. Update the indexes — see below. This is part of adding the recipe, not a follow-up.

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

Before you call a task done, verify it. Both checks below should print nothing but their headings —
they skip `TEMPLATE.md` and `AGENTS.md`, whose links are placeholders and examples by design.

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
```

## Style

Brevity is the point. If a line doesn't change what the cook does, drop it. No preamble, no
narrative, no filler adjectives, no emoji. A simple recipe should land well under 40 lines; a long
one earns its length with steps and ingredients, never with prose.
