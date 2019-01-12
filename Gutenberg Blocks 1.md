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
https://github.com/verytwisty/vt_gutenberg_testimonials

And all the presention files are located here
https://github.com/verytwisty/GutenbergTalk

^ Show the example editable block

---

# Getting started


---

## Development Environment

**Node and NPM**
NPM is a package manager that lets you install the packages and dependancies you need to have a working development environment.

**Webpack**
Webpack is a bundler / task runner that was originally designed for packaging up Javascript files, but has been extended with modules run sass and other files. It is similar to gulp and grunt in that you can use it to take your raw files and it will run tasks including error linting and minification and export production ready code.


---

## Getting set up

**NPM**
https://docs.npmjs.com/downloading-and-installing-node-js-and-npm

**Webpack**
https://webpack.js.org/guides/installation/

---

## What packages do I need?

![right fit](img/folder-layout.png)

**package.json**
https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/package.json

**webpack.config.js**
https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/webpack.config.js

In the terminal 

```
cd users/you/your-project-files-here/

npm i -D

```



^
In your working directory you will also need a package.json file. These will include all the packages you need for your current project. You may not need all of them in every project, but these are the ones we will use for our testimonials block.
We also need to add a webpack configuration file, so it knows where to save and what to do with your files, we won't worry about this for now, but it is set up for this project so you can play around with it
To install all the packages for this project go to the folder in the terminal and type in npm i -D, which installs everything in the working folder (not globally)

---
## Sumblime text users

Sublime text has problems with syntax highlighting JSX, Visual Studio and Atom have built in syntax highlighting.

To get round this you can install a sublime text package called Babel which will display it for you correctly

https://packagecontrol.io/packages/Babel

---
## Creating a plugin.



```php
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

// Enqueue JS and CSS
include __DIR__ . '/lib/enqueue-scripts.php';
 ```
[.footer: `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/vt_gutenberg_testimonials.php`]

^ 
Create a plugin the same way you would a normal plugin for WordPress.

---

## Backend JS and CSS files

[.code-highlight: 1-7]

```php
add_action( 'enqueue_block_editor_assets', 'enqueue_block_editor_assets' );

function enqueue_block_editor_assets() {

	wp_enqueue_script( 'testimonial-block-js', _get_plugin_url() . '/assets/js/editor.blocks.js', array( 'wp-editor', 'wp-blocks', 'wp-i18n', 'wp-element'), '1.0');
	wp_enqueue_style( 'testimonial-block-css', _get_plugin_url() . '/assets/css/blocks.editor.css', [ ], '1.0' );
}

add_action( 'enqueue_block_assets', 'enqueue_assets' );

function enqueue_assets() {

	wp_enqueue_style( 'testimonial-block', _get_plugin_url() . '/assets/css/blocks.style.css', [ ], '1.0' );
}

add_action( 'enqueue_block_assets', __NAMESPACE__ . '\enqueue_frontend_assets' );

function enqueue_frontend_assets() {

	if ( is_admin() ) {
		return;
	}
	wp_enqueue_script( 'testimonial-js-frontend', _get_plugin_url() . '/assets/js/frontend.blocks.js', [], '1.0' );
}

```

^ 3 functions in this file:
- Enque JS and CSS files that will be used in the backend only
- Enque CSS files that will be used in both the backend and the front end
- Enque JS that will be used on the front end only

^ `enqueue_block_editor_assets` Hook
- editor.blocks.js
	- This is the JS that will make your blocks work!
	- Make sure you load the file after all the other Gutenberg script dependancies
- blocks.editor.css
	- This is CSS that loads in styles that *should not* be loaded on the front end
	- This is good for styles for error messages, information etc

---

## Front & Back End CSS

[.code-highlight: 9-14]

```php
add_action( 'enqueue_block_editor_assets', 'enqueue_block_editor_assets' );

function enqueue_block_editor_assets() {

	wp_enqueue_script( 'testimonial-block-js', _get_plugin_url() . '/assets/js/editor.blocks.js', array( 'wp-editor', 'wp-blocks', 'wp-i18n', 'wp-element'), '1.0');
	wp_enqueue_style( 'testimonial-block-css', _get_plugin_url() . '/assets/css/blocks.editor.css', [ ], '1.0' );
}

add_action( 'enqueue_block_assets', 'enqueue_assets' );

function enqueue_assets() {

	wp_enqueue_style( 'testimonial-block', _get_plugin_url() . '/assets/css/blocks.style.css', [ ], '1.0' );
}

add_action( 'enqueue_block_assets', __NAMESPACE__ . '\enqueue_frontend_assets' );

function enqueue_frontend_assets() {

	if ( is_admin() ) {
		return;
	}
	wp_enqueue_script( 'testimonial-js-frontend', _get_plugin_url() . '/assets/js/frontend.blocks.js', [], '1.0' );
}

```

