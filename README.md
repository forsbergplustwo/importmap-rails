# Importmap for Rails

[Import maps](https://github.com/WICG/import-maps) let you import JavaScript modules using logical names that map to versioned/digested files – directly from the browser. So you can build modern JavaScript applications using JavaScript libraries made for ESM without the need for transpiling or bundling. Without the need for such preprocessing, you can build advanced Rails applications without Webpack, Yarn, NPM, or any other part of the JavaScript toolchain. All you need is the asset pipeline that's already included in Rails.

With this approach you'll ship many small JavaScript files instead of one big JavaScript file. Thanks to HTTP2 that no longer carries a material performance penalty during the initial transport, and in fact offers substantial benefits over the long run due to better caching dynamics. Whereas before, any change to any JavaScript file included in your big bundle would invalidate the cache for the the whole bundle, now only the cache for that single file is invalidated.

There's [native support for import maps in Chrome/Edge 89+](https://caniuse.com/?search=importmap), and [a shim available](https://github.com/guybedford/es-module-shims) for any browser with basic ESM support. So your app will be able to work with all the evergreen browsers.


## Installation

1. Add `importmap-rails` to your Gemfile with `gem 'importmap-rails'` (make sure it's included before any gems using it!)
2. Run `./bin/bundle install`
3. Run `./bin/rails importmap:install`

By default, all the files in `app/assets/javascripts` and the three major Rails JavaScript libraries are already mapped. You can add more mappings in `config/initializers/assets.rb`.


## Usage

The import map is configured programmatically through the `Rails.application.config.importmap.paths` assignment, which by default is setup in `config/initializers/assets.rb` after running the installer. (Note that since this is a config initializer, you must restart your development server after making any changes.)

This programmatically configured import map is inlined in the `<head>` of your application layout using `<%= javascript_importmap_tags %>`, which will setup the JSON configuration inside a `<script type="importmap">` tag. After that, the [es-module-shim](https://github.com/guybedford/es-module-shims) is loaded, and then finally the application entrypoint is imported via `<script type="module">import "application"</script>`. That logical entrypoint, `application`, is mapped in the importmap script tag to the file `app/assets/javascripts/application.js`, which is copied and digested by the asset pipeline.

It's in `app/assets/javascripts/application.js` you setup your application by importing any of the modules that have been defined in the import map. You can use the full ESM functionality of importing any particular export of the modules or everything.

It makes sense to use logical names that match the package names used by NPM, such that if you later want to start transpiling or bundling your code, you'll not have to change any module imports.


## Use with Hotwire

This gem was designed for use with [Hotwire](https://hotwired.dev) in mind. The Hotwire gems, like [turbo-rails](https://github.com/hotwired/turbo-rails) and [stimulus-rails](https://github.com/hotwired/stimulus-rails) (both bundled as [hotwire-rails](https://github.com/hotwired/hotwire-rails)), are automatically configured for use with `importmap-rails`. This means you won't have to manually setup the path mapping in `config/initializers/assets.rb`, and instead can simply refer to the logical names directly in your `app/assets/javascripts/application.js`, like so:

```js
import "@hotwired/turbo-rails"
import "@hotwired/stimulus-autoloader"
```


## Expected errors from using the es-module-shim

While import maps are native in Chrome and Edge, they need a shim in other browsers that'll produce a JavaScript console error like `TypeError: Module specifier, 'application' does not start with "/", "./", or "../".`. This error is normal and does not have any user-facing consequences.


## License

Importmap for Rails is released under the [MIT License](https://opensource.org/licenses/MIT).


