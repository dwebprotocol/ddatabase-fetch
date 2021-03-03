# @ddatabase/fetch

Implementation of Fetch that uses the dWeb SDK for loading p2p content

`npm install --save @ddatabase/fetch`

```javascript
const fetch = require('@ddatabase/fetch')()

const someURL = `dweb://peepsx.com`

const response = await fetch(`${someURL}/index.json`)

const json = await response.json()

console.log(json)
```

You can also use the bundled CLI

```
npm i -g @ddatabase/fetch

dfetch dweb://whatever

# Or

npm dfetch dweb://somethingorother
```

## API

### `makeFetch({DDrive, resolveName, base, session, writable}) => fetch()`

Creates a @ddatabase/fetch instance.

The `base` parameter can be used to specify what the base URL is for relative paths like `fetch('./dwebx.json')`.

You can pass in options for the [dWeb SDK](https://github.com/dwebprotocol/dweb-sdk) to have it be auto-created,
or you can pass in both a function matching  `const archive = DDrive(key)` and a `const resolved = await resolveName(url)` function.

Set `session` to your Electron session if you want to enable setting the `body` of fetch requests to Electron's [UploadData](https://www.electronjs.org/docs/api/structures/upload-data) API in their protocol handlers.

If you don't want to allow write access to archives, pass in `writable: false`.

Typically, you don't need to pass in any of these and they're there for more advanced users.

After you've created it, `fetch` will behave like it does in [browsers](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

### `await fetch.close()`

Closes resources for the Dat SDK. This does nothing if you specified the DDrive and `resolveName` options.

### Common Headers

Each response will contain a header for the canonical URL represented as a `Link` header with `rel=canonical`.

Each response will also contain the `Allow` header of all the methods currently allowed. If the archive is writable, this will contain `PUT`.

There is also an `ETag` header which will be a JSON string containging the drive's current `version`. This will change only when the drive has gotten an update of some sort and is monotonically incrementing.

### `fetch('dweb://NAME/example.txt', {method: 'GET'})`

This will attempt to load `example.txt` from the archive labeled by `NAME`.

It will also load `index.html` files automatically for a folder.
You can find the details about how resolution works in the [resolve-dweb-path](https://github.com/dwebprotocol/resolve-dat-path)) module.

`NAME` can either be the 64 character hex key for an archive, a domain to parse with [dweb-dns](https://github.com/dwebprotocol/dweb-dns), or a name for an archive which allows you to write to it.

The response headers will contain `X-Blocks` for the number of blocks of data this file represents on disk, and `X-Blocks-Downloaded` which is the number of blocks from this file that have been downloaded locally.

### `fetch('dweb://NAME/.well-known/dwebx', {method: 'GET'})`

This is used by the dweb-dns module for resoving dns domains to `dwebx://` URLs.

This will return some text which will have a `dwebx://` URL of your archive, followed by a newline and a TTL for the DNS record.

### `fetch('dweb://NAME/example/', {method: 'GET'})`

When doing a `GET` on a directory, you will get a directory listing.

By default it will render out an HTML page with links to files within that directory.

You can set the `Accept` header to `application/json` in order to have it return a JSON array with file names.

e.g.

```json
["example.txt", "posts/", "example2.md"]
```

Files in the directory will be listed under their name, sub-directories will have a `/` appended to them.

`NAME` can either be the 64 character hex key for an archive, a domain to parse with [dweb-dns](https://github.com/dwebprotocol/dweb-dns), or a name for an archive which allows you to write to it.

### `fetch('dweb://NAME/example.txt', {method: 'GET', headers: {'X-Resolve': 'none'}})`

Setting the `X-Resolve` header to `none` will prevent resolving `index.html` files and will attempt to load the path as is.
This can be useful for list files in a directory that would normally render as a page.

You should omit the header for the default behavior, different values may be supported in the future.

`NAME` can either be the 64 character hex key for an archive, a domain to parse with [dweb-dns](https://github.com/dwebprotocol/dweb-dns), or a name for an archive which allows you to write to it.

The response headers will contain `X-Blocks` for the number of blocks of data this file represents on disk, and `X-Blocks-Downloaded` which is the number of blocks from this file that have been downloaded locally.

### `fetch('dweb://NAME/example.txt', {method: 'PUT', body: 'Hello World'})`

You can add files to archives using a `PUT` method along with a `body`.

The `body` can be either a `String`, an `ArrayBuffer`, a `Blob`, a WHATWG `ReadableStream`, a Node.js `Stream`, or electron's [UploadData](https://www.electronjs.org/docs/api/structures/upload-data) object (make sure to specify the `session` argument in the `makeFetch` function for electron support).

`NAME` can either be the 64 character hex key for an archive, a domain to parse with [dweb-dns](https://github.com/dwebprotocol/dweb-dns), or a name for an archive which allows you to write to it.

Your `NAME` will likely be a `name` in most cases to ensure you have a writeable archive.

### `fetch('dweb://NAME/example.txt', {method: 'DELETE'})`

You can delete a file in an archive by using the `DELETE` method.

You cannot delete directories if they are not empty.

`NAME` can either be the 64 character hex key for an archive, a domain to parse with [dweb-dns](https://github.com/dwebprotocol/dweb-dns), or a name for an archive which allows you to write to it.

### `fetch('dweb://NAME/example.txt', {method: 'DOWNLOAD'})`

You can download a file or an entire folder using the `DOWNLOAD` method.

`NAME` can either be the 64 character hex key for an archive, a domain to parse with [dweb-dns](https://github.com/dwebprotocol/dweb-dns), or a name for an archive which allows you to write to it.

You can use `/` for the path to download the entire contents

### `fetch('dweb://NAME/example.txt', {method: 'CLEAR'})`

You can clear the data stored for a file using the `CLEAR` method.

This is like the opposite of the `DOWNLOAD` method.

This does not delete data, it only deletes the cached data from disk.

`NAME` can either be the 64 character hex key for an archive, a domain to parse with [dweb-dns](https://github.com/dwebprotocol/dweb-dns), or a name for an archive which allows you to write to it.

You can use `/` for the path to clear all data for the archive.

### `fetch('dweb://NAME/`, {method: 'TAG', body: 'Tag name here'})`

You can add a tag a version of the archive with a human readable name (like SPAGHETTI).

You can place the name of the tag into the `body` of the request.

Afterwards you can load the archive at that given version with `dweb://NAME+TAG_NAME`. E.g. `dweb://123kjh213kjh123+v4.20/example.txt`

### `fetch('dweb://NAME/', {method: 'TAGS'})`

You can get a list of all tags using the `TAGS` method.

The response will be a JSON object which maps tag names to archive versions.

Use `await response.json()` to get the data out.

### `fetch('dweb://NAME+TAG_NAME/', {method: 'TAG-DELETE'})`

You can delete a given tag with the `TAG-DELETE` method.

Specify the tag you want in the URL, and it'll be removed from the tags list.