^ `enqueue_block_assets` hook
- This adds the styles for the block to both the front end and the backend
	- blocks should look as similar as possible on the front end as on the backend for a good editing experience



---

## Front end JS

[.code-highlight: 16-24]

```php
add_action( 'enqueue_block_editor_assets', 'enqueue_block_editor_assets' );

function enqueue_block_editor_assets() {

	wp_enqueue_script( 'testimonial-block-js', _get_plugin_url() . '/assets/js/editor.blocks.js', array( 'wp-editor', 'wp-blocks', 'wp-i18n', 'wp-element'), '1.0');
	wp_enqueue_style( 'testimonial-block-css', _get_plugin_url() . '/assets/css/blocks.editor.css', [ ], '1.0' );
}

add_action( 'enqueue_block_assets', 'enqueue_assets' );

function enqueue_assets() {

	wp_enqueue_style( 'testimonial-block', _get_plugin_url() . '/assets/css/blocks.style.css', [ ], '1.0' );
}

add_action( 'enqueue_block_assets', __NAMESPACE__ . '\enqueue_frontend_assets' );

function enqueue_frontend_assets() {

	if ( is_admin() ) {
		return;
	}
	wp_enqueue_script( 'testimonial-js-frontend', _get_plugin_url() . '/assets/js/frontend.blocks.js', [], '1.0' );
}

```

^ enqueue_block_editor_assets
- This adds JS to the front end only
	- Good for making things work like carousels, tabs etc
- use the 'enqueue_block_assets' hook and bail if this is being run on the back end.


--- 

# Basic Block Architecture

---

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```
[.footer: `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

---

[.code-highlight: 1-2]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```
[.footer: **Import Block dependancies**  `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

^Import Block dependancies
Import code from scss files and other js files that is used by the block.

---

[.code-highlight: 4-5]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```
[.footer: **Import Block dependancies**  `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

^Import Block Libraries
Import Gutenberg dependancies that are required in the file.
These are the two basic depenancies that you will need for every block, there are many others that we will look at later.
- registerBlockType is pulled in from wp.blocks. This allows you to register your block
- __ is pulled in from the wp.i18n library. This allows all your text strings to be translated.

---

## Example of translatable String


```

__('This string is ready for translations in the .pot file', '_vt');

```

---

[.code-highlight: 7, 32 ]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```
[.footer: **Import Block dependancies**  `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

^create the block
This exports the block ready for import into your main index.js file. All your block code will be within this:

---

[.code-highlight: 8 ]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```
[.footer: **Block Name**  `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

^
- The first part is the name space - important this is unique as we don't want this to conflict with other blocks
	- The name space can be the same across all your blocks
- The second part is your individual block name
	- This must be different across all your blocks

---

## Gotchas

- Block names must contain a namespace and be in the namespace/blockname format
- Blocknames can only lowercase alphanumerical letters.

---

[.code-highlight: 10 ]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```
[.footer: **Title**  `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]


^ User readable short title, best 2-3 words max

---


![70%](img/block-name.png)

---

[.code-highlight: 11 ]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```

[.footer: **Description**  `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

^ 
- Description of the block, this will appear on the side bar when the user selects the block

---


![80%](img/block-description.png)

---

[.code-highlight: 12 ]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```

[.footer: **Category**  `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

^ 
- This is the section the block will appear. Possible options are:
- common | widget | formatting | layout | embeds

---

![80%](img/block-category.png)

---

[.code-highlight: 13 ]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```

[.footer: **Icon**  `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

^ This is the icon that will be displayed in the editor.

---

## Icons can be either:

- A Dashicon
- An JSX svg icon

```js
icon: 'admin-site',

icon: <svg width="20px" height="20px" viewBox="0 0 384 512">
	<path d="M320 0H64C28.7 0 0 28.7 0 64v384c0 35.3 28.7 64 64 64h256c35.3 0 64-28.7 64-64V64c0-35.3-28.7-64-64-64zm32 448c0 17.6-14.4 
	32-32 32H64c-17.6 0-32-14.4-32-32V64c0-17.6 14.4-32 32-32h256c17.6 0 32 14.4 32 32v384zM192 288c44.2 0 80-35.8 80-80s-35.8-80-80-80-80 
	35.8-80 80 35.8 80 80 80zm0-128c26.5 0 48 21.5 48 48s-21.5 48-48 48-48-21.5-48-48 21.5-48 48-48zm46.8 144c-19.5 0-24.4 
	7-46.8 7s-27.3-7-46.8-7c-21.2 0-41.8 9.4-53.8 27.4C84.2 342.1 80 355 80 368.9V408c0 4.4 3.6 8 8 8h16c4.4 0 8-3.6 8-8v-39.1c0-14 9-32.9 
	33.2-32.9 12.4 0 20.8 7 46.8 7 25.9 0 34.3-7 46.8-7 24.3 0 33.2 18.9 33.2 32.9V408c0 4.4 3.6 8 8 8h16c4.4 0 
	8-3.6 8-8v-39.1c0-13.9-4.2-26.8-11.4-37.5-12.1-18-32.7-27.4-53.8-27.4z"/>
</svg>,
 ```

