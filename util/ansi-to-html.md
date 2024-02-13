Install `ansifilter`, e.g. (`brew install ansifilter`).

Use:

```
./target-verus/release/verus --color=always --crate-type=lib program.rs 2>&1 | ansifilter --html --map path/to/util/ansi-map-dark -o file.html
```

`ansi-map-dark` is in this folder.

Manually extract the `pre` block, wrap the contents in a `code` block, and add the `class="ansi-html"` class to the `pre` block.
