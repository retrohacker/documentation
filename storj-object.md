

### 1. Authentication and Instantiation Station

Storj allows two kinds of authentication: basic auth with a username and
password (the default), or key-based authentication. For now, let's set up a
storj object with basic auth.

```
var opts = {
  basicAuth: {
    user: "user@email.com",
    password: "this is not a secure password"
  }
}

var storj = new Storj(opts)
```

Key-based auth generates an ECDSA keypair, and registers it to your account.
Because key-based auth is more secure, we recommend using it whenever possible.

### 2. Create a Bucket

Files in Storj are kept in buckets. Buckets help you organize files by project,
by type, or by user. Buckets can have separate permissions, or even be made
public.
