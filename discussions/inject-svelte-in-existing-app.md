Inject svelte in an existing app
==============
2020-05-05


This is a memo for myself.


So let's say you have a svelte component:



**file: MyComponent.svelte**
```html
I'm a svelte component
```

And an existing page:

**file: index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width,initial-scale=1'>
    <title>Svelte app</title>
</head>

<body>

<h1>My app</h1>

</body>
</html>

```


How do you inject your component in the html page?

Surprisingly, at the time of writing this memo (2020-05-05), I didn't found a simple
answer on the svelte website, hence this post.

Here is what I do (this might not be the recommended method).


1. Inject a target in your app.

Sounds obvious, but we can't inject directly the `<MyComponent>` tag in the html, because that's not valid html.
Instead, we can inject a target element like this one: `<div id="my-component"></div>`:


**file: index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width,initial-scale=1'>
    <title>Svelte app</title>
</head>

<body>

<h1>My app</h1>

<div id="my-component"></div>

</body>
</html>

```

Now we need some js code to transform our target into our svelte component. Let's add the js we would like:


**file: index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width,initial-scale=1'>
    <title>Svelte app</title>
    <script defer src='/my-component/dist/bundle.js'></script>
</head>

<body>

<h1>My app</h1>

<div id="my-component"></div>

<script>
    document.addEventListener("DOMContentLoaded", function (event) {
        new MyComponent({
            target: document.getElementById("my-component"),
        });
    });
</script>

</body>
</html>

```


Notice that we added two things: the **bundle.js** in the head (we will create them later), 
and the script at the bottom of the body, to effectively insert our **MyComponent** into the desired target.

Note: optionally we can add a **bundle.css** link in the head if our svelte component actually uses some css styling, but for the sake
of simplicity we will ignore it for now.
  


What's left to do at this point is create the **bundle.js** file which will connect our pieces together.



The building of the component
---------------


First, let's talk a bit about structure. For this example, we will have this structure:


```text
- /myapp/
----- public/                               (the web server's root dir)
--------- index.html                        (the index.html file that we created above)
--------- my-component/
------------- src/            
----------------- MyComponent.svelte        (the MyComponent.svelted created above)
----------------- main.js                   ( a new file that we will create below)
------------- dist/
----------------- bundle.js                 (not created yet, but our goal is to create this file)
----------------- bundle.css                (not created yet, but our goal is to create this file if our component has some css styling)
```



So first let's create the **main.js** file, which will be used in the process:


**file: /myapp/public/my-component/src/main.js**
```js
import SomeComponent from './MyComponent.svelte';


window.MyComponent = function (options) {
    return new SomeComponent({
        target: options.target
    });
};

```

Notice how this file creates a js global variable **MyComponent** and assigns the svelte component to it. 
Thanks to that global variable we can now call our svelte component in **index.html**.



Now how do we create the **bundle.js** file.


Let's use **rollup**.

So here is the rollup config I use:


**file: /myapp/rollup.config.js**
```js
import svelte from 'rollup-plugin-svelte';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import livereload from 'rollup-plugin-livereload';
import { terser } from 'rollup-plugin-terser';
import { sass } from 'svelte-preprocess-sass';




const production = !process.env.ROLLUP_WATCH;

export default {
	input: 'public/my-component/src/main.js', // (1)
	output: {
		sourcemap: true,
		format: 'iife',
		name: 'app',
		file: 'public/my-component/dist/bundle.js' // (2)
	},
	plugins: [
		svelte({
			// enable run-time checks when not in production
			dev: !production,
			// we'll extract any component CSS out into
			// a separate file - better for performance
			css: css => {
				css.write('public/my-component/dist/bundle.css'); // (3)
			},
			preprocess: {
				style: sass(),
			},
		}),

		// If you have external dependencies installed from
		// npm, you'll most likely need these plugins. In
		// some cases you'll need additional configuration -
		// consult the documentation for details:
		// https://github.com/rollup/plugins/tree/master/packages/commonjs
		resolve({
			browser: true,
			dedupe: ['svelte']
		}),
		commonjs(),

		// In dev mode, call `npm run start` once
		// the bundle has been generated
		!production && serve(),

		// Watch the `public` directory and refresh the
		// browser on changes when not in production
		!production && livereload('public'), // (4)

		// If we're building for production (npm run build
		// instead of npm run dev), minify
		production && terser(),
	],
	watch: {
		clearScreen: false
	}
};

function serve() {
	let started = false;

	return {
		writeBundle() {
			if (!started) {
				started = true;

				require('child_process').spawn('npm', ['run', 'start', '--', '--dev'], {
					stdio: ['ignore', 'inherit', 'inherit'],
					shell: true
				});
			}
		}
	};
}

```

You might want to change the lines with the numbers (1), (2), (3) and (4).




Before we can execute rollup we need to install some packages.

Let's create our **package.json**.


**file: /myapp/package.json**

```js
{
  "name": "svelte-app",
  "version": "1.0.0",
  "scripts": {
    "build": "rollup -c",
    "dev": "rollup -c -w",
    "start": "sirv public"
  },
  "devDependencies": {
    "@rollup/plugin-commonjs": "11.0.2",
    "@rollup/plugin-node-resolve": "^7.0.0",
    "node-sass": "^4.14.0",
    "rollup": "^1.20.0",
    "rollup-plugin-copy": "^3.3.0",
    "rollup-plugin-livereload": "^1.0.0",
    "rollup-plugin-svelte": "^5.0.3",
    "rollup-plugin-terser": "^5.1.2",
    "svelte": "^3.0.0",
    "svelte-preprocess-sass": "^0.2.0"
  },
  "dependencies": {
    "sirv-cli": "^0.4.4"
  }
}

```

If necessary, change the **sirv public** command to your webserver's root.


Let's switch to the terminal and install the dependencies:

```bash 
cd /myapp
npm install
```

Now we shall be able to call rollup.

We have to flavours, either use **npm run dev**, or use **npm run build**.

Both commands will call rollup, but **npm run dev** will start an http server and auto-refresh the page when you save a file according to your rollup configuration (see number (4)).

Here I will use **npm run dev** 


```bash
cd /myapp
npm run dev 
```

This will call rollup, which in turn will parse our **/myapp/public/my-component/src/main.js** file and
create our bundle file(s) (js and optionally css).

Additionally, an http server instance will be launched...



So that's it.
That's one way of injecting some svelte component into your existing app.

Cheers.




