Inject svelte in an existing html page
==============
2020-05-05 -> 2020-05-06




When you first encounter a new technology, like **svelte** for instance, a natural thing to do is to test it 
in a simple html page to see how it fits.

This is what we are going to do here.


This recipe assumes that:

- you know the basics of html, css and javascript
- you know what a webserver is and you know how to spawn one
- you have node installed on your machine, you know how to install an npm package, you are familiar with the package.json file
- you know how to open a terminal and type some commands


Let's get started.


Summary
---------
* [Create a test folder for this recipe](#create-a-test-folder-for-this-recipe)
* [Create an html page](#create-an-html-page)
* [Create a svelte component](#create-a-svelte-component)
* [Preparing the html page](#preparing-the-html-page)
 * [Inject a target element in your html](#inject-a-target-element-in-your-html)
* [The building of the component](#the-building-of-the-component)
 * [structure overview](#structure-overview)
 * [The main.js file](#the-mainjs-file)
 * [The rollup configuration](#the-rollup-configuration)
 * [The package.json file](#the-packagejson-file)
 * [rolling up our component](#rolling-up-our-component)




Create a test folder for this recipe
---------

Open a terminal:

```bash 
mkdir /myapp
cd /myapp
```

Of course, replace **myapp** with whatever directory you want. 


We will also create a **public** directory, which will be our webserver's root directory.


```bash
mkdir public
cd public
```


Create an html page
---------


**file: /myapp/public/index.html**
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



Create a svelte component
--------

Now that we have an html page, let's create a svelte component.

We will create a very simple svelte component called **MyComponent**. You can put it where you want.
For now, I'll put mine in a **my-component** directory (I recommend that you do the same for now,
you can always change your mind later).


```bash
cd /myapp/public 
mkdir my-component
cd my-component
```


Also, inside our component's folder, we will create a **src** and a **dist** directory. 

```bash 
mkdir src
mkdir dist
cd src
```


The **src** directory will be used to put all our source code. And the **dist** directory will contain the compiled version
of our source code (i.e. you need to compile a svelte component before you can use it in an html page, we will see how to compile
later in this document).


Now let's create our svelte component.


**file: /myapp/public/my-component/src/MyComponent.svelte**

```html
I'm a svelte component
```



Preparing the html page
-----------

Now that we have both an html page and a svelte component, how do you inject your svelte component in the html page?



### Inject a target element in your html


What we would like to do is call an `<MyComponent>` tag directly in the html. 
Unfortunately that's not a valid html tag so it won't work.


Instead, we can inject a target element like this one: `<div id="my-component"></div>`.

Update your **index.html** file:


**file: /myapp/public/index.html**

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

Now we need to transform our target into our svelte component. 

Let's update our **index.html** again.


**file: /myapp/public/index.html**
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

Notice that we added two things: the **bundle.js** reference in the head (we will create the actual file later), 
and the script at the bottom of the body, to effectively insert our svelte component (**MyComponent**) into 
the desired target (`<div id="my-component"></div>`).


Note: optionally we can add a **bundle.css** link in the head if our svelte component actually uses some css styling, but for the sake
of simplicity we will ignore it for now.
  

But right now, this code won't work, because the **bundle.js** file doesn't exist.
So in the next step we will create it, and the **index.html** will work as expected.



The building of the component
---------------


### structure overview
First, let's talk a bit about structure. For this example, we will have this structure:


```text
- /myapp/
----- public/                               (the web server's root dir)
--------- index.html                        (the index.html file that we created above)
--------- my-component/
------------- src/            
----------------- MyComponent.svelte        (the MyComponent.svelte component created above)
----------------- main.js                   (a new file that we will create below)
------------- dist/
----------------- bundle.js                 (not created yet, but our goal is to create this file)
----------------- bundle.css                (not created yet, but our goal is to create this file if our component has some css styling)
```




### The main.js file

Now let's talk about the general strategy to build/compile our component into the **bundle.js** file.

Fortunately for us, all the tools we need are already built for us.

We will use a tool called [rollup](https://rollupjs.org/guide/en/), which has the ability to compile our svelte component
into the **bundle.js** file.


Create the following **main.js** file:


**file: /myapp/public/my-component/src/main.js**
```js
import SomeComponent from './MyComponent.svelte';


window.MyComponent = function (options) {
    return new SomeComponent(options);
};

```

This file will be required/used by rollup later.

Notice how this file creates a js global variable **MyComponent** and assigns the svelte component to it. 
It's that same global variable that we referenced from the **index.html**.





### The rollup configuration

And now the scary part: the rollup configuration!


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


If you didn't use the same paths as I did, you might want to change the lines with the numbers (1), (2), (3) and (4).




### The package.json file


At this point, we are almost ready to execute rollup.
Before we do so, let's install the packages required by rollup for a smooth execution. 

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

If necessary, change the **sirv public** command (in the scripts section of the package.json) to your 
webserver's root (for instance sirv www).



Let's switch to the terminal and install the dependencies:

```bash 
cd /myapp
npm install
```



### rolling up our component

At this point, all the packages we need are installed, and we can rollup our component.

Open a terminal and type the following:

```bash
cd /myapp
npm run dev 
```

This will tell rollup to parse our **/myapp/public/my-component/src/main.js** file and
create our **bundle.js** file (and optionally the **bundle.css** of your component used some css).


Additionally, an http server instance will be launched at url: http://localhost:5000/ (by default).


The svelte component now appears in our app as expected.


So that's it.
That's one way of injecting some svelte component into your existing app.





