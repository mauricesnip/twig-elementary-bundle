# Twig Elementary Bundle

Build Twig components in the most elemental way, without a large footprint.

## Heads up!

The examples below require Twig's
[`HtmlExtension`](https://github.com/twigphp/html-extra) to be installed. See
the documentation for
[`html_classes`](https://twig.symfony.com/doc/3.x/functions/html_classes.html)
on how to install this extension.

## Basic usage

```twig
{# common/molecules/cards/card.html.twig #}

{% extends 'core/element-entry.html.twig' %}

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

## Advanced usage

If you want to extend from your newly created component, be sure to allow for
variables to receive values from their child components, like so:

```twig
{# common/molecules/cards/card.html.twig #}

{% extends 'core/element-entry.html.twig' %}

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
}|merge(attributes|default({})) %}
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

Be aware that overriding `attributes.class` will yield unexpected `class`
attributes, since the Twig `merge` function does not perform a deep merge.

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

Hence `classes|default` in `common/molecules/cards/card.html.twig`, for
appending classes to the final output:

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

You can provide `contents` as a variable.

```twig
{# common/molecules/cards/card.html.twig #}

...

{# Contents #}
{% set contents = 'I'm a card' %}
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
types) to extend from. Taking our advanced usage example one step further:

```twig
{# skeletons/cards/card.html.twig #}

{% extends 'elementary/core/element-entry.html.twig' %}

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
}|merge(attributes|default({})) %}

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

{# Element classes #}
{% set sku_class = html_classes(
    "#{base_class}__sku",
    sku_classes|default,
) %}

{# Contents #}
{% block body_inner %}
    {% block sku %}
        {% if sku|default %}
            <p class="{{ sku_class|trim }}">{{ sku }}</p>
        {% endif %}
    {% endblock %}
    {{ parent() }}
{% endblock %}
```

This inherits all the way up to `element.html.twig`, adding features along the
way. With that in mind, the following `include`:

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