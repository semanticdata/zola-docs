+++
title = "Shortcodes"
weight = 40
+++

Zola borrows the concept of [shortcodes](https://codex.wordpress.org/Shortcode_API) from WordPress.
In our case, a shortcode corresponds to a template defined in the `templates/shortcodes` directory or
a built-in one that can be used in a Markdown file. If you want to use something similar to shortcodes in your templates,
try [Tera macros](https://tera.netlify.com/docs#macros).

Broadly speaking, Zola's shortcodes cover two distinct use cases:

- Inject more complex HTML: Markdown is good for writing, but it isn't great when you need add inline HTML or styling.
- Ease repetitive data based tasks: when you have [external data](@/templates/overview.md#load-data) that you
  want to display in your page's body.

The latter may also be solved by writing HTML, however Zola allows the use of Markdown based shortcodes which end in `.md`
rather than `.html`. This may be particularly useful if you want to include headings generated by the shortcode in the
[table of contents](@/content/table-of-contents.md).

## Writing a shortcode

Let's write a shortcode to embed YouTube videos as an example.
In a file called `youtube.html` in the `templates/shortcodes` directory, paste the
following:

```jinja2
<div {% if class %}class="{{class}}"{% endif %}>
    <iframe
        src="https://www.youtube.com/embed/{{id}}{% if autoplay %}?autoplay=1{% endif %}"
        webkitallowfullscreen
        mozallowfullscreen
        allowfullscreen>
    </iframe>
</div>
```

This template is very straightforward: an iframe pointing to the YouTube embed URL wrapped in a `<div>`.
In terms of input, this shortcode expects at least one variable: `id`. Because the other variables
are in an `if` statement, they are optional.

That's it. Zola will now recognise this template as a shortcode named `youtube` (the filename minus the `.html` extension).

The Markdown renderer will wrap an inline HTML node such as `<a>` or `<span>` into a paragraph.
If you want to disable this behaviour, wrap your shortcode in a `<div>`.

A Markdown based shortcode in turn will be treated as if what it returned was part of the page's body. If we create
`books.md` in `templates/shortcodes` for example:

```jinja2
{% set data = load_data(path=path) -%}
{% for book in data.books %}
### {{ book.title }}

{{ book.description | safe }}
{% endfor %}
```

This will create a shortcode `books` with the argument `path` pointing to a `.toml` file where it loads lists of books with
titles and descriptions. They will flow with the rest of the document in which `books` is called.

Shortcodes are rendered before the page's Markdown is parsed so they don't have access to the page's table of contents.
Because of that, you also cannot use the `get_page`/`get_section`/`get_taxonomy` global functions. It might work while
running `zola serve` because it has been loaded but it will fail during `zola build`.

## Using shortcodes

There are two kinds of shortcodes:

- ones that do not take a body, such as the YouTube example above
- ones that do, such as one that styles a quote

In both cases, the arguments must be named and they will all be passed to the template.

Lastly, a shortcode name (and thus the corresponding `.html` file) as well as the argument names
can only contain numbers, letters and underscores, or in Regex terms `[0-9A-Za-z_]`.
Although theoretically an argument name could be a number, it will not be possible to use such an argument in the template.

Argument values can be of one of five types:

- string: surrounded by double quotes, single quotes or backticks
- bool: `true` or `false`
- float: a number with a decimal point (e.g., 1.2)
- integer: a whole number or its negative counterpart (e.g., 3)
- array: an array of any kind of value, except arrays

Malformed values will be silently ignored.

Both types of shortcode will also get either a `page` or `section` variable depending on where they were used
and a `config` variable. These values will overwrite any arguments passed to a shortcode so these variable names
should not be used as argument names in shortcodes.

### Shortcodes without body

Simply call the shortcode as if it was a Tera function in a variable block. All the examples below are valid
calls of the YouTube shortcode.

```markdown
Here is a YouTube video:

{{/* youtube(id="dQw4w9WgXcQ") */}}

{{/* youtube(id="dQw4w9WgXcQ", autoplay=true) */}}

An inline {{/* youtube(id="dQw4w9WgXcQ", autoplay=true, class="youtube") */}} shortcode
```

Note that if you want to have some content that looks like a shortcode but not have Zola try to render it,
you will need to escape it by using `{{/*` and `*/}}` instead of `{{` and `}}`.

### Shortcodes with body

Let's imagine that we have the following shortcode `quote.html` template:

```jinja2
<blockquote>
    {{ body }} <br>
    -- {{ author}}
</blockquote>
```

We could use it in our Markdown file like so:

```markdown
As someone said:

{%/* quote(author="Vincent") */%}
A quote
{%/* end */%}
```

The body of the shortcode will be automatically passed down to the rendering context as the `body` variable and needs
to be on a new line.

If you want to have some content that looks like a shortcode but not have Zola try to render it,
you will need to escape it by using `{%/*` and `*/%}` instead of `{%` and `%}`. You won't need to escape
anything else until the closing tag.

### Invocation Count

Every shortcode context is passed in a variable named `nth` that tracks how many times a particular shortcode has
been invoked in a Markdown file. Given a shortcode `true_statement.html` template:

```jinja2
<p id="number{{ nth }}">{{ value }} is equal to {{ nth }}.</p>
```

It could be used in our Markdown as follows:

```markdown
{{/* true_statement(value=1) */}}
{{/* true_statement(value=2) */}}
```

This is useful when implementing custom markup for features such as sidenotes or end notes.
