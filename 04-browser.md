The `File` object also has some cool browser-only tools for DOM manipulation.
After all, if you download a file from a distributed network in a browser, and
it never gets rendered to the page, did it even happen?

### 1. The Blob (1958)

The W3C defines a blob as a 'binary large object,' but I prefer to think of it
as an aggressive hunk of jelly. Unfortunately, I'm not in charge of modern
browser standards, so we'll stick with their definition. For now.

Within a browser we can access file data as a W3C `Blob` using
`file.getBlob()`, or via a Blob URL using `file.getBlobURL()`. Anyone who has
used [WebTorrent](https://github.com/feross/webtorrent) might think this sounds
awfully familiar. That is not an accident.

```javascript
download.getBlob(function (error, blob) {
  if (error) {
    console.log(error)
  }
  doTheThingWithThe(blob)
})
```

We can also get a Blob URL:

```javascript
download.getBlobURL(function (err, url) {
  console.log(url)
})
```

Blob URLs are particularly useful for making download links, or adding files to
the page:

```javascript
download.getBlobURL(function (error, url) {
  if (error) throw error
  var a = document.createElement('a')
  a.download = file.name
  a.href = url
  a.textContent = 'Download ' + file.name
  document.body.appendChild(a)
})

download.getBlobURL(function (error, url) {
  if (error) throw error
  var img = document.createElement('img')
  img.src = url
  document.body.appendChild(img)
})
```

Adding an `<img>` to the page via a Blob URL is not exactly elegant. Fortunately,
the `File` object has better ways to get file data into the DOM.

### 2. Rendering and appending (aka Appendering)

Rather than building an element from a Blob URL, we thought it'd be really nice
if something just did that for you. So that's what `file.appendTo()` and
`file.renderTo()` do. They take a container (either a CSS selector or an object
reference), and intelligently add the file to it based on the file's media
type.

Take this stupidly simple page:

```html
<body>
  <div id="vidGoesHere"></div>
  <img src="" id="imageGoesHere"></img>
</body>
```

We can just drop files into it no problem:

```javascript
var catVid = storj.getFile(bucketID, catVidFileID)
catVid.appendTo('#vidGoesHere')

var catPic = storj.getFile(bucketID, catPicFileID)
catPic.renderTo('#imageGoesHere')
```

The `File` object figures out the appropriate elements based on file type,
builds them, and adds them to the page. Video and audio get streaming players.
Images are displayed. PDFs get an iframe, etc.

Appending and rendering have slightly different behavior. Where
`file.appendTo()` adds the new elements to the DOM, as the last child of its
parent, `file.renderTo()` replaces the entire contents of the DOM node with the
new element. Appending and rendering have the same interface, so the rest of
the examples will focus on `file.appendTo()`.

### 3. Advanced Appendering

There are a couple optional arguments for `file.appendTo()` and
`file.renderTo()`: an options object, and a callback. The options argument
is an object that includes some options related to video and audio streaming,
just in case we want to specify those beforehand. The defaults are below. The
callback provides access to the element after it's rendered.

```javascript
var defaultOpts = {
  autoplay: true,                  // Autoplay the media.
  controls: true,                  // Show media playter controls.
  maxBlobLength: 200 * 1000 * 1000 // Run away from Blobs over 200 megabytes.
}

catVid.appendTo('#vidGoesHere', defaultOpts, function (error, elem) {
  if (error) {
    console.log(error)
  }
  console.log('Video player created.')
})
```

The callback is really useful for modifying the page when the file renders (or
fails to!). Depending on your page structure, it could look something like
this:

```javascript
document.getElementById('loading-animation').style.display = 'inline-block'
catVid.appendTo('#vidGoesHere', function cb(e) {
  if(e) {
    document.getElementById('status').innerText = 'Something very bad happened'
  }
  document.getElementById('loading-animation').style.display = 'none'
})
```

### What's next?

For more reading on how appendering works under the hood, check out the
`render-media` package [here](https://www.npmjs.com/package/render-media).

Appendering is a powerful tool. Let's let it soak in for a minute while we
explore the wondeful world of [Bucket Operations](05-bucket-ops.md).


### Putting it all together

```javascript
// Access file data as a W3C Blob
download.getBlob(function (error, blob) {
  if (error) {
    console.log(error)
  }
  doTheThingWithThe(blob)
})

// Access file data via a Blob URL
download.getBlobURL(function (err, url) {
  console.log(url)
})

// Build DOM elements from Blob URLs
download.getBlobURL(function (error, url) {
  if (error) throw error
  var a = document.createElement('a')
  a.download = file.name
  a.href = url
  a.textContent = 'Download ' + file.name
  document.body.appendChild(a)
})

download.getBlobURL(function (error, url) {
  if (error) throw error
  var img = document.createElement('img')
  img.src = url
  document.body.appendChild(img)
})

// A stupidly simple HTML document
<body>
  <div id="vidGoesHere"></div>
  <img src="" id="imageGoesHere"></img>
</body>

// Intelligently append file to a DOM node
var catVid = storj.getFile(bucketID, catVidFileID)
catVid.appendTo('#vidGoesHere')

// Replace the contents of a DOM node with the file
var catPic = storj.getFile(bucketID, catPicFileID)
catPic.renderTo('#imageGoesHere')


// Options for appendering
var defaultOpts = {
  autoplay: true,                  // Autoplay the media.
  controls: true,                  // Show media playter controls.
  maxBlobLength: 200 * 1000 * 1000 // Run away from Blobs over 200 megabytes.
}

// Appender with options and a callback
catVid.appendTo('#vidGoesHere', defaultOpts, function (error, elem) {
  if (error) {
    console.log(error)
  }
  console.log('Video player created.')
})

// Appender callbacks can (and often should!) modify the DOM.
// I didn't include a sample HTML document for this.
// Sorry.
document.getElementById('loading-animation').style.display = 'inline-block'
catVid.appendTo('#vidGoesHere', function cb(e) {
  if(e) {
    document.getElementById('status').innerText = 'Something very bad happened'
  }
  document.getElementById('loading-animation').style.display = 'none'
})
```

Now that we know what our files can do, it's time to circle back and take a
deeper look at [Bucket Operations](05-bucket-ops.md).

Or, take a quick break to read about the super-cool `render-media` package
[here](https://www.npmjs.com/package/render-media). Feross-senpai notice me
please.
