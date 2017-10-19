# lh-style [![Build Status](https://secure.travis-ci.org/LucHermitte/lh-style.png?branch=master)](http://travis-ci.org/LucHermitte/lh-style) [![Project Stats](https://www.openhub.net/p/21020/widgets/project_thin_badge.gif)](https://www.openhub.net/p/21020)


Discl. This page is currently under (re)construction!

TODO:

- Complete the documentation
- reorganize the sections to:
    -[X] Intro
    -[ ] Features
        -[ ] Code formatting
            -[x] rationale
            -[ ] end-user side
                -[x] `.editorconfig`
                -[x] `.clang-format` (to be implemented)
                -[ ] `:UseStyle`
                -[ ] `:AddStyle`
            -[ ] extending the style families
        -[ ] Naming policies
    -[ ] API
    -[ ] Contributing
    -[X] Installation

## Introduction

lh-style is a vim script library that defines vim functions and commands that permit to specify stylistic preferences
like naming conventions, bracket formatting, etc.

In itself the only feature end-users can directly exploit is name converting based on the style name (`snake_case`,
`UpperCamelCase`...) like Abolish plugin does, or on a given identifier kind (function, type, class, attribute...).
Check `:NameConvert` and `ConvertNames` -- sorry I wasn't inspired.

The main, and unique, feature is this plugin is to offer core functionalities, related to code-style, other plugins can
exploit. Typical client plugins would be code generating plugins: wizards/snippet/abbreviation plugins, and refactoring
plugins.

