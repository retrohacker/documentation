Now that we have a bucket ready to go, let's get some files in it. Storj makes
file uploading and downloading files easy.

Let's start by instantiating a `Storj` object, and authenticating with our
private key. We should also set up a variable for the bucket ID, just for
convenience.

```javascript
var storj = new Storj({key: privkey})
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
// Browsers:
<input type="file" id="input">

<script>
var data = document.getElementById('input').files[0]
</script>
```

Once we have the file, we can upload it to our bucket. We pass in the bucket
ID, a name for the file, the data, and a callback:

```javascript
// Both:
var file = storj.createFile(bucketID, 'cat.jpg', data, function () {
  console.log('Upload complete')
})
```

### 2. Get comfortable with the Storj File

Like several other methods, `storj.createFile()` returns a Storj `File` object.
This object has a few nice properties and methods to help us learn about it
and work with it. Let's log a few file properties, to get a feel for what we're
doing.

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
file.on('ready', function () {
  console.log('The file object is now ready to upload/download data.')
})

file.on('done', function () {
  console.log('Upload/download is complete!')
})

file.on('error', function (error) {
  console.log('The file has encountered some sort of error.')
  console.log(error)
})
```

### 3. Download a file



### 4. Do something with it!



### Now what?

Keep scrolling down for a review of file operations, or jump straight into
[Storj in a Browser](04-browser.md). Backend engineers can skip ahead to [Advanced Bucketing](05-bucket-ops.md).

### Putting it all together
```javascript
// Instantiate a new Storj object with your private key
var storj = new Storj({key: privkey})

// Node:
var fs = require('fs')
var data = fs.readFileSync('/path/to/file/cat.jpg',)

// Browsers:
<input type="file" id="input">

<script>
var data = document.getElementById('input').files[0]
</script>

// Both:
var file = storj.createFile(bucketID, 'cat.jpg', data, function () {
  console.log('Upload complete')
})

// Log the file information
console.log("name:", file.name)
console.log("type:", file.mimetype)
console.log("size in bytes:", file.length)
console.log("upload/download progress:", file.progress)
console.log("uploaded/downloaded bytes:", file.progress * file.bytes)

// Register file event listeners
file.on('ready', function () {
  console.log('The file object is now ready to upload/download data.')
})
file.on('done', function () {
  console.log('Upload/download is complete!')
})
file.on('error', function (error) {
  console.log('The file has encountered some sort of error.')
  console.log(error)
})
```

Next up: [Storj in a Browser](04-browser.md)
If front-end isn't your thing, go forward two spaces to
[Advanced Bucketing](05-bucket-ops.md)
