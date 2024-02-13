

## How to add a post

Add a `.md` file to `content/`. Take a look at `content/gentle-introduction-to-verus.md` for an example.

## How to preview the site

1. Make sure that git submodules have been initialized:
    ```sh
    git submodule init && git submodule update
    ```

2. Make sure you've got [Zola](https://www.getzola.org/documentation/getting-started/installation/) installed.

3. Run:

    ```sh
    zola serve
    ```

4. Open the url printed by Zola (likely, http://127.0.0.1:1111/)

### How to add colorized terminal output

Follow the instructions in [util/ansi-to-html.md](util/ansi-to-html.md).
