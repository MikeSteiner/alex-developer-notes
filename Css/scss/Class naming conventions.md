#Css #Scss #CssNaming

## BEM standard
#CssNamingBem

```scss
$item__color: white;
   
$item--selected__color: grey;

.uic-some-component__item {
    color: $item__color

    &.uic-selected {
        color: $item--selected__color;
    }
}
```

## Why BEM over the others?

No matter what methodology you choose to use in your projects, you will benefit from the advantages of more structured CSS and UI. Some styles are less strict and more flexible, while others are easier to understand and adapt in a team.

> The reason I choose BEM over other methodologies comes down to this: it is less confusing than the other methods (i.e. SMACSS) but still provides us the good architecture we want (i.e. OOCSS) and with a recognizable terminology. - Mark McDonnell, [Maintainable CSS with BEM](https://www.zcfy.cc/original/maintainable-css-with-bem)

## Blocks, Elements and Modifiers

### Block

Standalone entity that is meaningful on its own.

#### Examples

`header`, `container`, `menu`, `checkbox`, `input`

### Element

A part of a block that has no standalone meaning and is semantically tied to its block.

#### Examples

`menu item`, `list item`, `checkbox caption`, `header title`

### Modifier

A flag on a block or element. Use them to change appearance or behavior.

#### Examples

`disabled`, `highlighted`, `checked`, `fixed`, `size big`, `color yellow`

![GitHub with captions](https://getbem.com/assets/github_captions.3a78c10d_ZfCSDb.jpg)

## Under the hood

Let’s look how one particular element on a page can be implemented in BEM. We will take `button` from [GitHub](https://primer.style/css/components/buttons):

![GitHub buttons](https://getbem.com/assets/github_buttons.838c4512_Z2sSSxt.jpg)

We can have a normal button for usual cases, and two more states for different ones. Because we style blocks by class selectors with BEM, we can implement them using any tags we want (`button`, `a` or even `div`). The naming rules tell us to use `block--modifier-value` syntax.

#### HTML

```
<button class="button">
	Normal button
</button>
<button class="button button--state-success">
	Success button
</button>
<button class="button button--state-danger">
	Danger button
</button>
```

#### CSS

```
.button {
	display: inline-block;
	border-radius: 3px;
	padding: 7px 12px;
	border: 1px solid #D5D5D5;
	background-image: linear-gradient(#EEE, #DDD);
	font: 700 13px/18px Helvetica, arial;
}
.button--state-success {
	color: #FFF;
	background: #569E3D linear-gradient(#79D858, #569E3D) repeat-x;
	border-color: #4A993E;
}
.button--state-danger {
	color: #900;
}
```

## Benefits

### Modularity

Block styles are never dependent on other elements on a page, so you will never experience [problems from cascading](http://www.phase2technology.com/blog/used-and-abused-css-inheritance-and-our-misuse-of-the-cascade/).

You also get the ability to transfer blocks from your finished projects to new ones.

### Reusability

Composing independent blocks in different ways, and reusing them intelligently, reduces the amount of CSS code that you will have to maintain.

With a set of style guidelines in place, you can build a library of blocks, making your CSS super effective.

### Structure

BEM methodology gives your CSS code a solid structure that remains simple and easy to understand.

---
https://getbem.com/introduction/