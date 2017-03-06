Now I'm going to walk you through making a MVP image gallery with Storj.js.
Our MVP image gallery is going to display all images in a bucket (and ignore
other files).

1. An HTML page

We'll start with the most basic possible html page, add Storj.js, and then a
div to put our images in:

```html
<body>
  <script source="./storj.es6.js"></script>
  <div id="imagesGoHere"></div>
</body>
```

2. Set up a public bucket

Now let's make our backend. We'll do this in Node, so we don't need to write a
page for it.

We want a public bucket, so other people can view our images. Let's go ahead
and create one. Then we'll use `storj.makePublic()` to allow unauthenticated
clients to pull from it.

```javascript
// Node:
// Run this once, on a machine you trust!
// And don't forget to give it your key and mnemonic!
var Storj = require('storj')
var storj = new Storj({key:privkey, mnemonic:mnemonic})
var opts = {pull:true, push:false}

storj.createBucket('Image Gallery Tutorial', function (e, bucket) {
  if (e) {console.log(e)}

  storj.makePublic(meta.id, opts, function (e) {
    if (e) {console.log(e)}
  })
})
```

We also want to upload some photos, so that our image gallery isn't just a sad
blank div on a sad sad page. I have about 33 GB of cat photos stored locally,
so let's stick them in a bucket:

```javascript
// Node:
var fs = require('fs')
var catFolder = '~/cats/'
fs.readdir(catFolder, function (e, files) {
  files.forEach( function (file) {
    fs.readFile(file, function (e, data) {
      if (e) {console.log(e)}
      storj.createFile(bucketID, file, data)
    })
  })
})
```

This code reads a directory, and uploads all the files in it. A (much) better
version of this can be written using `fs.stat` to exclude directories, and the
`async` libraries for flow control. But we'll leave that as an excercise to the
reader.

3. Show the images
Let's make a little script to embed in the page. We want it to connect to a
bucket, and retrieve a list of files. We already set up a public bucket, so we
don't need to worry about authentication.

```javascript
var storj = new Storj()
var fileList = storj.getFileList(bucketID)

fileList.forEach(function (fileData) {
  if (fileData.mimetype.indexOf('image') !== -1) {
    var file = storj.getFile(bucketID, fileData.id)
    file.on("done", function () {
      file.appendTo('#imagesGoHere')
    })
  }
})
```

It gets a list of files in our bucket. Then, for each image in the bucket,
retrieves the file. The images append themselves to the `div` we set up earlier
as their downloads finish.

We're gonna stick this in our html page from earlier:

```html
<body>
  <script source="./storj.es6.js"></script>
  <script type="text/javascript">
    var storj = new Storj()
    var fileList = storj.getFileList(bucketID)

    fileList.forEach(function (fileData) {
      if (fileData.mimetype.indexOf('image') !== -1) {
        var file = storj.getFile(bucketID, fileData.id)
        file.on("done", function () {
          file.appendTo('#imagesGoHere')
        })
      }
    })    
  </script>
  <div id="imagesGoHere"></div>
</body>
```

Now the images will load for anyone visiting the page.

### Next up
See below for the entire image gallery app. We're going to keep updating our
image gallery to [add uploads in the browser](08-gallery-2.md).

### Putting it all together

```javascript
// Node:
// This is the bucket setup code, and the file upload code.
// Run this once, on a machine you trust.
// And don't forget to give it your key and mnemonic!
var Storj = require('storj')
var storj = new Storj({key:privkey, mnemonic:mnemonic})
var opts = {pull:true, push:false}

storj.createBucket('Image Gallery Tutorial', function (e, bucket) {
  if (e) {console.log(e)}

  storj.makePublic(meta.id, opts, function (e) {
    if (e) {console.log(e)}
  })
})

var fs = require('fs')
var catFolder = '~/cats/'
fs.readdir(catFolder, function (e, files) {
  files.forEach( function (file) {
    fs.readFile(file, function (e, data) {
      if (e) {console.log(e)}
      storj.createFile(bucketID, file, data)
    })
  })
})
```

```html
<!-- This is the browser page that displays the photos. -->
<body>
  <script source="./storj.es6.js"></script>
  <script type="text/javascript">
    var storj = new Storj()
    var fileList = storj.getFileList(bucketID)

    fileList.forEach(function (fileData) {
      if (fileData.mimetype.indexOf('image') !== -1) {
        var file = storj.getFile(bucketID, fileData.id)
        file.on("done", function () {
          file.appendTo('#imagesGoHere')
        })
      }
    })    
  </script>
  <div id="imagesGoHere"></div>
</body>
```

Now let's make some upgrades to our gallery:
[Gallery 2: Electric Boogaloo](08-gallery-2.md).
