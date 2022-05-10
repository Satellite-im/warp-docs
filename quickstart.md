# Quickstart

This guide will help you get Warp up and running locally.

#### Dependencies

Warp requires you to have [Rust](https://www.rust-lang.org/) installed locally. You should follow [this guide](https://www.rust-lang.org/tools/install) to set up Rust on your system.

If you will be updating the docs, you'll also need [NodeJS](https://nodejs.org/en/) installed as well as [DocsifyJS](https://docsify.js.org/#/quickstart). Once installed you can run `docsify serve docs` to get a live preview, hot reloading, and local instance of our docs.

#### Choosing Extensions

Warp ships with a few included extensions and many additional extensions may be available from the public. To configure your extensions simply clone their repo in the `extensions` directory, and start warp with the TUI enabled to choose your extensions.

By default, Warp is set up to use the [StorJ](https://docs.storj.io/dcs/) extension. You should follow [this guide](extensions/constellation/storj) and then return to this page after you've completed up to the `adding keys to Tesseract` step.

#### Starting Warp

After you've installed your dependencies you're ready to start Warp for the first time. You should have either used the terminal UI or command line to inject your required API keys into `Tesseract`. Now you're ready to start Warp. 

To get going, run `cargo run --bin warp`.
