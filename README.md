# Head First TiKV

**Head First TiKV** contains a series of articles to show [TiKV](https://github.com/tikv/tikv) - a distributed, transactional key-value database. Through these essays, we want to give you a deep introduction of TiKV, including but not limited to:

- The whole architecture of TiKV
- The detailed designs for different hierarchy layers of TiKV
- The techniques we choose to build TiKV
- Any problem we meet when we build TiKV and how we solve them
- How to build your own services based on TiKV

## Reading the Content

You can visit [the Github Pages site](https://tikv.github.io/head-first-tikv) to see the latest master version.

## Contributing to this repo

If you want to contribute to this repo, you can:

1. Mark the section you want to write in the [Outline](https://github.com/tikv/head-first-tikv/issues/1). If you want to add new sections, you can file an issue and discuss with us. After the proposal is approved, we will add it to the Outline and mark you as the writer.
2. Create the document in the specified chapter directory. If the directory doesn't exist, please create by yourself. For example, if you want to write the "introduction" section, you should create an `src/overview` directory and a file `src/overview/introduction.md` in this directory.
3. Don't forget to add your section to the catalog in "SUMMARY.md".
4. Enjoy writing, you can review your changes with `mdbook serve` and checking `localhost:3000`. (Install it with `cargo install mdbook`)
5. Send a PR. :)
