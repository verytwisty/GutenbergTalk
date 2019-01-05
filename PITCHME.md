autoscale: true

# [fit] Creating a Testimonials Block.

---
## This group of talks will teach you:

- Setting up the development environment
- Setting up a block
- Creating a static Testimonials block
- Making the Testimonials block have editable content
- Extending the functionality of the block
- Creating a Testimonials Custom Post Type with Gutenberg templates
- Creating a Dynamic Block
- Other useful features

All the code I'm going to use is here: so you can follow along if you wish.
!!! Git hub link to block file

And all the presention files are located here
!!! Link to the presentation on Git Hub

---

## All information from this talk has been learnt on Zac Gordon's Gutenberg Block Development Course, please check it out for more detail and information

https://javascriptforwp.com/product/gutenberg-block-development-course/

---



# [fit] Getting started



---

## Installing NPM

NPM is a package manager that lets you install the packages and dependancies you need to have a working development environment.

Node.js and NPM are installed together - to install them go here and download the files:

https://nodejs.org/en/

You also need to install webpack globally.

```
npm install --save-dev webpack
```

---
## What is Webpack

Webpack is a bundler / task runner that was originally designed for packaging up Javascript files, but has been extended with modules run sass and other files. It is similar to gulp and grunt in that you can use it to take your raw files and it will run tasks including error linting and minification and export production ready code.

https://webpack.js.org/

---
## Why do I need Webpack?

If Webpack is similar to Gulp and Grunt why do I need it? It has one advantage for Javascript in that if you have an index.js and include all the JS files in there in the order you want them to load, webpack will concatinate and minify them in the order you specify. Gulp and Grunt don't concatinate files in this way, so you could potentially end up with dependancy errors if the JS files are concatinated in the wrong order.

---
## What packages do I need?

In your working directory you will also need a package.json file. These will include all the packages you need for your current project. You may not need all of them in every project, but these are the ones we will use for our testimonials block.

```js
{
    "@wordpress/babel-preset-default": "^1.2.0", // this is for JSX
    "babel-core": "^6.26.3", // Translates React and JSX into JavaScript
    "babel-eslint": "^8.2.3", // error handling
    "babel-loader": "^7.1.4", // Allows transpiling JavaScript files
    "classnames": "^2.2.6", // lets you add more than one classname in React
    "cross-env": "^5.1.5", // Makes the dev environment work on PCs as well as Macs
    "css-loader": "^0.28.11", // loads css
    "eslint": "^4.19.1", // JS error handling
    "extract-text-webpack-plugin": "^3.0.2", // compiles files css into one main css file
    "node-sass": "^4.9.0", // compiles sass to css
    "postcss-loader": "^2.1.5", // loads postcss which automagically adds vendor prefixes to css
    "raw-loader": "^0.5.1", // A loader for webpack that allows importing files as a String works with sass loader
    "sass-loader": "^6.0.7", // turns sass into css works with node-sass
    "style-loader": "^0.19.1", // adds css to the DOM
    "webpack": "^3.11.0" // Loads Webpack into current project
}
```

---
## What packages do I need?

!!! Link to example JSON file here.

Save this in the root of your working folder and then install all the packages in the list.

In the terminal 

```
cd users/you/your-project-files-here/

npm i -D

```

---
## Webpack Configuration file

We also need to add a webpack configuration file, so it knows where to save and what to do with your files
- In the package.json this line will let webpack know where the main .js file is that needs to be loaded in.
```json
"main": "blocks/index.js",
```
The webpack file tells webpack what to do with the files

--- 
## Webpack Configuration file

For the CSS

```js
const blocksCSSPlugin = new ExtractTextPlugin( {
  filename: './assets/css/blocks.style.css',
} );
const editBlocksCSSPlugin = new ExtractTextPlugin( {
  filename: './assets/css/blocks.editor.css',
} );
```

For the JS

```js
  entry: {
    './assets/js/editor.blocks' : './blocks/index.js',
    './assets/js/frontend.blocks' : './blocks/frontend.js',
  },
```
!!! Link to the Webpack Config file

---
## Sumblime text users

