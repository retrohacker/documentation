We briefly covered key-based authentication earlier, but it's worth talking
more about the two different kinds of keys used in Storj. We use authentication
keys to authenticate users, and file keys to encrypt and decrypt files. Modern
cryptography makes Storj safer and more secure than any other object store.

### 1. Authentication keys

Storj uses keypairs to authenticate clients. Authentication keys are ECDSA
keypairs, which are used to sign and verify each request to the Storj API. Each
keypair has a private key for generating signatures, and a public key for
checking signatures. Keypairs are generated locally, and then the public key is
registered to your account.

The code is below, but don't forget to review [Chapter 2](02-storj-object.md)
for a more in-depth explanation of making and registering keys.

```javascript
var keypair = storj.generateKeyPair()

var privkey = keypair.getPrivateKey()
var pubkey = keypair.getPublicKey()

storj.registerKey(pubkey, function (error) {
  console.log(error)
})
```

When you call `storj.registerKey()` you send Storj your public key. Once we
have that, we can use it to verify signatures on future requests. That's how we
can be sure requests come from you. **Make sure you keep you private keys
secure** as anyone with them can access your account.

If a private key is compromised, you can revoke access to it by removing its
pubkey from your account:

```javascript
storj.removeKey(pubkey, function (error) {
  console.log(error)
})
```

Using keys for account access is safer than using passwords. They're longer, and
harder to guess, for one thing. They can also be used to identify devices: just
register a different public key for each device. Or, you can use them to grant
access to another person. It's easier to revoke a key than to change a
password, and we're working on more tools to control account permissions on a
key-by-key basis.

### 2. File keys

File keys are used to encrypt files before uploading, and to decrypt files
after downloading. This protects the contents of the file from the people
storing it, from anyone listening in, and from Storj. Encryption, decryption,
and generation are all handled by the `Storj` object.

Each file uses a different key. This means that to use a file after you
download it, you need its key. For convenience and portability file keys are
generated deterministically. This is done using a strong mnemonic passphrase.
The passphrase is combined with the file ID to make the file key. This means
that instead of having to move all file keys to new devices, we can just move
the mnemonic.

#### Passphrases

Passprases can be generated via `storj.generateMnemonic()`. We recommend
doing this only once. **Make sure to keep the passphrase in a safe place!**
Anyone with the passphrase can make your file keys, and if you lose the
passphrase, you may lose access to your files! The passphrase is a 12-word long
phrase, designed to be easy to write down, or even memorize.

The passphrase should be included in the `opts` argument of the `Storj` object.
`Storj` objects without passphrases can't upload or download from private
buckets, as they can't generate file keys.

```javascript
// Make sure to save this passphrase!
// Use the same passphrase every time!
var mnemonic = storj.generateMnemonic()

var opts = {
  key: privkey,
  mnemonic: mnemonic
}

var storj = new Storj(opts)
```

For server-side apps, passphrases should be stored locally, and injected via an
environment variable. For browser apps, require the user to input the
mnemonic passphrase, and keep it in memory. Don't send the mnemonic over the
wire. :)

#### Public buckets

For public buckets, instead of using the passphrase, a bucket key is generated.
The bucket key is stored in the bucket, and provided to any client requesting a
file from that bucket. This bucket key is combined with the file ID to make the
file key. This means that files can still be uniquely encrypted, and publicly
accessible.

### Conclusion

Keys take a minute to wrap your head around, so here's the basics: auth keys
give account access. The passphrase gives file access. Clients need both
account access AND file access to deal work with files in private buckets.
Clients need neither to work with files in public buckets.

Next, let's put together a (very) simple read-only
[image gallery](07-gallery.md).
