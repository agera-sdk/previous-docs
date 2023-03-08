# Internationalization Basics

Rialight apps place their language resources at the `res/lang` directory.
`res/lang` contains directories named as locale identifiers, such as `en-us`.
These locale directories contain FTL files of the extension `.ftl`.

FTL stands for Fluent Translation List. A FTL basically defines messages for a specific locale.

## FTL Syntax

[See guide for FTL syntax.](https://github.com/projectfluent/fluent/tree/master/guide)

## Working With FTL

The Rialight app contains a Rust module at the file `src/app_ftl.rs`, which looks like this:

```rust
use rialight::intl::ftl::{Ftl, FtlOptions, FtlOptionsForAssets, FtlLoadMethod};
use rialight::util::{hashmap};
use std::sync::Arc;

pub fn create() -> Arc<Ftl> {
    Arc::new({
        let mut app_ftl = Ftl::new(
            FtlOptions::new()
                // specify supported locales.
                // the form in which the locale identifier appears here
                // is a post-component for the assets "src" path. 
                // for example: "path/to/res/lang/en-US"
                .supported_locales(vec!["en"])
                .default_locale("en")
                // .fallbacks(hashmap! {
                //     "xx" => vec!["xy"],
                // })
                .assets(FtlOptionsForAssets::new()
                    .source("app://res/lang")
                    .files(vec!["_"])
                    // "clean_unused" indicates whether to clean previous unused locale data. 
                    .clean_unused(true)
                    // specify FtlLoadMethod::FileSystem or FtlLoadMethod::Http
                    .load_method(FtlLoadMethod::FileSystem)))
        ;
        app_ftl.initialize_locale(|_locale, _bundle| {
            //
        });
        app_ftl
    })
}
```

This code tells how to resolve the FTL resources and which locales are supported.
The `Ftl` type is the most important type for resolving messages.
It requires a `_.ftl` file for each supported locale.

Let's suppose `res/lang/en/_.ftl` contains this content:

```text
hello-world = Hello, world!
```

The following code creates a `Ftl` using the above `create()` function, attempts to load the default locale, `"en"`, and prints `"Hello, world!"` to the console:

```rust
let mut app_ftl = crate::app_ftl::create();
if !app_ftl.load(None).await {
    return;
}
println!("{}", app_ftl.get_message("hello-world", None, &mut vec![]).unwrap());
```

Meaning of the arguments to `app_ftl.get_message`:

- The first argument is the message identifier.
- The second argument is an optional arguments map, which is `None` in this case.
- The third argument is the destination of any errors while resolving the message.
In this case we are ignoring any errors with a throwaway vector.

## Arguments

Arguments maps for `app_ftl.get_message_string` can be literally created with the `arguments!` macro:

```rust
use rialight::intl;
let arguments = intl::ftl::arguments!{ "x" => "y" };
```
