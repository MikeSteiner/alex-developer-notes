https://sam.hooke.me/note/2018/03/hugo-tag-and-category-pages/

#Hugo #Taxonomy 
# Hugo tag and category pages

Taxonomies are ways of grouping content within hugo. [By default](https://gohugo.io/content-management/taxonomies/#hugo-taxonomy-defaults), hugo defines two taxonomies: “tags” and “categories”. You can then, for example, write a post and define `tags = ["test"]` in the front matter. Hugo will then generate a taxonomy list page at `https://mywebsite.com/tags/` and you can view all posts with the `"test"` tag at `https://mywebsite.com/tags/test`.

Note that in all this “tags” is plural. For this website, I wanted the taxonomy list page to be `https://mywebsite.com/tag` _singular_. It turns out this is rather easy!

To use singular rather than plural, override the default values for the tags and categories taxonomies:

##### `config.toml` [](https://sam.hooke.me/note/2018/03/hugo-tag-and-category-pages/#configtoml)

```toml
[taxonomies]
# Use singular rather than plural for default taxonomies
tag = "tag"
category = "category"
```

As shown above, the `[taxonomies]` section of the config file defines a list of key/value pairs, where the key is the singular term, and the value is the plural term. Hugo uses the plural term of the taxonomy for the list page URL, so by making this singlar we get the URL we desire.

You can then go to `https://mywebsite.com/tag/` to view the list of tags.

## Templates [](https://sam.hooke.me/note/2018/03/hugo-tag-and-category-pages/#templates)

In order to customise things further, you can modify the templates used:

-   `layouts/_default/terms.html` is the template for `https://mywebsite.com/tag/`, which lists all the terms for that tag.
-   `layouts/_default/taxonomy.html` is the template for `https://mywebsite.com/tag/term/`, which lists all the pages for that term.

The hugo documentation has a good starting point for [writing your own taxonomy templates](https://gohugo.io/templates/taxonomy-templates/#order-taxonomies).

## Disabling taxonomies [](https://sam.hooke.me/note/2018/03/hugo-tag-and-category-pages/#disabling-taxonomies)

To [disable the default taxonomies entirely](https://discourse.gohugo.io/t/turning-off-taxonomy-pages/2622/3), simply set their values to empty strings.

##### `config.toml` [](https://sam.hooke.me/note/2018/03/hugo-tag-and-category-pages/#configtoml-1)

```toml
[taxonomies]
# Disable the default taxonomies
tag = ""
category = ""
```


