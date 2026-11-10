# Article Pagination and Contents

A standalone ProcessWire Textformatter for plain Textarea, TinyMCE, and other editors backed by a Textarea field. It uses PHP 8.4 or newer with the DOM extension for HTML5 parsing.

## About
This Textformatter was created using ChatGPT Codex for a specific use I have. I prompted Codex to create a standalone version and add additional markup inline with Ryan Cramer's original Textformatter. [TextformatterPagination](https://processwire.com/modules/textformatter-pagination/)

## Requirements

- A ProcessWire site running PHP 8.4 or newer with the PHP DOM extension enabled.
- A ProcessWire Textarea field for the article. This can be a plain Textarea or a rich-text field using TinyMCE.

To split an article across multiple URLs, enable **Allow Page Numbers** in the template’s **URLs** settings. A single-page Contents box with heading links works without this setting.

## Installation

1. Install **Article Pagination and Contents** under Modules.
2. In the article field’s **Details** tab, add it under **Textformatters**. Render the formatted field value in the template.
3. To split articles across URLs, enable **Allow Page Numbers** under the template’s **URLs** settings. Use one paginated article field per template.
4. For the supplied styling, link to `article-pagination.css` from the template. The module’s classes can also be styled in the site stylesheet.

Put five or more hyphens on their own line or in their own rich-text paragraph. Add a title after the hyphens to name the next part. A marker in a heading keeps that heading visible on the next page.

```text
[contents]

Opening text

----- Across the river

The next part begins here.
```

Put `[contents]` on its own line or in its own paragraph at the spot where the box should appear. For a single-page article, it links to headings on the same page. On a paginated article, it can group heading links under part titles. To show the box on additional pages, add a marker within each part.

Use `[contents split=h2]` to start a new page at each top-level H2. Its text becomes the part title and the heading stays visible in the article. The first Contents marker chooses the split method for the whole article. It accepts `h1` through `h6` or `hyphens` (the default). A Contents marker before the first heading stays with that heading when there is no introductory text. Use the editor’s heading styles and enable **Allow Page Numbers** on the template.

```text
[contents pages=off headings=h2 width=medium align=right wrap=on]
```

The first marker controls `pages` (`on` or `off`) and `headings` (`all`, `h2`, `h3`, or `none`) throughout the article. Each marker can set its own `width` (`small`, `medium`, `large`, or `full`), `align` (`left`, `center`, or `right`), and `wrap` (`on` or `off`). Headings get generated anchors if they have no ID. Give a heading an explicit HTML ID if its incoming links need to survive renaming.

Module settings control the numbered pager, titled previous and next links, an optional list of part titles, the default heading levels, the Contents title, and automatic insertion when an article has no Contents marker. The stylesheet accepts `--article-contents-small`, `--article-contents-medium`, and `--article-contents-large` for widths, along with `--article-contents-padding`, `--article-contents-font`, `--article-contents-background`, and `--article-contents-border`.

Use `paginate=horizontal` on the first Contents marker for a row with Previous, page numbers, and Next. Long articles show the first and last numbers, a few near the current page, and an ellipsis wherever numbers are skipped. Use `named=off` to hide the separate links labelled with neighbouring part titles, or `named=on` to show them. With no options, the pager follows the module settings and uses the standard list markup.

```text
[contents split=h2 paginate=horizontal named=off]
```

Use `[contents paginate=off]` on the first Contents marker to suppress the entire generated footer. The article still splits into separate URLs and Contents still links to the parts.

## Copy-and-paste template examples

To use the supplied styles, put the stylesheet link in your template’s `<head>` and render the formatted field inside the article. Replace `body` with your field name.

```php
<link rel="stylesheet" href="<?= $config->urls->siteModules ?>TextformatterArticlePagination/article-pagination.css">
```

```php
<article><?= $page->body ?></article>
```

For your own styling, paste this into the site stylesheet. It works with the default pager markup too.

```css
.article-page-numbers {
    display: flex;
    flex-wrap: wrap;
    gap: .5rem;
    padding: 0;
    list-style: none;
}
.article-page-numbers a {
    display: inline-block;
    padding: .35rem .6rem;
    border: 1px solid currentColor;
}
.article-page-numbers a[aria-current="page"] {
    font-weight: bold;
    background: #ececec;
}
.article-contents {
    padding: 1rem;
    background: #f8f8f4;
    border: 1px solid #ddd;
}
```

To place the pager outside the article, pass the formatted field to `splitOutput()`. Print the body and pager where the template needs them. The Contents box stays inside the body, and the pager is empty on single-page articles.

```php
<?php
$formatter = $modules->get('TextformatterArticlePagination');
$article = $formatter->splitOutput((string) $page->body);
?>
<article><?= $article['body'] ?></article>
<?= $article['pager'] ?>
```

Read the formatted field first, then call `getPartInfo()` with the page and field name. It returns the current part number, total part count, and plain-text part title, including when `paginate=off`. An ordinary article defaults to part 1 of 1 with the ProcessWire page title. Escape the title before inserting it into HTML.

```php
<?php
$formatter = $modules->get('TextformatterArticlePagination');
$body = $page->body;
$part = $formatter->getPartInfo($page, 'body');
?>
<?php if($part['count'] > 1): ?>
    <p>Part <?= $part['number'] ?> of <?= $part['count'] ?> · <?= $sanitizer->entities($part['title']) ?></p>
<?php endif; ?>
<article><?= $body ?></article>
```

## Migrating from TextformatterPagination

The five-hyphen page-break convention comes from Ryan Cramer’s [TextformatterPagination](https://processwire.com/modules/textformatter-pagination/). Existing five-hyphen breaks with optional titles can stay in the article.

1. Check that the server has PHP 8.4 or newer with DOM and keep **Allow Page Numbers** enabled on the template.
2. Replace the old formatter on the field and check how the new module settings and CSS classes fit your template.
3. Replace legacy `pagination-off` with `paginate=off` on the first Contents marker. To bypass formatting for an entire field, remove the new formatter from that field. Review any articles that used `pagination-off-all` as a per-article opt-out.
4. Update templates that read `$page->TextformatterPagination` to use `getPartInfo()` after reading the formatted field.
