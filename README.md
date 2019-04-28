**The aim of this branch is just to test this repository with many use cases.
Use master branch instead!**

# Webpack, Vue, SSR project template

Includes:

* Webpack 4
* [polka](https://github.com/lukeed/polka) web-server
* [buble](https://github.com/Rich-Harris/buble) transpiler (light-weight babel)
* Vue 2 with SSR, Vuex and vue-loader
* Stylus with [kouto-swiss](http://kouto-swiss.io/)
* Axios
* Pug
* SVG sprites builder

## Getting started

```bash
npm i

# development server on localhost:8080
npm run dev

# production build
npm run build

# production server on localhost:8080
npm start
```

## .env configuration

You can configure application port, HTTP proxy and base path for API.

```bash
cp .env.example.js .env.js

# modify .env.js file, see the file itself for more information
```

## Application structure

* `index.js` - application server
* `assets/` - application static assets (images, fonts, icons etc.)
	* `sprite.svg` - generated sprites file, `require('assets/sprite.svg')` will return file contents string
	* `docs/` - downloadable static content
	* `fonts/` - guess what
	* `i/` - static images (backgrounds, patterns etc.)
	* `icons/` - contains SVG icons for the sprite
* `build/` - code related to project building
	* `setup-dev-server` - development server setup with hot reloading
	* `svg-sprite` - svg sprite generation script, gathers icons from `assets/icons` and compiles them into `assets/sprite.svg`
	* `webpack/` - webpack config, `base` - common, `server` for server with SSR, `client` for browser
* `dist/` - production build files
* `src/`
	* `entry/` - main entry points
		* `app` - shared between server and client, exports a factory function returning root component instance, mixes it with `app.vue`
		* `client` - client entry
		* `server` - server entry
	* `components/` - vue components
		* `shared/` - components registered implicitly via `Vue.component()`
	* `pages/` - components here are implicitly attached to routes same with componets\' file names
		(excluding leading `_` in file or folder names and `404.vue` which will be used as a catch-all route)
	* `filters/` - vue filters registered implicitly via `Vue.filter()`
	* `directives/` - vue directives registered implicitly via `Vue.directive()`
	* `mixins/` - vue mixins
	* `store/` - Vuex storage, `index` returns a factory function returning configured Vuex store instance
	* `app.vue` - aplication root component, implicitly mixed with `entry\app`
	* `http` - exports http client instance (Axios)
	* `layout.pug` - application HTML layout
	* `router` - exports a factory function returning vue-router instance
	* `shared.styl` - globally included stylus file (for variables, mixins, etc.)

## SSR related component features

Every component within `src/pages` directory can use some special features providing full SSR support:

* `component.routePath`, String - additional route suffix. Usually used to provide dynamic route segments.
	You can use any string allowed for the vue-router path definition. All dynamic segments are automatically mapped
	to component `props`.
* `component.routeMeta`, Object - `route.meta`. Include `statusCode` here to modify an HTTP status returned with SSR.
	404 route includes 404 status code by default.
* `component.prefetch({ store, props, route })`, function
	(`store` - vuex store instance, `props` - route params, `route` - current route object).
	
	Must return a promise. Allows some async routine before actual application rendering on server side.
	To pass any data to component: resolve promise with needed data and add corresponding `component.data` fields with
	their initial values to prevent *...property or method not defined...* error.
	Automatically called on client side from `beforeMount` and `beforeRouteChange` hooks as well.
	See `src/mixins/prefetch` mixin.

	**IMPORTANT: there is no component context within `prefetch` function because component instance is not created yet!**

`prefetch` also works on the root component (`src/app.vue`) with some restrictions:

* no `props` field in the argument;
* no way to pass component data (only store can be affected).

Root application component (`app.vue`) has a `serverPrefetched` property.
It is always set to `true` on client side and after all prefetching is done on server side.
It is not reactive until you explicitly declare it in component`s data.

## Development checklist

* SSR configurable cache
* basic Nginx + Phusion Passenger configuration example
