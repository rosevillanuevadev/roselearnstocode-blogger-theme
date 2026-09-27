# Rose Learns to Code Blogger Theme: Forensic Build Documentation

**What it is:** A dark, terminal-inspired customization of the Delilah Blogger theme, built for a computer science study journal that publishes math, code, and diagrams. Includes a custom navigation menu with an auto-updating Studies dropdown and five sidebar widgets.

**Live site:** https://roselearnstocode.com (Blogger)
**Repository:** https://github.com/rose2023va/roselearnstocode-blogger-theme
**Built:** May 21 to 23, 2026 · **Documented:** September 27, 2026
**Author:** Rose

---

## 1. How this build works

A Blogger theme is one XML file. Three parts of that file matter here:

- **`<b:skin>`** holds the theme's CSS inside a CDATA block. Blogger "Theme Designer" colors and fonts are declared as `<Variable>` tags at the top of the skin and referenced inside the CSS.
- **The `<head>`** is where external libraries (KaTeX, Prism.js, Mermaid.js) are loaded, right after `<b:include data='blog' name='all-head-content'/>`.
- **`<b:section>` and `<b:widget>`** blocks define page regions and gadgets. `<b:class cond='...'>` tags on `<body>` add CSS classes depending on the page type (`data:view.isHomepage`, `data:view.isPost`, and so on).

The build follows one rule: **never rewrite the base theme's CSS.** Every visual change lives in one override layer appended at the end of the skin, under a comment banner `ROSE CUSTOMIZATIONS`. Template edits are kept to a short, documented list. This keeps the customization portable and makes it easy to see what changed.

**Base theme:** Delilah by November Dahlia (novemberdahlia.etsy.com), a paid Etsy theme. It is **not** included in the repository. You need your own copy.

---

## 2. What you need

- A Blogger blog and a Google account
- Your own purchased copy of the Delilah theme XML
- A desktop browser with DevTools (Chrome or Firefox). Most of this build was debugged by inspecting computed styles, not by guessing.
- The files in this repository

---

## 3. Design tokens

| Token | Hex | Used for |
|---|---|---|
| navy | `#0a192f` | Page background, header, inputs |
| navy-lt | `#112240` | Nav bar, cards, code blocks, entry footer |
| navy-mid | `#1d3557` | Borders, dividers, Prism labels |
| slate | `#8892b0` | Dates and muted text |
| slate-lt | `#a8b2d8` | Post titles in lists, widget text |
| white | `#e6f1ff` | Primary text, headings |
| coral | `#FF6B6B` | Accent: links, hover, borders on code |
| coral hover | `#FF8E8E` | Link and icon hover |

**Type:** Fira Code (site title, nav, post list, widgets, code) and Raleway (loaded for headings where the theme allows). The theme's own Quattrocento Sans, Oswald, and Sorts Mill Goudy remain loaded for any element not overridden.

---

## 4. Step by step

### Step 0: Back up

Blogger → Theme → the arrow beside Customize → **Backup**. Keep the download. Every step below edits the theme XML (Theme → arrow → **Edit HTML**), and a backup is the only undo.

### Step 1: Install the base theme

Theme → arrow → **Restore** → upload your Delilah `.xml`. Confirm the blog renders before changing anything.

### Step 2: Add the fonts

Find the theme's Google Fonts `<link>` near the top of the file and append Raleway and Fira Code to its `family=` list. The exact line used is in `theme-edits/google-fonts-link.xml`:

```
...Oswald:400,400italic|Sorts+Mill+Goudy:400,400italic|Raleway:300,400,500,600,700|Fira+Code:300,400,500
```

### Step 3: Set the Theme Designer variables

Replace the color and font `value` (and `default`) attributes of the `<Variable>` tags with the palette above. All 94 edited variable lines are in `theme-edits/variables.xml` so you can compare them against yours line by line. Two worth calling out:

```xml
<Variable name="blog.title.font" ... value="300 normal 28px Fira Code, monospace"/>
<Variable name="blog.title.font.size" ... value="22px"/>   <!-- mobile title -->
```

### Step 4: Load math, code, and diagram libraries

Paste the contents of `head/head-includes.xml` on the line **directly after** `<b:include data='blog' name='all-head-content'/>`. It contains three blocks:

**Mermaid 10.6.1** with `theme: 'base'` and `themeVariables` mapped to the palette (coral lines and borders, navy fills, Fira Code labels), plus a `.mermaid` container style with a coral left border.

