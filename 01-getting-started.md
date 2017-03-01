# Getting Started

### 1. Create an account

This part is really simple:

1. [**Sign up**](https://app.storj.io/#/signup) for a Storj account.
2. Check your email for an activation link.
3. No step 3, you're already signed up.

### 2. Get the tools

We recommend building on our Javascript library, Storj.js. It works in Node and
in the browser.

To install via npm:
```
npm install --save storj
```

To download the pre-built browser libraries, check our
[releases page](https://github.com/Storj/storj.js/releases).
Use `storj.es5.js` for older browsers, or `storj.es6.js` for modern browsers.

The browser libraries can also be built from source by cloning our github repo.

```
git clone git@github.com:Storj/storj.js.git storj.js
cd storj.js
npm build
```

### 3. Create a Bucket

Files in Storj are kept in buckets. Buckets help you organize files by project,
by type, or by user. Each bucket can have separate permissions.
