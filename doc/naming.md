# Naming policy


Thanks to lh-style, we can define how the names of functions, classes, constants, attributes, etc. shall be written: in
`UpperCamelCase`, in `lowerCamelCase`, in `snake_case`, or in `Any_otherStyle`).

This information can then be retrieved by plugins through the [Naming API](API.md/#naming-api).

This information can also be used from `:NameConvert` and `:ConvertNames` commands.

NB: both commands support command-line auto-completion on naming policy names.

## `:NameConvert`

`:NameConvert` converts the identifier under the cursor to one of the following naming policies:
  * naming styles: `upper_camel_case`, `lower_camel_case`, `underscore`/`snake`, `variable`,
  * identifier kinds: `getter`, `setter`, `local`, `global`, `member`, `static`, `constant`, `param` (the exact conversion process can be tuned thanks to the [following options](#options-to-tune-the-naming-policy)).

## `:ConvertNames`
`:[range]ConvertNames/pattern/policy/[flags]` transforms according to the _policy_ all names that match the _pattern_ -- it applies `:NameConvert` on text matched by `:substitute`.

## Options to tune the naming policy

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

Once in the *main name* form, the `..._re` regex options match the *main name* while the `..._subst` replacement text is applied instead.

You can find examples for these options in mu-template
[template](http://github.com/LucHermitte/mu-template/blob/master/after/template/vim/internals/vim-rc-local-cpp-style.template)
used by [BuildToolsWrapper](http://github.com/LucHermitte/BuildToolsWrapper)'s `:BTW new_project` command.