**Prism.js 1.29.0** with the Okaidia theme and five plugins: line numbers, toolbar, copy to clipboard, show language, and autoloader. The autoloader fetches language grammars on demand from jsDelivr, so any `language-xxx` class works without listing languages up front:

```javascript
Prism.plugins.autoloader.languages_path = 'https://cdn.jsdelivr.net/npm/prismjs@1.29.0/components/';
```

A style block then repaints Prism to the palette: navy background, coral left border, Fira Code at `.85rem`, and coral inline code.

**KaTeX 0.16.9** with the auto-render extension and a custom renderer for posts migrated from WordPress. The old WordPress KaTeX plugin stored equations as HTML blocks, so the script finds and renders those directly, then runs auto-render for new posts:

```javascript
document.querySelectorAll('.wp-block-katex-display-block pre').forEach(function(el) {
  katex.render(el.textContent, el.parentNode, { displayMode: true, throwOnError: false });
});
document.querySelectorAll('.wp-block-katex-inline-block').forEach(function(el) {
  katex.render(el.textContent, el, { displayMode: false, throwOnError: false });
});
renderMathInElement(document.body, {
  delimiters: [
    {left: "$$", right: "$$", display: true},
    {left: "\\(", right: "\\)", display: false},
    {left: "\\[", right: "\\]", display: true}
  ],
  throwOnError: false
});
```

**Blogger XML rules that bite here:** attributes use single quotes; external scripts must be self-closed (`<script src='...'/>`); inline JavaScript must be wrapped in `//<![CDATA[` and `//]]>` or Blogger rejects characters like `<` and `&`.

### Step 5: Give every list page the no-sidebar class

The theme only gives archive, label, and search pages a `remove-sidebar` class. Replace the homepage `b:class` line with the four lines in `theme-edits/body-classes.xml`:

```xml
<b:class cond='data:view.isHomepage' name='home blog remove-sidebar'/>
<b:class cond='data:view.isMultipleItems and !data:view.isHomepage' name='archive remove-sidebar'/>
<b:class cond='data:view.isLabelSearch' name='label remove-sidebar'/>
<b:class cond='data:view.isSearch' name='search remove-sidebar'/>
```

Result: every list page (home, page 2 and beyond, labels, search) is full width with no sidebar. Single posts keep the sidebar because they never get this class.

### Step 6: Strip images and excerpts from the post list

In the Blog widget's post loop, delete the two featured image blocks and the excerpt plus jump link. These are the exact removed lines:

```xml
<b:if cond='data:post.featuredImage'>
  <div class='entry-thumbnail'>...<img expr:src='data:post.featuredImage'/>...</div>
</b:if>

<div class='entry-excerpt'><b:eval expr='data:post.snippets.long snippet {length: 180, linebreaks: false }'/></div>
<b:include data='post' name='postJumpLink'/>
```

The CSS layer also hides these classes, so either change alone works. Doing both keeps the page lighter.

### Step 7: Turn off the Featured Post widget

Set `visible='false'` on `<b:widget id='FeaturedPost1' ...>`. The widget was pinned to a specific post and pulled it out of the homepage list, which made the post count per page look wrong. The CSS layer also sets `.FeaturedPost { display: none }`.

### Step 8: Paste the CSS override layer

Paste all of `css/rose-customizations.css` at the **end** of the skin, immediately before `]]></b:skin>`. Being last in the cascade, plus `!important`, lets it win over the base theme without editing it. The layer does the following, in file order:

1. **Site title:** Fira Code 300, lowercase.
2. **Hide the original nav** (`nav.main-menu-wrapper { display: none }`), replaced in Step 9.
3. **Content spacing:** `.content-wrapper { padding-top: 2rem }` below the nav.
4. **Hide related-post images**, which render as broken boxes on posts without a featured image.
5. **Sticky footer** on short pages:
   ```css
   #wrapper { min-height: 100vh; display: flex; flex-direction: column; }
   .content-wrapper { flex: 1; }
   ```
