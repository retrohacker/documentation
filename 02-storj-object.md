The Storj library exposes all of its tools via the `Storj` object. It helps you
upload, download, and interact with your account. Most Storj methods follow a
simple callback pattern.

Setting up a `Storj` object is easy:

```javascript
// Node:
var Storj = require('storj')
```

```html
// Browsers:
<script src="storj.es6.js"></script>
```

```javascript
// Both
var storj = new Storj()
```

Once you have the `Storj` object, you can use it to do all sorts of things.
However, most of them require authentication. So let's work on that next.

### 1. Authentication Station

Storj allows two kinds of authentication: basic auth with a username and
password (the default), or key-based authentication. For now, let's set up a
`Storj` object with basic auth. We do this by passing in an `opts` object that
contains the basic auth information:

```javascript
var opts = {
  basicAuth: {
    user: "user@email.com",
    password: "this is not a secure password"
  }
  mnemonic: mnemonic // we'll come back to this later
}

var storj = new Storj(opts)
```

Key-based auth generates an ECDSA keypair, and registers it to your account.
Because key-based auth is more secure, we recommend using it whenever possible.
For more information on ECDSA keypairs in Storj, see [Chapter 6:
Keys and Keys](06-keys.md).

First we need to generate a keypair:

```javascript
var keypair = storj.generateKeyPair()
var privkey = keypair.getPrivateKey()
var pubkey = keypair.getPublicKey()
```

Once the keypair has been made, we register the public key with Storj using the
`registerKey` function.

```javascript
storj.registerKey(pubkey, function (error) {
  console.log(error)
})
```

After the keypair has been registered, it can be used to authenticate instead
of a username and password.

```javascript
var opts = {
  key: privkey,
  mnemonic: mnemonic // we'll come back to this later
}

var storj = new Storj(opts)
```

Now you're using keypair auth. Make sure to store the private key somewhere
safe!

An account can have any number of keys registered. We recommend generating a
new keypair for each device used for development. In the future, we'll add
features to track actions by key, and provide tools for assigning permissions
on a key-by-key basis.

### 2. Create a Bucket

Once you're authenticated, you can start using Storj. The first step is to make
a bucket.

```javascript
storj.createBucket("My New Bucket", function (error, meta){
  if (error) {
    console.log(error)
  }
  console.log("id:", meta.id)
  console.log("name:", meta.name)
})
```

Files in Storj are kept in buckets. Buckets help you organize files by project,
by type, or by user. Buckets are private by default, but can be opened to the
public.

Buckets are usually referenced by ID, so make sure you keep track of that. Here
are a few self-explanatory bucket operations: `deleteBucket`, `updateBucket`,
`getBucket`, `makePublic`.

### What's next?

See below for a review of the code so far, or head on to
[Uploading and Downloading Files](03-upload-download.md)


### Putting it all together

```javascript
// In Node:
var Storj = require('storj')

// In browsers:
<script src="storj.es6.js"></script>

// Create an unauthenticated Storj object
var storj = new Storj()

// Authenticate with basicAuth
var opts = {
  basicAuth: {
    user: "user@email.com",
    password: "this is not a secure password",
  }
  mnemonic: mnemonic // we'll come back to this later
}

var storj = new Storj(opts)

// Create a new keypair
var keypair = storj.generateKeyPair()
var privkey = keypair.getPrivateKey()
var pubkey = keypair.getPublicKey()

// Register the public key to your account
storj.registerKey(pubkey, function (error) {
  console.log(error)
})

// Authenticate with the keypair
var opts = {
  key: privkey,
  mnemonic: mnemonic // we'll come back to this later
}

var storj = new Storj(opts)

// Create a bucket
storj.createBucket("My New Bucket", function (error, meta){
  if (error) {
    console.log(error)
  }
  console.log("id:", meta.id)
  console.log("name:", meta.name)
})
```

Let's push on to [Uploading and Downloading](03-upload-download.md)?
