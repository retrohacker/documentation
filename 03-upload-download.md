Now that we have a bucket ready to go, let's get some files in it. Storj makes
file uploading and downloading files easy.

Let's start by instantiating a `Storj` object, and authenticating with our
private key. We should also set up a variable for the bucket id, just for
convenience.

```javascript
var encryptionKey = Storj.generateEncryptionKey()
var storj = new Storj({key: privateKey, encryptionKey: encryptionKey})
bucketId = 'your_bucket_id'
```

### 1. Introducing the encryption key

In the last document, we introduced you to the private/public keypair. This is
used for authenticating to the Storj.js bridge instead of using your username
and password. That keypair can be used for creating and managing buckets, and
other aspects of a user account.

In order to work with files however, you will need a key to encrypt/decrypt the
file itself. This is where the encryption key comes in. As you can see in the
example above, you can generate a new encryption key by calling
`Storj.generateEncryptionKey()`. This key is in the format of a friendly
mnemonic, 12-24 words long!

```
console.log(Storj.generateEncryptionKey())
```

### 2. Upload a file
The `Storj` object can upload files in three formats: `stream.Readable`,
`Blob` and `String`. Because a javascript `File` object is a type of `Blob` we
can use those too.

Let's grab a file and upload it. In Node we can get a file from the filesystem:

```javascript
// Node:
var fs = require('fs')
var data = fs.readFileSync('/path/to/file/cat.jpg',)
```

In a browser we can get files from a file input:

```html
<!-- Broswers: -->
<input type="file" id="input" onchange="upload()">

<script>
function upload() {
  var data = document.getElementById('input').files[0]
}
</script>
```

Once we have the file, we can upload it to our bucket. We pass in the bucket
id, a name for the file, and the data:

```javascript
// Both:
var file = storj.createFile(bucketId, 'cat.jpg', data)
file.on('error', console.log)
file.on('done', function() {
    console.log('Created file with id: ' + file.id)
})

```

### 3. Download a file

Now that we have the file uploaded, let's pull it back. Downloading is even
easier than uploading:

```javascript
fileId = file.id
var download = storj.getFile(bucketId, fileId)
download.on('error', console.log)
download.on('done', function() {
  /* download is ready to use */
})
```

When a file is uploaded, it's assigned an id. Files are referenced by bucket id
and file id. That way we can have many files or buckets with the same name,
without worrying about collisions or ambiguity.

### 4. Get comfortable with the Storj `File`

Both `storj.createFile()` and `storj.getFile()` return a Storj `File` object.
When we upload or download a file, there's a lot going on behind the scenes.
Before leaving the client machine, uploaded files are encrypted and split into
pieces. When downloading a file, the pieces are retrieved from several sources,
reassembled, and decrypted. Fortunately, the `File` object handles all of that
for you.

Storj `File` objects have a few nice properties to help us learn about it, and
methods to help us work with it. Let's log a few file properties, to get a feel
for what we're doing.

```javascript
console.log("name:", file.name)
console.log("type:", file.mimetype)
console.log("size in bytes:", file.length)
console.log("upload/download progress:", file.progress)
console.log("uploaded/downloaded bytes:", file.progress * file.length)
```

The Storj `File` object also allows us to set event listeners, just in case we
care about what our `File` is up to.

```javascript
download.on('ready', function () {
  console.log('The file object is now ready to upload/download data.')
})

download.on('done', function () {
  console.log('Upload/download is complete!')
})

download.on('error', function (error) {
  console.log('The file has encountered some sort of error.')
  console.log(error)
})
```

### 5. Do something with it!

Storj `File` objects are versatile. Rather than forcing you into a specific
code style, they expose the underlying data in several different formats,
including `stream.Readable`s, `Buffer`s, `Blob`s, and even direct DOM injection.

We're going to gloss over DOM operations and focus on node for a second.
But don't worry, we'll cover it in depth in the next chapter. Let's look at
some ways of accessing our downloaded data, first with `Buffer`s:

```javascript
download.getBuffer(function (error, buffer) {
  if (error) {
    console.log(error)
  }
  doTheThingWithThe(buffer)
})
```

`getBuffer()` calls back when the file is completely downloaded. But what if we
want access to data as it comes in?

```javascript
var downStream = download.createReadStream()

downStream.on('data', function (data) {
  console.log('bytes downloaded:', file.length)
  doTheThingWithThe(data)
})

downStream.on("end", function() {
  console.log('Stream over.')
})

// Node:
var catFile = fs.createWriteStream('./cat.jpg')
downStream.pipe(catFile)
```

The stream makes data available as soon as it's retrieved, and can be piped
into any writable stream.

### Now what?

Keep scrolling down for a review of file operations, or jump straight into
[Storj in a Browser](04-browser.md). Backend engineers can skip ahead to
[Advanced Bucketing](05-bucket-ops.md).

### Putting it all together

Node.js:

```javascript
var Storj = require('storj')
var fs = require('fs')

// Instantiate a new Storj object with your private key
var storj = new Storj({key: privateKey, mnemonic: mnemonic})
var data = fs.readFileSync('/path/to/file/cat.jpg',)

var file = storj.createFile(bucketId, 'cat.jpg', data)
file.on('error', console.log)
file.on('done', fucntion () {
  console.log('Upload complete')
  var fileId = file.id
  var download = storj.getFile(bucketId, fileId)
  download.on('error', console.log)
  download.on('ready', function () {
    console.log('The file object is now ready to upload/download data.')
  })
  download.on('done', function () {
    // Log the file information
    console.log("name:", download.name)
    console.log("type:", download.mimetype)
    console.log("size in bytes:", download.length)
    console.log("upload/download progress:", download.progress)
    console.log("uploaded/downloaded bytes:", download.progress * download.length)
    var downStream = download.createReadStream()
    var catFile = fs.createWriteStream('./cat2.jpg')
    downStream.pipe(catFile)
  })
})
```

Browser:

```html
<html>
<body>
  <input type="file" id="input" onchange="upload()">
  <script>
  var encryptionKey = Storj.generateEncryptionKey()
  var storj = new Storj({ key, encryptionKey })
  function upload() {
    var data = document.getElementById('input').files[0]
    var file = storj.createFile(bucketId, 'cat.jpg', data)
    file.on('error', console.log)
    file.on('done', function () {
      console.log('Upload complete')
      var fileId = file.id
      var download = storj.getFile(bucketId, fileId)
      download.on('error', console.log)
      download.on('ready', function () {
        console.log('The file object is now ready to upload/download data.')
      })
      download.on('done', function () {
        // Log the file information
        console.log("name:", download.name)
        console.log("type:", download.mimetype)
        console.log("size in bytes:", download.length)
        console.log("upload/download progress:", download.progress)
        console.log("uploaded/downloaded bytes:",
            download.progress * download.length)
      })
    })
  }
  </script>
</body>
</html>
```

In either, once you have the `download` `file` object, you can use these
methods to play with the downloaded data!

```javascript
// Get the file contents as a buffer
download.getBuffer(function (error, buffer) {
  if (error) {
    console.log(error)
  }
  doTheThingWithThe(buffer)
})

// And/or a readable stream for piping fun
var downStream = download.createReadStream()

downStream.on('data', function (data) {
  console.log('bytes downloaded:', file.length)
  doTheThingWithThe(data)
})

downStream.on("end", function() {
  console.log('Stream over.')
})
```

Next up: [Storj in a Browser](04-browser.md)
Or, if front-end isn't your thing, move forward two spaces to
[Advanced Bucketing](05-bucket-ops.md)