Sublime text has problems with syntax highlighting JSX, Visual Studio and Atom have built in syntax highlighting.

To get round this you can install a sublime text package called Babel which will display it for you correctly

https://packagecontrol.io/packages/Babel

---
## Creating a plugin.

Create a plugin the same way you would a normal plugin for WordPress.
```
/**
 * Plugin Name: Gutenberg Testimonials
 * Plugin URI:  https://github.com/verytwisty/vt_gutenberg_testimonials
 * Description: A Testimonials Plugin inspired by <a href="https://gutenberg.courses/development">Zac Gordon's Gutenberg Development Course</a> for the MWUG talk.
 * Version:     1.0
 * Author:      Belinda Mustoe
 * Author URI:  https://verytwisty.com
 * Text Domain: vt_testimonials
 * Domain Path: /languages
 * License:     GPL2+
 * License URI: http://www.gnu.org/licenses/gpl-2.0.html
 */
 ```

---

## Enqueue Scripts and Styles

```php
// Enqueue JS and CSS
include __DIR__ . '/lib/enqueue-scripts.php';
```

!!! Link to file


---

## Folder Structure (optional)

-> assets   - concatinated js, css and img files that will be loaded by WP
-> blocks   - Raw React and Sass files you will be working on
-> lib      - PHP files you may need to add in functionality.
-> (node_modules) - ignore

(package.json) - ignore
(webpack.js)   - ignore
`vt_gutenberg_testimonials.php` - plugin file

!!! Link to repro

---
## Enqueing the js and sass

3 functions in this file:

- Enque JS and CSS files that will be used in the backend only
- Enque CSS files that will be used in both the backend and the front end
- Enque JS that will be used on the front end only


!!! Link to file

---

## Backend JS and CSS files

```php
add_action( 'enqueue_block_editor_assets', __NAMESPACE__ . '\enqueue_block_editor_assets' );
function enqueue_block_editor_assets() {

	$block_path = '/assets/js/editor.blocks.js';
	$style_path = '/assets/css/blocks.editor.css';

	wp_enqueue_script(
		'testimonial-block-js',
		_get_plugin_url() . $block_path,
		array(
			'wp-editor', 'wp-blocks', 'wp-i18n', 'wp-element',
		),
		filemtime( _get_plugin_directory() . $block_path )
	);
	wp_enqueue_style(
		'testimonial-block-css',
		_get_plugin_url() . $style_path,
		[ ],
		filemtime( _get_plugin_directory() . $style_path )
	);
}
```

---

## What does this do?

- editor.blocks.js
	- This is the JS that will make your blocks work!
	- Make sure you load the file after all the other Gutenberg script dependancies

- blocks.editor.css
	- This is CSS that loads in styles that *should not* be loaded on the front end
	- This is good for styles for error messages, information etc

- Use the `enqueue_block_editor_assets` hook.

---

## Front and backend CSS files

- This adds the styles for the block to both the front end and the backend
	- blocks should look as similar as possible on the front end as on the backend for a good editing experience
- Use the `enqueue_block_assets` hook

```php
add_action( 'enqueue_block_assets', __NAMESPACE__ . '\enqueue_assets' );

function enqueue_assets() {
	$style_path = '/assets/css/blocks.style.css';
	wp_enqueue_style(
		'testimonial-block',
		_get_plugin_url() . $style_path,
		null,
		filemtime( _get_plugin_directory() . $style_path )
	);
}
```

---

## Front end JS

- This adds JS to the front end only
	- Good for making things work like carousels, tabs etc
- use the 'enqueue_block_assets' hook and bail if this is being run on the back end.

```php
add_action( 'enqueue_block_assets', __NAMESPACE__ . '\enqueue_frontend_assets' );
function enqueue_frontend_assets() {

	if ( is_admin() ) {
		return;
	}

	$block_path = '/assets/js/frontend.blocks.js';
	wp_enqueue_script(
		'testimonial-block-frontend',
		_get_plugin_url() . $block_path,
		[],
		filemtime( _get_plugin_directory() . $block_path )
	);
}
```

---

## Basic Block File Structure

