# Code formatting

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

* Plugin maintainers will use the [dedicated API](#formatting-api) to reformat on-the-fly the code they generate.
* End-users will specify the coding style used on their project(s):
    - either by specifying a set of independent styles on different topics (families) (`:UseStyle`, `.editorconfig`, `.clang-format`),
    - or by being extremely precise (`:AddStyle`).


## Style families

### Families already implemented

At this time, the following style families are implemented:

* [EditorConfig styles](https://github.com/editorconfig/editorconfig/wiki/EditorConfig-Properties#ideas-for-domain-specific-properties):
   - `curly_bracket_next_line`  = [`yes`, `no`] // `true,` `false,` `0,` `1`.
   - `indent_brace_style`       = [`0tbs`, `1tbs`, `allman`, `bsd_knf`, `gnu`, `horstmann`, `java`, `K&R`,
                                 `linux_kernel`, `lisp`, `none`, `pico`, `ratliff`, `stroustrup`, `whitesmiths]`
   - `spaces_around_brackets`   = [`inside`, `outside`, `both`, `none`]

* [Clang-Format styles](https://clangformat.com/):
   - `breakbeforebraces`        = [`allman`, `attach`, `gnu`, `linux`, `none`, `stroustrup` ]
   - `spacesbeforeparens`       = [`none`, `never`, `always`, `control-statements`]
   - `spacesinemptyparentheses` = [`yes`, `no`] // `true,` `false,` `0,` `1`.
   - `spacesinparentheses`      = [`yes`, `no`] // `true,` `false,` `0,` `1`.


Styles [can easilly be added](#extending-the-families) in `{&rtp}/autoload/lh/style/`.

If you want more precise control, without family management, you can use [`:AddStyle`](#addstyle) instead.

### `:UseStyle`
`:UseStyle {style-family}={value} [-buffer] [-ft[={ft}]] [-prio={prio}]`

Given *style families*, `:UseStyle` can be used to specify which particular style shall be used when generating code.

For instance,

```vim
:UseStyle breakbeforebraces=Allman -ft=c
:UseStyle spacesbeforeparens=control-statements -ft=c
:UseStyle spacesinparentheses=no
```

will tune snippets/abbreviations to follow [Allman indenting style](https://en.wikipedia.org/wiki/Indentation_style#Allman_style), and to add a space before control-statement parentheses, and to never insert a space inside parentheses.

Note that some families are incompatible with other families. It happens quickly when we mix overlapping families from
different origins.

#### `:UseStyle` options
*  `{style-family}={value}` specifies, given a style family, what choice has been made.

    If you don't remember them all, don't worry, `:UseStyle` supports command-line completion.

    Setting the style to `none` unsets the whole family for the related
    [`buffers`](http://vimhelp.appspot.com/windows.txt.html#buffers)/[`filetype`](http://vimhelp.appspot.com/filetype.txt.html#filetype).
    Giving a new _{value}_, overrides the style for the related
    [`buffers`](http://vimhelp.appspot.com/windows.txt.html#buffers)/[`filetype`](http://vimhelp.appspot.com/filetype.txt.html#filetype).

* "`-buffer`" defines this association only for the current buffer. This option is meant to be used with plugins like [local_vimrc](https://github.com/LucHermitte/local_vimrc).
* "`-ft[={ft}]`" defines this association only for the specified filetype. When `{ft}` is not specified, it applies only to the current filetype. This option is meant to be used in .vimrc, in the global zone of [filetype-plugin](http://vimhelp.appspot.com/usr_43.txt.html#filetype%2dplugin)s or possibly in [local_vimrcs](https://github.com/LucHermitte/local_vimrc) (when combined with "`-buffer`").
* "`-prio={prio}`" Sets a priority that'll be used to determine which key is matching the text to enhance. By default all styles have a priority of 1. The typical application is to have template expander ignore single curly brackets.

### `.editorconfig`
lh-style registers a hook to [editorconfig-vim](https://github.com/editorconfig/editorconfig-vim) in order to extract
style choices expressed in any `.editorconfig` file that applies.

The syntax would be:

```ini
[*]
indent_brace_style=Allman
```

In every buffer where EditorConfig applies its settings, it will be translated into:

```vim
:UseStyle -b indent_brace_style=allman
```


### `.clang-format`
The idea is the same: to detect automatically a `.clang-format` configuration file in project root directory and apply
the styles supported by lh-style.

At this time, this feature isn't implemented yet.

### Extending the families
TBD

---

## Low-level style configuration
Historically, there wasn't any way to group style configurations as [`:UseStyle`](#usestyle)
permits. We add to define everything manually, and switching from one complex
configuration to another was tedious.

While using `:UseStyle` is now the preferred method, we can still use the low
level method.

### `:AddStyle`

`:AddStyle {key} [-buffer] [-ft[={ft}]] [-prio={prio}] {Replacement}`
  * `{key}` is a regex that will get replaced automatically (by plugins supporting this API)
  * `{replacement}` is what will be inserted in place of `{key}`
  * "`-buffer`" defines this association only for the current buffer. This option is meant to be used with plugins like [local\_vimrc](https://github.com/LucHermitte/local_vimrc).
  * "`-ft[={ft}]`" defines this association only for the specified filetype. When `{ft}` is not specified, it applies only to the current filetype. This option is meant to be used in .vimrc, in the global zone of [filetype-plugin](http://vimhelp.appspot.com/usr_43.txt.html#filetype%2dplugin)s or possibly in [local\_vimrcs](https://github.com/LucHermitte/local_vimrc) (when combined with "`-buffer`").
  * "`-prio={prio}`" Sets a priority that'll be used to determine which key is matching the text to enhance. By default all styles have a priority of 1. The typical application is to have template expander ignore single curly brackets.

#### (Deprecated) `:AddStyle` Examples:

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
