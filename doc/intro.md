Introduction
============

lh-style is a vim script library that defines vim functions and commands that permit to specify stylistic preferences
like naming conventions, bracket formatting, etc.

In itself the only feature end-users can directly exploit is name converting based on the style name (`snake_case`,
`UpperCamelCase`...) like Abolish plugin does, or on a given identifier kind (_function_, _type_, _class_,
_attribute_...).  Check [`:NameConvert`](naming.md#nameconvert) and [`:ConvertNames`](naming.md#convertnames) --
sorry I wasn't inspired.

The main, and unique, feature this plugin offers is core code-style functionalities that other plugins can exploit.
Typical client plugins would be code generating plugins: wizards/snippet/abbreviation plugins, and refactoring plugins.

The style can be tuned through options. The options are meant to be tuned by end-users, and indirectly used by plugin
maintainers.  See [API section](doc/API.md) to see how you could exploit these options from your plugins.

Snippets from [lh-cpp](http://github.com/LucHermitte/lh-cpp) and
[mu-template](http://github.com/LucHermitte/mu-template), and refactorings from
[lh-refactor](http://github.com/LucHermitte/lh-refactor) exploit the options offered by lh-style for specifying code
style.


> Note: The library has been extracted from [lh-dev](http://github.com/LucHermitte/lh-dev) v2.x.x. in order to remove dependencies to [lh-tags](http://github.com/LucHermitte/lh-tags) and other plugins from template/snippet expander plugins like [mu-template](http://github.com/LucHermitte/mu-template). Yet, I've decided to reset the version counter to 1.0.0.

