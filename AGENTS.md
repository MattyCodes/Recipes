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
     fit, delete the rest. Never invent a yield or a time.
   - Ingredients: one line each, `quantity ingredient, prep`. Keep the source's units; add a
     conversion only if the source gives one.
   - Steps: imperative, one action per step, no step numbers in the text. Merge trivial steps.
   - Notes: only substitutions, make-ahead or warnings that change the outcome — otherwise delete
     the section.
   - Delete every leftover `{placeholder}` and template comment.
4. Add a row to that directory's `README.md` index, alphabetical by recipe name, with a few words
   of description in the Notes column (and replace the `_nothing here yet_` row if it's still there).

## Style

Brevity is the point. If a line doesn't change what the cook does, drop it. A finished recipe should
read in under ~40 lines. No preamble, no narrative, no filler adjectives, no emoji.
