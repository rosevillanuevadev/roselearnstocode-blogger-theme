# Case Study: A Study-Journal Theme for Rose Learns to Code

## Title

**Rose Learns to Code: Customizing a Commercial Blogger Theme into a Technical Study Journal**

## Description

A dark, terminal-inspired redesign of a purchased Blogger theme for a computer science study blog. The build adds math rendering (KaTeX), syntax-highlighted code (Prism.js), and diagrams (Mermaid.js), replaces the theme's navigation with a menu that builds its own Studies dropdown from post labels, restyles every list page into a one-line index, and adds five sidebar widgets. It shipped alongside a migration of the blog from paid WordPress hosting to free Blogger hosting.

**Live:** roselearnstocode.com · **Code:** github.com/rosevillanuevadev/roselearnstocode-blogger-theme · **Role:** Solo

## Project Scope (Problem)

I wanted a public study journal for computer science coursework. The blog had three problems:

- **Cost.** It ran on paid WordPress hosting that I could not justify keeping long term.
- **Content fit.** Discrete Mathematics notes need set notation and proofs, programming notes need readable code, and data structures need diagrams. The theme supported none of these.
- **Reading fit.** The theme was built for a lifestyle blog: large image cards, excerpts, and a 60px gap between posts. A study journal needs a scannable index where the title and date matter most.

## Constraints

- **No hosting budget.** The destination had to be Blogger, with its fixed XML template system and no server-side code.
- **Commercial base theme.** Delilah is a paid Etsy theme. I could customize my copy but not redistribute it, so every change had to be separable from the original.
- **Existing content.** 121 posts and 21 pages were migrating from WordPress, including math written with a WordPress KaTeX plugin that stored equations in a different HTML format.
- **Blogger platform limits.** No control over URL structure (`.html`, `/p/`), no plugins, and all JavaScript must run client-side from inside the template.

## Approach

**One override layer, not a rewrite.** Instead of editing the theme's 1,800 lines of CSS, every visual change lives in one block appended to the end of the stylesheet, where the cascade lets it win. Template edits were limited to a short list: body classes, the post loop markup, and one widget's visibility. This keeps the customization portable and lets it be published without the paid theme.

**Inspect before fixing.** Most layout bugs came from base-theme rules I could not see in the source I was reading: a hidden 60px entry margin, a grid layout applied only to label pages, a hardcoded padding on the content wrapper. Each fix started in browser DevTools, reading computed styles, and the causes are recorded in the build documentation.

**Let the content drive the navigation.** Rather than hardcoding a subject menu, the nav reads the blog's own label feed. Publishing a post with a new course label adds that course to the menu.

## Implementation

- **Libraries in the head:** Mermaid 10.6.1 themed through `themeVariables`, Prism.js 1.29.0 with line numbers, copy button, language labels, and an on-demand grammar autoloader, and KaTeX 0.16.9 with a custom renderer so migrated WordPress equations display without editing 121 posts.
- **List layout:** a flex row on the entry header (title left, date right, bottom-aligned) applied to every list view through a shared `remove-sidebar` class, with explicit overrides for the theme's grid on label and search pages.
- **Custom navigation:** JSONP call to the Blogger posts feed, collects labels with their newest post date, excludes non-course labels, sorts by recency.
- **Sidebar widgets:** Goodreads shelves restyled by widget-scoped CSS, latest 20 posts and currently studying (3 most recent course labels) from the Blogger JSON feed, and links to three reference pages.
- **Reference pages:** LaTeX, code block, and Mermaid cheat sheets published as Blogger pages for use while writing.
- **Platform fixes:** comment form switched to full page to avoid a third-party cookie failure; custom domain moved to Blogger's DNS records; duplicate import timestamps identified as the cause of inconsistent pagination.

## Result

- The blog moved off paid hosting; the remaining cost is the domain name.
- Every list page (home, pagination, labels, search) renders as the same one-line index, and single posts keep a sidebar.
- New posts can include rendered math, highlighted code with copy buttons, and diagrams using plain HTML in the Blogger editor.
- Migrated WordPress math renders without editing the posts.
- The Studies menu maintains itself as I add course labels.
- The customization is published as an open repository that installs onto any licensed copy of the theme.

## Stack

| Layer | Tools |
|---|---|
| Platform | Blogger (XML template, Theme Designer variables, HTML/JavaScript gadgets) |
| Styling | CSS override layer, flexbox, cascade and specificity management |
| Math | KaTeX 0.16.9 with auto-render and a custom WordPress-block renderer |
| Code | Prism.js 1.29.0 (Okaidia, line numbers, toolbar, copy, show language, autoloader) |
| Diagrams | Mermaid 10.6.1 |
| Data | Blogger JSON feed via JSONP, Goodreads widgets |
| Type | Fira Code, Raleway (Google Fonts) |
| Tooling | Browser DevTools, jsDelivr CDN, Namecheap DNS |