The style can be tuned through options. The options are meant to be tuned by end-users, and indirectly used by plugin
maintainers.  See [API section](#API) to see how you could exploit these options from your plugins.

Snippets from [lh-cpp](http://github.com/LucHermitte/lh-cpp) and
[mu-template](http://github.com/LucHermitte/mu-template), and refactorings from
[lh-refactor](http://github.com/LucHermitte/lh-refactor) exploit the options offered by lh-style for specifying code
style.


> Note: The library has been extracted from [lh-dev](http://github.com/LucHermitte/lh-dev) v2.x.x. in order to remove dependencies to [lh-tags](http://github.com/LucHermitte/lh-tags) and other plugins from template/snippet expander plugins like [mu-template](http://github.com/LucHermitte/mu-template). Yet, I've decided to reset the version counter to 1.0.0.

---

## Features


### Naming policy
Thanks to lh-style, we can define how functions, classes, constants, attributes, etc. shall be named (`UpperCamelCase`,
`lowerCamelCase`, `snake_case`, or `Any_thingElse`).

This information can then be retrieved by plugins through the [Naming API](#naming-api).

This information can also be used from `:NameConvert` and `:ConvertNames` commands.
  * `:NameConvert` converts the identifier under the cursor to one of the following naming policies (the command supports autocompletion):
    * naming styles: upper\_camel\_case, lower\_camel\_case, underscore/snake, variable,
    * identifier kinds: getter, setter, local, global, member, static, constant, param (the exact conversion process can
      be tuned thanks to the [following options](#naming-conventions)).
  * `:[range]ConvertNames/pattern/policy/[flags]` transforms according to the _policy_ all names that match the _pattern_
    -- it applies `:NameConvert` on text matched by `:substitute`.

#### Naming policy options

Naming conventions can be defined to:
  * Control prefix and suffix on:
      * variables (main name)
      * global and local variables
      * member and static variables
      * (formal) parameters
      * constants
      * getters and setters
      * types
  * Control the case policy (`snake_case`, `UpperCamelCase`, `lowerCamelCase`) on functions (and thus on setters and
    getters too) and types.

It is done, respectively, with the following options:
  * regarding prefix and suffix:
      * `(bpg):[{ft}_]naming_strip_re` and `(bpg):[{ft}_]naming_strip_subst`,
      * `(bpg):[{ft}_]naming_global_re`, `(bpg):[{ft}_]naming_global_subst`, `(bpg):[{ft}_]naming_local_re`, and `(bpg):[{ft}_]naming_local_subst`,
      * `(bpg):[{ft}_]naming_member_re`, `(bpg):[{ft}_]naming_member_subst`, `(bpg):[{ft}_]naming_static_re`, and `(bpg):[{ft}_]naming_static_subst`,
      * `(bpg):[{ft}_]naming_param_re`, and `(bpg):[{ft}_]naming_param_subst`,
      * `(bpg):[{ft}_]naming_constant_re`, and `(bpg):[{ft}_]naming_constant_subst`,
      * `(bpg):[{ft}_]naming_get_re`, `(bpg):[{ft}_]naming_get_subst`, `(bpg):[{ft}_]naming_set_re`, and `(bpg):[{ft}_]naming_set_subst`
      * `(bpg):[{ft}_]naming_type_re`, and `(bpg):[{ft}_]naming_type_subst`,
  * regarding case:
      * `(bpg):[{ft}_]naming_function`
      * `(bpg):[{ft}_]naming_type`

Once in the _main name_ form, the `..._re` regex options match the _main name_ while the `..._subst` replacement text is applied instead.

You can find examples for these options in mu-template
[template](http://github.com/LucHermitte/mu-template/blob/master/after/template/vim/internals/vim-rc-local-cpp-style.template)
used by [BuildToolsWrapper](http://github.com/LucHermitte/BuildToolsWrapper)'s `:BTW new_project` command.

---

### Code formatting

Some projects will want to have open curly-brackets on new lines (see
[Allman indenting style](https://en.wikipedia.org/wiki/Indentation_style#Allman_style)), other will prefer to have the
open bracket on the same line as the function/control-statement/... (see
[K&R indenting style](https://en.wikipedia.org/wiki/Indentation_style#K.26R),
[Java coding style](https://en.wikipedia.org/wiki/Indentation_style#Variant:_Java)...). We can also choose whether
one space shall be inserted before opening braces, and so on.

Of course we could apply reformatting tools (like clang-format) on demand, but then we'd need to identify a set of
different tools dedicated to different languages. The day code formatting is handled by
[Language Server Protocol](http://langserver.org/), we would have access to a simple and extensible solution. In the
mean time, here is lh-style.

lh-style doesn't do any replacement by itself on snippets or abbreviation. It is expected to be used by snippet plugins.
So far, only [mu-template](http://github.com/LucHermitte/mu-template) and [lh-cpp](http://github.com/LucHermitte/lh-cpp)
exploit this feature.


Different people will need to do different things:
- Plugin maintainers will use the [dedicated API](#formatting-api) to reformat on-the-fly the code they generate.
- End-users will specify the coding style used on their project(s):
  - either by specifying a set of independent styles on different topics (families) (`:UseStyle`, `.editorconfig`,
    `.clang-format`),
  - or by being extremely precise (`:AddStyle`).


#### Style families

##### Families already implemented

At this time, the following style families are implemented:

* [EditorConfig styles](https://github.com/editorconfig/editorconfig/wiki/EditorConfig-Properties#ideas-for-domain-specific-properties):
   - `curly_bracket_next_line`  = [`yes`, `no`] // `true,` `false,` `0,` `1`.
   - `indent_brace_style`       = [`0tbs`, `1tbs`, `allman`, `bsd_knf`, `gnu`, `horstmann`, `java`, `K&R`,
                                 `linux_kernel`, `lisp`, `none`, `pico`, `ratliff`, `stroustrup`, `whitesmiths]`
   - `spaces_around_brackets`   = [`inside`, `outside`, `both`, `none`]

* Clang-Format styles:
   - `breakbeforebraces`        = [`allman`, `attach`, `gnu`, `linux`, `none`, `stroustrup` ]
   - `spacesbeforeparens`       = [`none`, `never`, `always`, `control-statements`]
   -` spacesinemptyparentheses` = [`yes`, `no`] // `true,` `false,` `0,` `1`.
   -` spacesinparentheses`      = [`yes`, `no`] // `true,` `false,` `0,` `1`.

If you want more precise control, without family management, you can use `:AddStyle` instead.


##### `:UseStyle`
`:UseStyle {style-family}={value} [-buffer] [-ft[={ft}]] [-prio={prio}]`

Given _style families_, `:UseStyle` can be used to specify which particular style shall be used when generating code.

For instance,

```vim
:UseStyle breakbeforebraces=Allman -ft=c
:UseStyle spacesbeforeparens=control-statements -ft=c
:UseStyle spacesinparentheses=no
```

will tune snippets/abbreviations to follow [Allman indenting style](https://en.wikipedia.org/wiki/Indentation_style#Allman_style), and to add a space before control-statement parentheses, and to never insert a space inside parentheses.

Note that some families are incompatible with other families. In happens quickly when we mix overlapping families from
different origins.

###### Command Options
*  `{style-option}` specifies given a style family what choice has been made.

    Styles can easilly be added in `{&rtp}/autoload/lh/style/`. If you don't
    remember them all, don't worry, `:UseStyle` supports command-line
    completion.

    Setting the style to `none` unsets the whole family for the related
    [`buffers`](http://vimhelp.appspot.com/windows.txt.html#buffers)/[`filetype`](http://vimhelp.appspot.com/filetype.txt.html#filetype)
    Giving a new {value}, overrides the style for the related
    [`buffers`](http://vimhelp.appspot.com/windows.txt.html#buffers)/[`filetype`](http://vimhelp.appspot.com/filetype.txt.html#filetype).

* "`-buffer`" defines this association only for the current buffer. This option is meant to be used with plugins like [local\_vimrc](https://github.com/LucHermitte/local_vimrc).
* "`-ft[={ft}]`" defines this association only for the specified filetype. When `{ft}` is not specified, it applies only to the current filetype. This option is meant to be used in .vimrc, in the global zone of |filetype-plugin|s or possibly in [local\_vimrcs](https://github.com/LucHermitte/local_vimrc) (when combined with "`-buffer`").
* "`-prio={prio}`" Sets a priority that'll be used to determine which key is matching the text to enhance. By default all style have a priority of 1. The typical application is to have template expander ignore single curly brackets.

##### `.editorconfig`
lh-style registers a hook to editorconfig-vim in order to extract style choices expressed in any `.editorconfig` file
applicable.

The syntax would be:

```dosini
[*]
indent_brace_style: Allman
```

In every buffer where editor config applies its settings, it will be translated into:

```vim
:UseStyle -b indent_brace_style=allman
```


##### `.clang-format`
The idea is the same: to detect automatically a `.clang-format` in project root directory and apply the styles supported
by lh-style.

At this time, this feature isn't implemented yet.

##### `:AddStyle`

`:AddStyle {key} [-buffer] [-ft[={ft}]] [-prio={prio}] {Replacement}`
  * `{key}` is a regex that will get replaced automatically (by plugins supporting this API)
  * `{replacement}` is what will be inserted in place of `{key}`
  * "`-buffer`" defines this association only for the current buffer. This option is meant to be used with plugins like [local\_vimrc](https://github.com/LucHermitte/local_vimrc).
  * "`-ft[={ft}]`" defines this association only for the specified filetype. When `{ft}` is not specified, it applies only to the current filetype. This option is meant to be used in .vimrc, in the global zone of |filetype-plugin|s or possibly in [local\_vimrcs](https://github.com/LucHermitte/local_vimrc) (when combined with "`-buffer`").
  * "`-prio={prio}`" Sets a priority that'll be used to determine which key is matching the text to enhance. By default all style have a priority of 1. The typical application is to have template expander ignore single curly brackets.

######Examples:

```vim
" # Space before open bracket in C & al {{{2
" A little space before all C constructs in C and child languages
" NB: the spaces isn't put before all open brackets
AddStyle if(     -ft=c   if\ (
AddStyle while(  -ft=c   while\ (
AddStyle for(    -ft=c   for\ (
AddStyle switch( -ft=c   switch\ (
AddStyle catch(  -ft=cpp catch\ (

" # Ignore style in comments after curly brackets {{{2
AddStyle {\ *// -ft=c \ &
AddStyle }\ *// -ft=c &

" # Multiple C++ namespaces on same line {{{2
AddStyle {\ *namespace -ft=cpp \ &
AddStyle }\ *} -ft=cpp &

" # Doxygen {{{2
" Doxygen Groups
AddStyle @{  -ft=c @{
AddStyle @}  -ft=c @}

" Doxygen Formulas
AddStyle \\f{ -ft=c \\\\f{
AddStyle \\f} -ft=c \\\\f}

" # Default style in C & al: Stroustrup/K&R {{{2
AddStyle {  -ft=c -prio=10 \ {\n
AddStyle }; -ft=c -prio=10 \n};\n
AddStyle }  -ft=c -prio=10 \n}

" # Inhibated style in C & al: Allman, Whitesmiths, Pico {{{2
" AddStyle {  -ft=c -prio=10 \n{\n
" AddStyle }; -ft=c -prio=10 \n};\n
" AddStyle }  -ft=c -prio=10 \n}\n

" # Ignore curly-brackets on single lines {{{2
AddStyle ^\ *{\ *$ -ft=c &
AddStyle ^\ *}\ *$ -ft=c &

" # Handle specifically empty pairs of curly-brackets {{{2
" On its own line
" -> Leave it be
AddStyle ^\ *{}\ *$ -ft=c &
" -> Split it
" AddStyle ^\ *{}\ *$ -ft=c {\n}

" Mixed
" -> Split it
" AddStyle {} -ft=c -prio=5 \ {\n}
" -> On the next line (unsplit)
AddStyle {} -ft=c -prio=5 \n{}
" -> On the next line (split)
" AddStyle {} -ft=c -prio=5 \n{\n}

" # Java style {{{2
" Force Java style in Java
AddStyle { -ft=java -prio=10 \ {\n
AddStyle } -ft=java -prio=10 \n}
```

When you wish to adopt Allman coding style, in `${project_root}/_vimrc_local.vim`
```vim
   AddStyle { -b -ft=c -prio=10 \n{\n
   AddStyle } -b -ft=c -prio=10 \n}
```


Local configuration (with "`-buffer`") have the priority over filetype
specialized configuration (with "`-ft`").


---

## API

This part is just a draft for the moment.

### Formatting API
TBC

* `lh#style#clear()`
* `lh#style#get()`
* `lh#style#get_groups()`
* `lh#style#apply()`
* `lh#style#apply_these()`
* `lh#style#reinject_cached_ignored_matches()`
* `lh#style#ignore()`
* `lh#style#just_ignore_this()`
* `lh#style#surround()`
* `lh#style#use()`
* `lh#style#define_group()`

### Naming API
TBC

* `lh#naming#variable()`
* `lh#naming#getter()`
* `lh#naming#setter()`
* `lh#naming#ref_getter()`
* `lh#naming#proxy_getter()`
* `lh#naming#global()`
* `lh#naming#local()`
* `lh#naming#member()`
* `lh#naming#static()`
* `lh#naming#constant()`
* `lh#naming#param()`
* `lh#naming#type()`
* `lh#naming#to_lower_camel_case()`
* `lh#naming#to_upper_camel_case()`
* `lh#naming#to_underscore()`

---

## Contributing
Contributions are welcomed. I've yet to write a proper CONTRIBUTING.md guide regarding copyright, licence and so on.

---
## Installation
  * Requirements:
      * Vim 7.+,
      * [lh-vim-lib](http://github.com/LucHermitte/lh-vim-lib) (v4.0.0),
      * [editorconfig-vim](https://github.com/editorconfig/editorconfig-vim) (optional).
  * Install with [vim-addon-manager](https://github.com/MarcWeber/vim-addon-manager) any plugin that requires lh-style should be enough.
  * With [vim-addon-manager](https://github.com/MarcWeber/vim-addon-manager), install lh-style (this is the preferred method because of the [dependencies](http://github.com/LucHermitte/lh-style/blob/master/addon-info.txt)).
```vim
ActivateAddons lh-style
" will also install editorconfig-vim
```
  * [vim-flavor](http://github.com/kana/vim-flavor) (which also supports
    dependencies)
```
flavor 'LucHermitte/lh-style'
" will also install editorconfig-vim
```
  * Vundle/NeoBundle:
```vim
Bundle 'LucHermitte/lh-vim-lib'
Bundle 'LucHermitte/lh-style'
" Optional
Bundle 'editorconfig/editorconfig-vim'
```
  * Clone from the git repositories
```
git clone git@github.com:LucHermitte/lh-vim-lib.git
git clone git@github.com:LucHermitte/lh-style.git
" Optional
git clone git@github.com:editorconfig/editorconfig-vim'
```
