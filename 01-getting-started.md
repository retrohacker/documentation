# Getting Started

### 1. Create an account

This part is really simple:

1. [**Sign up**](https://app.storj.io/#/signup) for a Storj account.
2. Check your email for an activation link.
3. No step 3, you're already signed up.

### 2. Get the tools

We recommend building your application using our Javascript library, Storj.js.
It works in Node and in the browser.

To install via npm (for Node.js or [browserify](http://browserify.org/) et.al.
development):

```sh
npm install --save storj
```

To download the pre-built browser libraries, check our
[releases page](https://github.com/Storj/storj.js/releases).
Use `storj.es5.js` for older browsers, or `storj.es6.js` for modern browsers.

The browser libraries can also be built from source by cloning our github repo.

```sh
git clone git@github.com:Storj/storj.js.git storj.js
cd storj.js
npm build
```

Now that the setup is out of the way, let's dive in to the
[`Storj` object](02-storj-object.md)