-> Block-Name 
	-> index.js     - This is where the React code for the Block Goes
	-> style.scss   - This is where the main styles for the Block Goes (front and backend)
	-> editor.scss  - This is where the editor styles for the Block Goes

index.js      - Import all your block here.
frontend.js   - Import all JS files that only display on the front end here.

*Don't forget to import all your blocks into the main index.js file as they will not be compiled into the final output until you do.*

```js

import "./Block-Name";

```

--- 

# Basic Block Architecture

---
## Import Block dependancies

Import code from scss files and other js files that is used by the block.

```js
import './editor.scss';
import './style.scss';
import icon from './icon';
```

!!! Link to file

---
## Import Block Libraries

Import Gutenberg dependancies that are required in the file.

```js
const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;
```
These are the two basic depenancies that you will need for every block, there are many others that we will look at later.
- registerBlockType is pulled in from wp.blocks. This allows you to register your block
- __ is pulled in from the wp.i18n library. This allows all your text strings to be translated.

```

__('This string is ready for translations in the .pot file', '_vt');

```

---

## create the block

This exports the block ready for import into your main index.js file. All your block code will be within this:

```jsx
export default registerBlockType(

);
```

---
## Block Name

```jsx

export default registerBlockType(
	'namespace/blockname',
);

```

- The first part is the name space - important this is unique as we don't want this to conflict with other blocks
	- The name space can be the same across all your blocks
- The second part is your individual block name
	- This must be different across all your blocks

*Gotchas*

- Block names must contain a namespace and be in the namespace/blockname format
- Blocknames can only lowercase alphanumerical letters.

---

## Title

- User readable short title, best 2-3 words max

```js
export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Name the user will see', '_vt'),
	}
);
```

---
## Description

- Description of the block, this will appear on the side bar when the user selects the block

```js
export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Name the user will see', '_vt'),
		description: __('Description of the block', '_vt'),
	}
);
```

---
## Category

- This is the section the block will appear. Possible options are:

	- common | widget | formatting | layout | embeds

!!! Need to check exact wording here.

```js
export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Name the user will see', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
	}
);
```

---
## Icon

This is the icon that will be displayed in the editor.


```js
export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Name the user will see', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon
	}
);
```

---

## Icons can be either:

- A Gravatar

```js
icon: 'admin-site',
```
 - An JSX svg icon

 ```jsx
<svg width="20px" height="20px" viewBox="0 0 384 512">
	<path d="M320 0H64C28.7 0 0 28.7 0 64v384c0 35.3 28.7 64 64 64h256c35.3 0 64-28.7 64-64V64c0-35.3-28.7-64-64-64zm32 448c0 17.6-14.4 32-32 32H64c-17.6 0-32-14.4-32-32V64c0-17.6 14.4-32 32-32h256c17.6 0 32 14.4 32 32v384zM192 288c44.2 0 80-35.8 80-80s-35.8-80-80-80-80 35.8-80 80 35.8 80 80 80zm0-128c26.5 0 48 21.5 48 48s-21.5 48-48 48-48-21.5-48-48 21.5-48 48-48zm46.8 144c-19.5 0-24.4 7-46.8 7s-27.3-7-46.8-7c-21.2 0-41.8 9.4-53.8 27.4C84.2 342.1 80 355 80 368.9V408c0 4.4 3.6 8 8 8h16c4.4 0 8-3.6 8-8v-39.1c0-14 9-32.9 33.2-32.9 12.4 0 20.8 7 46.8 7 25.9 0 34.3-7 46.8-7 24.3 0 33.2 18.9 33.2 32.9V408c0 4.4 3.6 8 8 8h16c4.4 0 8-3.6 8-8v-39.1c0-13.9-4.2-26.8-11.4-37.5-12.1-18-32.7-27.4-53.8-27.4z"/>
</svg>
 ```

---
## Keywords

Key words can help the user find your block when they search for them in the search field

```js
export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Name the user will see', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),

		],
	}
);
```

---

## Attributes (not needed for static blocks)

This is like 'State' in react and are where the information is stored that the user enters before it is saved into the Database.

```js
export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Name the user will see', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),

		],
		attributes: {
			attr: {
				type: 'array',
			},
		},
	}
);
```

---

## edit



---
## save
