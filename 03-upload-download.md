Now that we have a bucket ready to go, let's get some files in it. Storj makes
file uploading and downloading files easy.

Let's start by instantiating a `Storj` object, and authenticating with our
private key. We should also set up a variable for the bucket ID, just for
convenience.

```javascript
var storj = new Storj({key: privkey, mnemonic: mnemonic})
bucketID = 'your_bucket_id'
```

### 1. Upload a file
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
<input type="file" id="input">

<script>
var data = document.getElementById('input').files[0]
</script>
```

Once we have the file, we can upload it to our bucket. We pass in the bucket
ID, a name for the file, and the data:

```javascript
// Both:
var file = storj.createFile(bucketID, 'cat.jpg', data)
```

### 2. Download a file

Now that we have the file uploaded, let's pull it back. Downloading is even
easier than uploading:

```javascript
fileID = file.id
var download = storj.getFile(bucketID, fileID)
```

When a file is uploaded, it's assigned an id. Files are referenced by Bucket ID
and File ID. That way we can have many files or buckets with the same name,
without worrying about collisions or ambiguity.

### 3. Get comfortable with the Storj `File`

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
console.log("uploaded/downloaded bytes:", file.progress * file.bytes)
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

### 4. Do something with it!

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
```javascript
// Instantiate a new Storj object with your private key
var storj = new Storj({key: privkey, mnemonic: mnemonic})

// Node:
var fs = require('fs')
var data = fs.readFileSync('/path/to/file/cat.jpg',)

// Browsers:
<input type="file" id="input">

<script>
var data = document.getElementById('input').files[0]
</script>

// Both:
// Upload the file
var file = storj.createFile(bucketID, 'cat.jpg', data, function () {
  console.log('Upload complete')
})

// Download the file
fileID = file.id
var download = storj.getFile(bucketID, fileID)

// Log the file information
console.log("name:", download.name)
console.log("type:", download.mimetype)
console.log("size in bytes:", download.length)
console.log("upload/download progress:", download.progress)
console.log("uploaded/downloaded bytes:", download.progress * download.bytes)

// Register file event listeners
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

// Set up a Buffer callback
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

// Node:
var catFile = fs.createWriteStream('./cat.jpg')
downStream.pipe(catFile)
```

Next up: [Storj in a Browser](04-browser.md)
Or, if front-end isn't your thing, move forward two spaces to
[Advanced Bucketing](05-bucket-ops.md)
