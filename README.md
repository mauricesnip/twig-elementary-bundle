<div>
    <p align="center">
        <img src="https://www.mauricesnip.nl/packages/twig-elementary-bundle/twig-elementary-bundle.svg" alt="Twig Elementary Bundle Logo">
    </p>
</div>

# Twig Elementary Bundle

Build Twig components in the most elemental way, without a large footprint.

## Heads up!

**This bundle is in early development stage and should not be used in production.**

The examples below require Twig's
[`HtmlExtension`](https://github.com/twigphp/html-extra) to be installed. See
the documentation for
[`html_classes`](https://twig.symfony.com/doc/3.x/functions/html_classes.html)
on how to install this extension.

## Basic usage

Start by extending from
[`element-entry.html.twig`](./src/Resources/views/core/element-entry.html.twig),
which provides switching between
[normal](./src/Resources/views/core/element.html.twig) and
[void elements](./src/Resources/views/core/element-void.html.twig)
automatically.

```twig
{# common/molecules/cards/card.html.twig #}

{% extends '@TwigElementary/core/element-entry.html.twig' %}

{# Config #}
{% set tag_name = 'article' %}
{% set base_class = 'card' %}

{# Attributes #}
{% set attributes = {
    class: html_classes(
        base_class|default,
        {
            ("#{base_class}--#{size|default}"): size|default,
            ("#{base_class}--#{theme|default}"): theme|default,
            'is-active': is_active|default,
        },
    ),
} %}
```

Using the template above, the following `include`:

```twig
{% include 'common/molecules/cards/card.html.twig' with {
    theme: 'primary',
    is_active: true,
} %}
```

Will render:

```html
<article class="card card--primary is-active"></article>
```

## Templates

### `element.html.twig`

Renders a normal element.

#### Blocks

| Name        | Description                                        |
|:------------|:---------------------------------------------------|
| `element`   | Renders the full element.                          |
| `start_tag` | Renders the start tag with all of it's attributes. |
| `end_tag`   | Renders the end tag.                               |
| `contents`  | Renders the element's contents.                    |

#### Parameters

| Name              | Type      | Default     | Description                                          |
|:------------------|:----------|:------------|:-----------------------------------------------------|
| `attributes`      | `Object`  | `{}`        | The HTML-attributes to render, eg.: `{ id: 'foo' }`. |
| `contents`        | `String`  | `undefined` | The contents of the element.                         |
| `tag_name`        | `String`  | `span`      | The HTML element's tag name, eg.: `a`, `div` or `p`. |
| `is_raw_contents` | `Boolean` | `undefined` | Whether to render raw `contents` or not.             |

### `element-void.html.twig`

Renders a void element. Extends
[`element.html.twig`](./src/Resources/views/core/element.html.twig).

#### Blocks

| Name        | Description                                        |
|:------------|:---------------------------------------------------|
| `element`   | Renders the full element.                          |
| `start_tag` | Renders the start tag with all of it's attributes. |

#### Parameters

| Name         | Type     | Default | Description                                          |
|:-------------|:---------|:--------|:-----------------------------------------------------|
| `attributes` | `Object` | `{}`    | The HTML-attributes to render, eg.: `{ id: 'foo' }`. |
| `tag_name`   | `String` | `span`  | The HTML element's tag name, eg.: `a`, `div` or `p`. |

### `element-entry.html.twig`

Renders a normal or void element automatically.

For blocks and parameters, see the documentation for normal and void element
above. Extends
[`element.html.twig`](./src/Resources/views/core/element.html.twig) or
[`element-void.html.twig`](./src/Resources/views/core/element-void.html.twig),
based on the provided `tag_name`.

## Advanced usage

If you want to extend from your newly created component, be sure to allow for
variables to receive values from their child components. Continuing on the
[basic usage](#basic-usage) example:

```twig
{# common/molecules/cards/card.html.twig #}

{% extends '@TwigElementary/core/element-entry.html.twig' %}

{# Config #}
{% set tag_name = tag_name|default('article') %}
{% set base_class = base_class|default('card') %}

{# Attributes #}
{% set attributes = {
    class: html_classes(
        element_class|default,
        base_class|default,
        {
            ("#{base_class}--#{size|default}"): size|default,
            ("#{base_class}--#{theme|default}"): theme|default,
            'is-active': is_active|default,
        },
        classes|default,
    ),
    ...attributes|default({}),
} %}
```

### Variable overriding

Note that this also enables variable overriding for `include` or `embed`, for
example:

```twig
{% include 'common/molecules/cards/card.html.twig' with {
    base_class: 'item',
    element_class: 'block__item',
    tag_name: 'div',
    theme: 'primary',
    attributes: {
        id: 'specific-item',
        'data-group': 'featured',
    },
} %}
```

Will render:

```html
<div class="block__item item item--primary" id="specific-item" data-group="featured"></div>
```

### `attributes` quirks

Be aware that overriding `attributes.class` will replace the entire `class`
attribute, since Twig does not support deep merge. Nor does merge concatenate
string values in any way. Thus, the following include:

```twig
{% include 'common/molecules/cards/card.html.twig' with {
    attributes: {
        class: 'is-active',
    },
} %}
```

Will render:

```html
<article class="is-active"></article>
```

Hence `classes|default` in the [advanced usage](#advanced-usage) example for
appending classes to the final output. In this case, the following:

```twig
{% include 'common/molecules/cards/card.html.twig' with {
    classes: html_classes(
        'is-featured',
        'w-100',
    ),
    element_class: 'block__card',
    theme: 'primary',
} %}
```

Will render:

```html
<article class="block__card card card--primary is-featured w-100"></article>
```

## Adding content

You can provide `contents` as a variable, which is mostly used for text
content. Markup is supported by setting `is_raw_contents` to `true`. See
[documentation for `element.html.twig`](#elementhtmltwig).

```twig
{# common/molecules/cards/card.html.twig #}

...

{# Contents #}
{% set contents = 'Lorem ipsum' %}
```

Or, what you probably want most of the time, override the contents `block`.

```twig
{# common/molecules/cards/card.html.twig #}

...

{# Contents #}
{% block contents %}
    {% block body %}
        <div class="{{ "#{base_class}__body" }}">
            {% block body_inner %}
                {% block title %}
                    {% if title|default %}
                        <h2 class="{{ "#{base_class}__title" }}">{{ title }}</h2>
                    {% endif %}
                {% endblock %}
            {% endblock %}
        </div>
    {% endblock %}
{% endblock %}
```

## Concrete example

Skeletons provide a base for components that come in many different flavors (or
types) to extend from. Taking the [advanced usage](#advanced-usage) example one
step further:

```twig
{# skeletons/cards/card.html.twig #}

{% extends '@TwigElementary/core/element-entry.html.twig' %}

{# Config #}
{% set tag_name = tag_name|default('article') %}
{% set base_class = base_class|default('card') %}

{# Attributes #}
{% set attributes = {
    class: html_classes(
        element_class|default,
        base_class|default,
        {
            ("#{base_class}--#{size|default}"): size|default,
            ("#{base_class}--#{theme|default}"): theme|default,
            ("#{base_class}--#{type|default}"): type|default,
            'is-active': is_active|default,
        },
        classes|default,
    ),
    ...attributes|default({}),
} %}

{# Element classes #}
{% set body_class = html_classes(
    "#{base_class}__body",
    body_classes|default,
) %}

{% set title_class = html_classes(
    "#{base_class}__title",
    title_classes|default,
) %}

{% set text_class = html_classes(
    "#{base_class}__text",
    text_classes|default,
) %}

{# Contents #}
{% block contents %}
    {% block header %}{% endblock %}
    {% block image %}{% endblock %}
    {% block body %}
        <div class="{{ body_class|trim }}">
            {% block body_inner %}
                {% block title %}
                    {% if title|default %}
                        <h2 class="{{ title_class|trim }}">{{ title }}</h2>
                    {% endif %}
                {% endblock %}
                {% block text %}
                    {% if text|default %}
                        <p class="{{ text_class|trim }}">{{ title }}</p>
                    {% endif %}
                {% endblock %}
            {% endblock %}
        </div>
    {% endblock %}
    {% block footer %}{% endblock %}
{% endblock %}
```

And a product card extending from the above skeleton:

```twig
{# common/molecules/cards/card-product.html.twig #}

{% extends 'skeletons/cards/card.html.twig' %}

{# Config #}
{% set type = 'product' %}

{# Super (use parent variables here) #}
{% block contents %}
    {% set sku_class = html_classes(
        "#{base_class}__sku",
        sku_classes|default,
    ) %}
    {{ parent() }}
{% endblock %}

{# Body inner #}
{% block body_inner %}
    {% block sku %}
        {% if sku|default %}
            <p class="{{ sku_class|trim }}">{{ sku }}</p>
        {% endif %}
    {% endblock %}
    {{ parent() }}
{% endblock %}
```

This inherits all the way up to
[`element.html.twig`](./src/Resources/views/core/element.html.twig), adding
features along the way. With that in mind, the following `include`:

```twig
{% set some_size_variable = 'lg' %}

...

{% include 'common/molecules/cards/card-product.html.twig' with {
    title: 'Fancy product',
    text: 'Lorem ipsum dolor sit amet consectetur adipiscing elit.',
    sku: 12345678,
    sku_classes: html_classes(
        {
            'font-size-small': some_size_variable|default == 'sm',
            'font-size-large': some_size_variable|default == 'lg',
        },
    ),
} %}
```

Will render:

```html
<article class="card card--product">
    <div class="card__body">
        <p class="card__sku font-size-large">12345678</p>
        <h2 class="card__title">Fancy product</h2>
        <p class="card__text">Lorem ipsum dolor sit amet consectetur adipiscing elit.</p>
    </div>
</article>
```

### Visualized

```
   ┌───────────────────┐
   │                   │
   │ element.html.twig │        Handles tag name, start and end tag, attributes and contents
   │                   │
   └─────────▲─────────┘
             │
┌────────────┴────────────┐
│                         │
│ element-entry.html.twig │     Handles element type (normal vs. void)
│                         │
└────────────▲────────────┘
             │
     ┌───────┴────────┐
     │                │
     │ card.html.twig │         Handles classes, structure, title and text
     │                │
     └───────▲────────┘
             │
 ┌───────────┴────────────┐
 │                        │
 │ card-product.html.twig │     Handles SKU and sku_classes
 │                        │
 └────────────────────────┘
```