[.footer: convert svg to JSX with this: https://svg2jsx.herokuapp.com/]

---

[.code-highlight: 14-18 ]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```

[.footer: **Keywords**  | `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

^ Key words can help the user find your block when they search for them in the search field

---

![80%](img/block-keywords.png)

---

[.code-highlight: 19-24 ]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```

[.footer: **Edit**  | `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

^ To use JSX you need to have Bable installed, otherwise you'll have to create the markup using react or JavaScript

---


![fit](img/block-backend.png)

---

[.code-highlight: 25-30 ]

```js
import './editor.scss';
import './style.scss';

const { __ } = wp.i18n;
const { registerBlockType } = wp.blocks;

export default registerBlockType(
	'namespace/blockname',
	{
		title: __('Block Name', '_vt'),
		description: __('Description of the block', '_vt'),
		category: 'common',
		icon: dashicon || svgIcon,
		keywords: [
			__( 'Key Word 1', '_vt' ),
			__( 'Key Word 2', '_vt' ),
			__( 'Key Word 3', '_vt' ),
		],
		edit: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Back End </div>
			);
		},
		save: props => {
			const { className } = props;
			return (
				<div className={ className }>Block Front End</div>
			);
		},
	},
);
```

[.footer: **Save**  | `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/0-starter-block/index.js`]

---


![fit](img/block-frontend.png)

---

# Static Testimonial Block

^ Switch screen to show the block in situ

---

```js
edit: props => {
	const { className, isSelected } = props;
	return (
		<div className={ className }>
			<div className="image">
				<img src="/wp-content/plugins/vt_testimonials/assets/images/David-Brent.jpg" />
			</div>
			<div className="info">
				<h2>David Brent</h2>
				<h3>General Manager</h3>
				<div className="text">
					<p>People see me, and they see the suit, and they go: 'you're not 
					fooling anyone', they know I'm rock and roll through and through.</p> 
					<p>But you know that old thing, live fast, die young? Not my way. 
					Live fast, sure, live too bloody fast sometimes, but die young? Die old.</p>
				</div>
				<a href="https://www.wernham-hogg-limited.com" className="website">Wernham Hogg</a>
			</div>
			{
				isSelected && (
					<div className="warn"><p>Sorry you can't edit this block</p></div>
				)
			}
		</div>
	);
},

```

[.footer: **Static Block**  | `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/1-static-block/index.js`]

^ The First Block we are going to make is a Static Block that is not editable by the user

---

[.code-highlight: 2 ]

```js
edit: props => {
	const { className, isSelected } = props;
	return (
		<div className={ className }>
			<div className="image">
				<img src="/wp-content/plugins/vt_testimonials/assets/images/David-Brent.jpg" />
			</div>
			<div className="info">
				<h2>David Brent</h2>
				<h3>General Manager</h3>
				<div className="text">
					<p>People see me, and they see the suit, and they go: 'you're not 
					fooling anyone', they know I'm rock and roll through and through.</p> 
					<p>But you know that old thing, live fast, die young? Not my way. 
					Live fast, sure, live too bloody fast sometimes, but die young? Die old.</p>
				</div>
				<a href="https://www.wernham-hogg-limited.com" className="website">Wernham Hogg</a>
			</div>
			{
				isSelected && (
					<div className="warn"><p>Sorry you can't edit this block</p></div>
				)
			}
		</div>
	);
},

```

[.footer: **Static Block**  | `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/1-static-block/index.js`]

^ First we pull in the props (or properties) of our new Block. This structure allows you to call named variables in the props object to refer back to. It is the same as writing props.className

---

## Let's Talk Props

**JavaScript Object**
attributes: {} // where to store variables 

**Generated Strings**
className: "wp-block-testimonials-static" // Unique Class Name so styles don't conflict
clientId: "2e1f42d4-9c2f-434a-9b68-b037a545c438"
name: "testimonials/static"

**Bolean**
isSelected: false // If the block is selected or not
isSelectionEnabled: true

**Functions**
insertBlocksAfter: ƒ ()
mergeBlocks: ƒ ()
onReplace: ƒ ()
setAttributes: ƒ () // function to call to set variables
toggleSelection: ƒ ()

--- 

## JavaScript Dot Notation

```js
props{
	attributes: {},
	className: "wp-block-testimonials-static",
	clientId: "2e1f42d4-9c2f-434a-9b68-b037a545c438",
	insertBlocksAfter: ƒunction( ... ),
	isSelected: false,
	isSelectionEnabled: true,
	mergeBlocks: ƒunction( ... ),
	name: "testimonials/static",
	onReplace: ƒunction( ... ),
	setAttributes: ƒunction( ... ),
	toggleSelection: ƒunction( ... ),
}
```

```
{ className } = props

props.className === className;


```

---

[.code-highlight: 3-25 ]

```js
edit: props => {
	const { className, isSelected } = props;
	return (
		<div className={ className }>
			<div className="image">
				<img src="/wp-content/plugins/vt_testimonials/assets/images/David-Brent.jpg" />
			</div>
			<div className="info">
				<h2>David Brent</h2>
				<h3>General Manager</h3>
				<div className="text">
					<p>People see me, and they see the suit, and they go: 'you're not 
					fooling anyone', they know I'm rock and roll through and through.</p> 
					<p>But you know that old thing, live fast, die young? Not my way. 
					Live fast, sure, live too bloody fast sometimes, but die young? Die old.</p>
				</div>
				<a href="https://www.wernham-hogg-limited.com" className="website">Wernham Hogg</a>
			</div>
			{
				isSelected && (
					<div className="warn"><p>Sorry you can't edit this block</p></div>
				)
			}
		</div>
	);
},
```

[.footer: **Static Block**  | `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/1-static-block/index.js`]

^ Everything within the return statement is what will be displayed in the Gutenberg backend - this is the JSX / HTML 

---

## Gotchas when writing JSX

- If you forget a closing tag, the build will fail
- Class is a protected word in JS, use className instead
- You can write JS and call variables within curly braces in the JSX ` { javascript }`
- All the JSX must be contained within one parent element - no adjacent elements

```html
**This is allowed**
<div className="container">
	<div className="another-container"></div>
	<div className="another-container"></div>
</div>

**This is will error**
<div className="another-container"></div>
<div className="another-container"></div>
```

---

[.code-highlight: 19-23 ]

```js
edit: props => {
	const { className, isSelected } = props;
	return (
		<div className={ className }>
			<div className="image">
				<img src="/wp-content/plugins/vt_testimonials/assets/images/David-Brent.jpg" />
			</div>
			<div className="info">
				<h2>David Brent</h2>
				<h3>General Manager</h3>
				<div className="text">
					<p>People see me, and they see the suit, and they go: 'you're not 
					fooling anyone', they know I'm rock and roll through and through.</p> 
					<p>But you know that old thing, live fast, die young? Not my way. 
					Live fast, sure, live too bloody fast sometimes, but die young? Die old.</p>
				</div>
				<a href="https://www.wernham-hogg-limited.com" className="website">Wernham Hogg</a>
			</div>
			{
				isSelected && (
					<div className="warn"><p>Sorry you can't edit this block</p></div>
				)
			}
		</div>
	);
},
```

[.footer: **Static Block**  | `https://github.com/verytwisty/vt_gutenberg_testimonials/blob/master/blocks/1-static-block/index.js`]

^ The Block can look different when the user has selected the block - this can be useful for adding information or displaying information

---

## Confusing Syntax?

- {} denotes there will be some JS
- `isSelected &&` is equivalent to `if( isSelected == true )`
- () denotes that there will be some JSX to display if the condition is met


```js
{
	isSelected && (
		<div className="warn"><p>Sorry you can't edit this block</p></div>
	)
}
```

---

## All information from this talk has been learnt on Zac Gordon's Gutenberg Block Development Course, please check it out for more detail and information

https://javascriptforwp.com/product/gutenberg-block-development-course/

---

## Resources

[Gutenberg starter theme](https://github.com/ahmadawais/create-guten-block)
[Gutenberg block reference ](https://wp-storybook.netlify.com/?selectedKind=Components%7CBaseControl&selectedStory=Basic&full=0&addons=1&stories=1&panelRight=1&addonPanel=storybook%2Factions%2Factions-panel)
[Coding a Custom Block Type for Gutenberg Block Editor (video) ](https://www.youtube.com/watch?v=Mv68Sa-iHyo&feature=youtu.be)
[WordPress Webinar: Building your First Gutenberg Block (video) ](https://www.youtube.com/watch?v=2wM6VyJ9Dp4)