6. **Palette overrides** for backgrounds, borders, header, links, meta, footer, sidebar, search, entry footer, pager, scroll-to-top, text selection, submenus, inputs, comments, post navigation, related posts, and the scrollbar.
7. **Hide** the Featured Post widget, popular-post thumbnails, and the Instagram section.
8. **Post list layout** (the core of the design, explained in Section 5): one line per post, title left, date right, a faint bottom rule.
9. **Title and date type:** titles in Fira Code 300, 13px, capitalized (11px on mobile); dates in Fira Code `.72rem`, `#8892b0`, never italic.
10. **Pager:** remove the theme's italics.
11. **Alignment:** `.content-wrap` side padding set to `20px` to match the nav.

### Step 9: Install the custom navigation

Paste `nav/custom-nav-menu.html` directly **before** `<nav class='main-menu-wrapper'>`. The original nav stays in the file (so theme updates don't break) but is hidden by Step 8.

The menu is Home, Journal, Studies (dropdown), About. The Studies dropdown builds itself from the blog's own labels:

1. Loads the public feed as JSONP: `/feeds/posts/default?max-results=500&alt=json-in-script&callback=rlcNavLabels`.
2. For every post, records each label with the newest publish date it appears on.
3. Drops labels in `EXCLUDE = ['Journal', 'Study Notes']`.
4. Sorts by most recently used and renders links to `/search/label/<name>`.

Publishing a post with a new label adds that label to the menu with no code change. The bar is centered with `max-width: 700px` to match the content column.

### Step 10: Clean up footer and Instagram

- The Instagram gadget content was replaced with an empty comment.
- The designer credit line in the footer was removed in this build. **Check your Delilah license before doing the same.** Many Etsy theme licenses require the credit unless you buy a removal option. If yours does, keep it.

### Step 11: Add the sidebar widgets

Layout → a sidebar section → **Add a Gadget** → **HTML/JavaScript**. Paste one file per gadget and put the widget title in the gadget's title field (the files contain no titles of their own).

| File | What it shows | How it works |
|---|---|---|
| `widgets/goodreads-bookshelf.html` | Last 5 books on the "read" shelf | Goodreads custom widget: static HTML fallback plus Goodreads' script. CSS scoped to the widget ID repaints it to the palette and hides the header, ratings, tags, and logo. |
| `widgets/goodreads-currently-reading.html` | Currently reading shelf | Same approach, different widget ID |
| `widgets/latest-posts.html` | 20 newest post titles | Blogger JSON feed with a JSONP callback, `max-results=20`, titles only |
| `widgets/currently-studying.html` | 3 most recently used labels | Scans the newest 20 posts, keeps each label the first time it appears, skips `EXCLUDE = ['Journal']`, stops at 3 |
| `widgets/cheatsheet-links.html` | Links to the 3 cheat sheet pages | Static list styled like the others |

**Goodreads note:** generate your own widget code at goodreads.com (Widgets page) and keep only the CSS from these files. The widget ID in the class names (for example `1779463554`) is unique to each generated widget, so update the CSS selectors to match yours.

Every widget uses the same style so the sidebar reads as one system: Fira Code 300, `.75rem`, `#a8b2d8` text, `#1d3557` dividers, coral on hover, `font-style: normal` (the theme italicizes gadget links by default).

### Step 12: Publish the cheat sheet pages

The files in `cheatsheets/` are Blogger post drafts written in HTML: LaTeX notation, Prism code blocks, and Mermaid diagrams. Create each as a **Page** (Pages → New page → HTML view → paste). The links widget points to `/p/code-block.html`, `/p/latex.html`, and `/p/mermaid.html`.

### Step 13: Blogger settings

- **Posts per page:** Settings → Posts → set the number. It is a maximum, not a guarantee (see Known limitations).
- **Comments:** set Comment Location to **Full page** or **Pop-up window**. The embedded form fails with "An error occurred while trying to allow cookies" in browsers that block third-party cookies.
- **Custom domain:** at the registrar, use the registrar's own DNS, add Blogger's four A records (`216.239.32.21`, `216.239.34.21`, `216.239.36.21`, `216.239.38.21`), a `www` CNAME to `ghs.google.com`, and the verification CNAME Blogger shows when you first try to save the domain. Then enable **Redirect domain** and **HTTPS redirect**.

---

## 5. Debugging record

These are the problems hit during the build and what actually fixed them. Most were caused by base-theme rules that were invisible until checked in DevTools.

**Date sat next to the title instead of at the right.** The flex container was on the wrong element. The real markup is `.entry-wrap > .entry-text > .entry-header > (.meta-before-title, h2.entry-title)`: the date comes first in the DOM. Fix: make `.entry-header` the flex row with `justify-content: space-between`, give the title `order: 1; flex: 1` and the date `order: 2; flex-shrink: 0`, and give `.entry-text` `flex: 1` so the header can span the full width.

**Title pushed inward on mobile.** Two causes: a `gap: 2rem` on `.entry-header`, and the theme's hardcoded `.content-wrap { padding-left: 1.5em; padding-right: 1.5em }`. Fix: remove the gap, set the padding to `20px` to match the nav.

**Huge vertical gaps between posts.** The theme sets `.blog-posts .entry { margin: 0 0 60px }` (made for image cards) and `.entry-text .entry-header { margin: 0 0 18px }`. Fix: zero the first, set the second to `0 0 5px`, and hide `.entry-footer`, which has its own background and padding.

**Page 2 of the homepage showed the sidebar and old card layout.** Every list rule was scoped to `.home`, which only matches page 1. Fix: add `remove-sidebar` to all list views (Step 5) and scope every list rule to `.remove-sidebar` instead of `.home`.

**Label and search pages ignored the flex layout.** The theme gives them a `grid-layout` class with `.grid-layout .blog-posts { display: grid; grid-template-columns: repeat(3, 1fr) }`, which won on specificity. Fix: target `body.label-page`, `body.archive-page`, and `body.search-page` explicitly and reset the grid. The divider then vanished inside the grid context, so the border moved from `.entry-wrap` to `.entry`.

**Nav and content edges didn't line up.** `#main-menu-wrap` used `max-width: calc(1150px - 40px)` while the content column was `700px`. Fix: the custom nav uses the same `700px` column, centered.

**Dates rendered in italics.** The theme italicizes meta text, and the first fix only targeted `.home`. Fix: set `font-style: normal` on `.meta-before-title` and all of its children for every list page.

**Pagination showed an inconsistent number of posts** (14 when set to 20). Not a theme bug. Posts imported from WordPress in one batch shared identical publish timestamps, and Blogger's paging is unreliable with duplicate timestamps. Fix: give those posts unique publish times in the post editor.

---

## 6. Writing posts with this theme

Always switch the editor to **HTML view** to insert these.

```html
<!-- Display math and inline math -->
$$A \cap B = \{x : x \in A \land x \in B\}$$
<p>Inline: \(E = mc^2\)</p>

<!-- Code with line numbers -->
<pre class="line-numbers"><code class="language-python">
def factorial(n):
    return 1 if n <= 1 else n * factorial(n - 1)
</code></pre>

<!-- Diagram -->
<div class="mermaid">
graph TD
  A[Start] --> B{n even?}
  B -->|Yes| C[Even]
  B -->|No| D[Odd]
</div>
```

Escape `<` and `>` inside code blocks as `&lt;` and `&gt;`. A stray `<` is the most common reason a code block breaks.

---

## 7. Verification checklist

- [ ] Homepage, page 2, a label page, and search results all show one line per post with the date on the right and no sidebar
- [ ] A single post shows the sidebar with all widgets in Fira Code, no italics
- [ ] Nav shows Home, Journal, Studies, About, and Studies lists your labels
- [ ] A post with `$$...$$` renders math, a `language-python` block shows line numbers and a copy button, a `.mermaid` block draws a diagram
- [ ] Footer sits at the bottom on a label page with one post
- [ ] Mobile width: title and date align with the nav edges

---

## 8. Known limitations

- Blogger URLs always end in `.html`, and static pages always live under `/p/`. Neither can be changed.
- The Studies dropdown reads at most 500 posts from the feed.
- Libraries load from jsDelivr. If the CDN is blocked, posts still render as plain text and code.
- Goodreads widgets depend on Goodreads' script and your shelf being public.
- Theme updates from the designer will not include these edits. Re-apply them from this repository.

---

## 9. Repository map

```
head/head-includes.xml         Mermaid, Prism, KaTeX (Step 4)
theme-edits/google-fonts-link.xml
theme-edits/variables.xml       Theme Designer values (Step 3)
theme-edits/body-classes.xml    remove-sidebar on list pages (Step 5)
css/rose-customizations.css     The override layer (Step 8)
nav/custom-nav-menu.html        Custom nav with Studies dropdown (Step 9)
widgets/                        Five sidebar gadgets (Step 11)
cheatsheets/                    LaTeX, code, diagram reference pages (Step 12)
screenshots/                    Real captures of the live blog
```
