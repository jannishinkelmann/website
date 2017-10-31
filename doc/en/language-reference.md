# Language reference

## Attributes

Tag attributes look similar to HTML (with optional commas),
but their values are just expressions.

```phug
a(href='google.com') Google
="\n"
a(class='button' href='google.com') Google
="\n"
a(class='button', href='google.com') Google
```

(`="\n"` are just here to add whitespace between links for an
easier reading of the output HTML).

Normal PHP expressions work fine by default on **Phug** and
**Tale-pug**; and on **Pug-php** with `expressionLanguage`
option set to `php`:
```phug
- $authenticated = true
body(class=$authenticated ? 'authed' : 'anon')
```

Normal JavaScript expressions work fine too with **js-phpize-phug**
extension installed ([see how to install](#use-javascript-expressions))
or **Pug-php** last version with default options (or `expressionLanguage`
set to `js`):
```pug
- var authenticated = true
body(class=authenticated ? 'authed' : 'anon')
```

### Multiline Attributes

If you have many attributes, you can also spread them across many
lines:
```phug
input(
  type='checkbox'
  name='agreement'
  checked
)
```

Multiline texts are not at all a problem either you use JS or PHP
expressions:
```pug
input(data-json='
  {
    "very-long": "piece of ",
    "data": true
  }
')
```

Note: if you migrate templates from a JavaScript project you might
have to replace <code>&#96;</code> by simple quotes `'` or double
quotes `"`.

### Quoted Attributes

If your attribute name contains odd characters that might interfere
with expressions syntax, either quote it using `""` or `''`, or use
commas to separate different attributes. Examples of such characters
include `[]` and `()` (frequently used in Angular 2).

```phug
//- In this case, `[click]` is treated
//- as an offset getter, resulting
//- in the unusual error.
div(class='div-class' [click]='play()')
```

```phug
div(class='div-class', [click]='play()')
div(class='div-class' '[click]'='play()')
```

### Attribute Interpolation

A small note about a syntax you may hav known in **pugjs 1**:
`a(href="/#{url}") Link`, this syntax is no longer valid in **pugjs 2**
and so we decided to not support it in **Phug**. You can PHP
interpolation:
```phug
- $btnType = 'info'
- $btnSize = 'lg'
button(type="button" class="btn btn-$btnType btn-$btnSize")
- $btn = (object) ['size' => 'lg']
button(type="button" class="btn btn-{$btn->size}")
```
For JS expressions, use concatenation `" + btnType + "`.

### Unescaped Attributes

By default, all attributes are escaped to prevent attacks (such as
cross site scripting). If you need to use special characters,
use `!=` instead of `=`.
```phug
div(escaped="&lt;code>")
div(unescaped!="&lt;code>")
```

**Caution, unescaped buffered code can be dangerous.** You must be
sure to sanitize any user inputs to avoid
[cross-site scripting](https://en.wikipedia.org/wiki/Cross-site_scripting).

### Unchecked attributes

Here is a specific concept of **Phug** you would not find in **pugjs**,
it's about checking a variable exist.

In PHP, when error are displayed and error level enable notices, calling
a non-existing variable will throw an error.

By default, we will hide those error, but this could hide some bug, so
you can use `?=` operator to avoid this behavior:

```phug
- $wrong = ''
img(src?=$wronk)
```
In this example, the `?=` operator will reveal the miss typo of "wrong".
Click on the `[Preview]` button to see the error.

Attributes can be both unchecked and unescaped:

```phug
- $html = '&lt;strong>OK&lt;/strong>'
img(alt?!=$html)
```

To disable globally the checking (always throw an error if the variable
called is not defined), use the `php_token_handlers` option:

```php
Phug::setOption(['php_token_handlers', T_VARIABLE], null);
```

### Boolean Attributes

Boolean attributes are mirrored by **Phug**. Boolean values (`true` and
`false`) are accepted. When no value is specified `true` is assumed.
```phug
input(type='checkbox' checked)
="\n"
input(type='checkbox' checked=true)
="\n"
input(type='checkbox' checked=false)
="\n"
input(type='checkbox' checked='true')
```

If the doctype is `html`, **Phug** knows not to mirror the attribute,
and instead uses the terse style (understood by all browsers).
```phug
doctype html
="\n"
input(type='checkbox' checked)
="\n"
input(type='checkbox' checked=true)
="\n"
input(type='checkbox' checked=false)
="\n"
input(type='checkbox' checked='checked')
```

### Style Attributes

The `style` attribute can be a string, like any normal attribute; but
it can also be an object or an array.

PHP-style:
```phug
a(style=['color' => 'red', 'background' => 'green'])
="\n"
a(style=(object)['color' => 'red', 'background' => 'green'])
```

[JS-style](#use-javascript-expressions):
```pug
a(style={color: 'red', background: 'green'})
```

### Class Attributes

The `class` attribute can be a string, like any normal attribute;
but it can also be an array of class names, which is handy when
generated.
```phug
- $classes = ['foo', 'bar', 'baz']
a(class=$classes)
="\n"
//- the class attribute may also be repeated to merge arrays
a.bang(class=$classes class=['bing'])
```

It can also be an array which maps class names to `true` or
`false` values. This is useful for applying conditional classes.

PHP-style:
```pug
- $currentUrl = '/about'
a(ref='/' class=['active' => $currentUrl === '/']) Home
="\n"
a(href='/about' class=['active' => $currentUrl === '/about']) About
```

[JS-style](#use-javascript-expressions):
```pug
- var currentUrl = '/about'
a(href='/' class={active: currentUrl === '/'}) Home
="\n"
a(href='/about' class={active: currentUrl === '/about'}) About
```

### Class Literal

Classes may be defined using a `.classname` syntax:
```phug
a.button
```

Since `div`'s are such a common choice of tag, it is the
default if you omit the tag name:
```phug
.content
```

### ID Literal

IDs may be defined using a `#idname` syntax:
```phug
a#main-link
```

Since `div`'s are such a common choice of tag,
it is the default if you omit the tag name:
```phug
#content
```

### &attributes

Pronounced as “and attributes”, the `&attributes` syntax can
be used to explode an array into attributes of an element.

PHP-style:
```phug
div#foo(data-bar="foo")&attributes(['data-foo' => 'bar'])
```

[JS-style](#use-javascript-expressions):
```pug
div#foo(data-bar="foo")&attributes({'data-foo': 'bar'})
```

The above examples uses an array literal. But you can also use
a variable whose value is an array, too. (See also:
[Mixin Attributes](#mixin-attributes)).
```phug
- $attributes = []
- $attributes['class'] = 'baz'
div#foo(data-bar="foo")&attributes($attributes)
```

**Caution, attributes applied using `&attributes` are not
automatically escaped.** You must be sure to sanitize any user
inputs to avoid
[cross-site scripting](https://en.wikipedia.org/wiki/Cross-site_scripting)
(XSS). If passing in `attributes` from a mixin call, this is
done automatically.

## Case

The `case` statement is a shorthand for the PHP `switch` statement.
It takes the following form:

```phug
- $friends = 10
case $friends
  when 0
    p you have no friends
  when 1
    p you have a friend
  default
    p you have #{$friends} friends
```

### Case Fall Through

You can use fall through, just as you would in a PHP `switch` statement.

```phug
- $friends = 0
case $friends
  when 0
  when 1
    p you have very few friends
  default
    p you have #{$friends} friends
```

The difference, however, is a fall through in PHP happens whenever a
`break` statement is not explicitly included; in **Phug**, it only happens
when a block is completely missing.

If you would like to not output anything in a specific case, add an explicit
unbuffered `break`:

```phug
- $friends = 0
case $friends
  when 0
    - break
  when 1
    p you have very few friends
  default
    p you have #{$friends} friends
```

### Block Expansion

Block expansion may also be used:

```phug
- $friends = 1
case $friends
  when 0: p you have no friends
  when 1: p you have a friend
  default: p you have #{$friends} friends
```

## Code

**Phug** allows you to write inline PHP or JavaScript code in
your templates. The code can be buffered or not. When it is
buffered, it can be escaped or not, checked or not the same
way as attributes.

### Unbuffered Code

Unbuffered code starts with `-`. It does not directly add
anything to the output.

```phug
- for ($x = 0; $x < 3; $x++)
  li item
```

**Phug** also supports block unbuffered code:

```phug
-
  $list = ["Uno", "Dos", "Tres",
          "Cuatro", "Cinco", "Seis"]
each $item in $list
  li= $item
```

### Buffered Code

Buffered code starts with `=`. It evaluates the PHP or JavaScript
expression and outputs the result.
For security, buffered code is first HTML escaped.

```phug
p
  = 'This code is &lt;escaped>!'
```

It can also be written inline, and supports the full range of
expressions:

```phug
p= 'This code is' . ' &lt;escaped>!'
```

Note: if you use JavaScript expressions, concatenations must use
the `+` operator:

```pug
p= 'This code is' + ' &lt;escaped>!'
```

### Unescape/escape code

Precede the `=` with `!` to not escape HTML entities, with `?`
to not check variables to be set and `?!` for both:

```phug
- $start = '&lt;strong>'
- $end = '&lt;/strong>'
//- This one is escaped
div= $start . 'Word' . $end
//- This one is not escaped
div!= $start . 'Word' . $end
//- Both are checked
div= 'start' . $middle . 'end'
div!= 'start' . $middle . 'end'
```

### Uncheck/check code

Checked code does not throw an error when variables are
undefined.

Unchecked code throws an error when variables are
undefined. See the examples below with on the one hand
an existing variable, and on the other hand a missing
variable:
```phug
- $middle = ' middle '
div?= 'start' . $middle . 'end'
div?!= 'start' . $middle . 'end'
```

```phug
div?= 'start' . $middle . 'end'
div?!= 'start' . $middle . 'end'
```

**Caution: unescaped buffered code can be dangerous.** You must
be sure to sanitize any user inputs to avoid
[cross-site scripting](https://en.wikipedia.org/wiki/Cross-site_scripting)
(XSS).

## Commentaires

Buffered comments look the same as single-line JavaScript comments.
They act sort of like markup tags, producing *HTML* comments in the
rendered page.

Like tags, buffered comments must appear on their own line.

```phug
// just some paragraphs
p foo
p bar
// each line
// will produce
// a HTML comment
footer
```

**Phug** also supports unbuffered comments (they will be not compiled
so you can add many with no impact on the cache file size).
Simply add a hyphen (`-`) to the start of the comment.

```phug
//- will not output within markup
p foo
p bar
```

### Block Comments

Block comments work, too:
```phug
body
  //-
    Comments for your template writers.
    Use as much text as you want.
  //
    Comments for your HTML readers.
    Use as much text as you want.
```

### Conditional Comments

**Phug** does not have any special syntax for conditional comments.
(Conditional comments are a peculiar method of adding fallback
markup for old versions of Internet Explorer.)

However, since all lines beginning with `<` are treated as [plain
text](#plain-text), normal HTML-style conditional comments work
just fine.
```phug
doctype html

&lt;!--[if IE 8]>
&lt;html lang="fr" class="lt-ie9">
&lt;![endif]-->
&lt;!--[if gt IE 8]>&lt;!-->
&lt;html lang="fr">
&lt;!--&lt;![endif]-->

body
  p Supporting old web browsers is a pain.

&lt;/html>
```

## Conditionals

Conditionals look like the following if you use PHP-style:
```phug
- $user = [ 'description' => 'foo bar baz' ]
- $authorised = false
#user
  if $user['description']
    h2.green Description
    p.description= $user['description']
  else if $authorised
    h2.blue Description
    p.description.
      User has no description,
      why not add one...
  else
    h2.red Description
    p.description User has no description
```

If you [use JS-style](#use-javascript-expressions):
```pug
- var user = { description: 'foo bar baz' }
- var authorised = false
#user
  if user.description
    h2.green Description
    p.description= user.description
  else if authorised
    h2.blue Description
    p.description.
      User has no description,
      why not add one...
  else
    h2.red Description
    p.description User has no description
```

**Phug** also provides the conditional `unless`, which works
like a negated `if`.

PHP-style example:
```phug
unless $user['isAnonymous']
  p You're logged in as #{$user['name']}.
//- is equivalent to
if !$user['isAnonymous']
  p You're logged in as #{$user['name']}.
```
```vars
[
  'user' => [
    'isAnonymous' => false,
    'name' => 'Jefferson Airplane',
  ],
]
```

[JS-style](#use-javascript-expressions) example:
```pug
unless user.isAnonymous
  p You're logged in as #{user.name}.
//- is equivalent to
if !user.isAnonymous
  p You're logged in as #{user.name}.
```
```vars
[
  'user' => [
    'isAnonymous' => false,
    'name' => 'Jefferson Airplane',
  ],
]
```

## Doctype

```phug
doctype html
```

### Doctype Shortcuts

Shortcuts to commonly used doctypes:

```phug
doctype html
doctype xml
doctype transitional
doctype strict
doctype frameset
doctype 1.1
doctype basic
doctype mobile
doctype plist
```

### Custom Doctypes

You can also use your own literal custom doctype:

```phug
doctype html PUBLIC "-//W3C//DTD XHTML Basic 1.1//EN"
```

### Doctype Option

In addition to being buffered in the output, a doctype in **Phug**
can affect compilation in other ways. For example, whether
self-closing tags end with `/>` or `>` depends on whether HTML
or XML is specified. The output of
[boolean attributes](#booleens-attributes)
may be affected as well.

If, for whatever reason, it is not possible to use the `doctype`
keyword (e.g., just rendering HTML fragments), but you would
still like to specify the doctype of the template, you can do
so via the [doctype option](#options-doctype).

```php
$source = 'img(src="foo.png")';

Phug::render($source);
// => '<img src="foo.png"/>'

Phug::render($source, [], [
  'doctype' => 'xml',
]);
// => '<img src="foo.png"></img>'

Phug::render($source, [], [
  'doctype' => 'html',
]);
// => '<img src="foo.png">'
```

## Filters

Filters let you use other languages in Pug templates. They take a
block of plain text as an input.

To pass options to the filter, add them inside parentheses after
the filter name (just as you would do with
[tag attributes](#attributes)): `:less(compress=false)`.

All [JSTransformer modules](https://www.npmjs.com/browse/keyword/jstransformer)
can be used as Pug filters. Popular
filters include :babel, :uglify-js, :scss, and :markdown-it.
Check out the documentation for the JSTransformer for the options
supported for the specific filter.

To use JSTransformer with **Phug**, install the following
extension:

```shell
composer require phug/js-transformer-filter
```

Then enable it:

```php
use Phug\JsTransformerExtension;
use Phug\Phug;

Phug::addExtension(JsTransformerExtension::class);
```

With **Pug-php**, this extension is installed and enabled by
default since 3.0.1 version.

You can also use any of the following PHP projects as filter
in all **Phug** and **Pug-php** projects:
http://pug-filters.selfbuild.fr

If you can't find an appropriate filter for your use case, you
can write your own custom filter. It can be any callable (closure,
function or object with __invoke method).

```php
Phug::setFilter('upper-start', function ($inputString, $options) {
  return strtoupper(mb_substr($inputString, 0, $options['length'])).
    mb_substr($inputString, $options['length']);
});

Phug::display('
div
  :upper-start(length=3)
    gggggg
');
```

This will output:
```html
<div>GGGggg</div>
```

The same goes for **Pug-php**:

```php
$pug = new Pug();
$pug->setFilter('upper-start', function ($inputString, $options) {
  return strtoupper(mb_substr($inputString, 0, $options['length'])).
    mb_substr($inputString, $options['length']);
});

$pug->display('
div
  :upper-start(length=3)
    gggggg
');
```

**Phug** comes with `cdata` filter pre-installed:
```phug
data
  :cdata
    <![CDATA[
      Since this is a CDATA section
      I can use all sorts of reserved characters
      like > < " and &
      or write things like
      <foo></bar>
      but my document is still well formed!
    ]]>
```

**Pug-php** pre-install `JsTransformerExtension` and embed `cdata`,
`css`, `escaped`, `javascript`, `php`, `pre`, `script`
and if you not set the `filterAutoLoad` to `false`, it will load
automatically as filter any invokable class in the `Pug\Filter`
namespace:

```pug
doctype html
html
  head
    :css
      a {
        color: red;
      }
  body
    :php
      $calcul = 9 + 8
    p=calcul
    :escaped
      <foo>
    :pre
      div
        h1 Example of Pug code
    :javascript
      console.log('Hello')
    :script
      console.log('Alias of javascript')
```