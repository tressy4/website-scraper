## Introduction
Download website to a local directory (including all css, images, js, etc.)

[![Build Status](https://travis-ci.org/website-scraper/node-website-scraper.svg?branch=master)](https://travis-ci.org/website-scraper/node-website-scraper)
[![Build status](https://ci.appveyor.com/api/projects/status/s7jxui1ngxlbgiav/branch/master?svg=true)](https://ci.appveyor.com/project/s0ph1e/node-website-scraper/branch/master)
[![Test Coverage](https://codeclimate.com/github/website-scraper/node-website-scraper/badges/coverage.svg)](https://codeclimate.com/github/website-scraper/node-website-scraper/coverage)
[![Code Climate](https://codeclimate.com/github/website-scraper/node-website-scraper/badges/gpa.svg)](https://codeclimate.com/github/website-scraper/node-website-scraper)
[![Dependency Status](https://david-dm.org/website-scraper/node-website-scraper.svg?style=flat)](https://david-dm.org/website-scraper/node-website-scraper)

[![Version](https://img.shields.io/npm/v/website-scraper.svg?style=flat)](https://www.npmjs.org/package/website-scraper)
[![Downloads](https://img.shields.io/npm/dm/website-scraper.svg?style=flat)](https://www.npmjs.org/package/website-scraper)
[![Gitter](https://badges.gitter.im/website-scraper/node-website-scraper.svg)](https://gitter.im/website-scraper/node-website-scraper?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

[![NPM Stats](https://nodei.co/npm/website-scraper.png?downloadRank=true&stars=true)](https://www.npmjs.org/package/website-scraper)

You can try it in [demo app](https://scraper.nepochataya.pp.ua/) ([source](https://github.com/website-scraper/web-scraper))

**Note:** by default dynamic websites (where content is loaded by js) may be saved not correctly because `website-scraper` doesn't execute js, it only parses http responses for html and css files. If you need to download dynamic website take a look on [website-scraper-phantom](https://github.com/website-scraper/node-website-scraper-phantom).


## Installation
```
npm install website-scraper
```

## Usage
```javascript
const scrape = require('website-scraper');
const options = {
  urls: ['http://nodejs.org/'],
  directory: '/path/to/save/',
};

// with promise
scrape(options).then((result) => {
	/* some code here */
}).catch((err) => {
	/* some code here */
});

// with async/await
const result = await scrape(options);

// or with callback
scrape(options, (error, result) => {
	/* some code here */
});
```

## options
* [urls](#urls) - urls to download, *required*
* [directory](#directory) - path to save files, *required*
* [sources](#sources) - selects which resources should be downloaded
* [recursive](#recursive) - follow hyperlinks in html files
* [maxRecursiveDepth](#maxrecursivedepth) - maximum depth for hyperlinks
* [maxDepth](#maxdepth) - maximum depth for all dependencies
* [request](#request) - custom options for for [request](https://github.com/request/request)
* [subdirectories](#subdirectories) - subdirectories for file extensions
* [defaultFilename](#defaultfilename) - filename for index page
* [prettifyUrls](#prettifyurls) - prettify urls
* [ignoreErrors](#ignoreerrors) - whether to ignore errors on resource downloading
* [urlFilter](#urlfilter) - skip some urls
* [filenameGenerator](#filenamegenerator) - generate filename for downloaded resource
* [updateMissingSources](#updatemissingsources) - update url for missing sources with absolute url
* [requestConcurrency](#requestconcurrency) - set maximum concurrent requests
* [plugins](#plugins) - plugins, allow to customize filenames, request options, response handling, saving to storage, etc.
 
Default options you can find in [lib/config/defaults.js](https://github.com/website-scraper/node-website-scraper/blob/master/lib/config/defaults.js) or get them using `scrape.defaults`.

#### urls
Array of objects which contain urls to download and filenames for them. **_Required_**.
```javascript
scrape({
  urls: [
    'http://nodejs.org/',	// Will be saved with default filename 'index.html'
    {url: 'http://nodejs.org/about', filename: 'about.html'},
    {url: 'http://blog.nodejs.org/', filename: 'blog.html'}
  ],
  directory: '/path/to/save'
});
```

#### directory
String, absolute path to directory where downloaded files will be saved. Directory should not exist. It will be created by scraper. **_Required_**.

#### sources
Array of objects to download, specifies selectors and attribute values to select files for downloading. By default scraper tries to download all possible resources.
```javascript
// Downloading images, css files and scripts
scrape({
  urls: ['http://nodejs.org/'],
  directory: '/path/to/save',
  sources: [
    {selector: 'img', attr: 'src'},
    {selector: 'link[rel="stylesheet"]', attr: 'href'},
    {selector: 'script', attr: 'src'}
  ]
});
```

#### recursive
Boolean, if `true` scraper will follow hyperlinks in html files. Don't forget to set `maxRecursiveDepth` to avoid infinite downloading. Defaults to `false`.

#### maxRecursiveDepth
Positive number, maximum allowed depth for hyperlinks. Other dependencies will be saved regardless of their depth. Defaults to `null` - no maximum recursive depth set. 

#### maxDepth
Positive number, maximum allowed depth for all dependencies. Defaults to `null` - no maximum depth set. 

#### request
Object, custom options for [request](https://github.com/request/request#requestoptions-callback). Allows to set cookies, userAgent, encoding, etc.
```javascript
// use same request options for all resources
scrape({
  urls: ['http://example.com/'],
  directory: '/path/to/save',
  request: {
    headers: {
      'User-Agent': 'Mozilla/5.0 (Linux; Android 4.2.1; en-us; Nexus 4 Build/JOP40D) AppleWebKit/535.19 (KHTML, like Gecko) Chrome/18.0.1025.166 Mobile Safari/535.19'
    }
  }
});
```

#### subdirectories
Array of objects, specifies subdirectories for file extensions. If `null` all files will be saved to `directory`.
```javascript
/* Separate files into directories:
  - `img` for .jpg, .png, .svg (full path `/path/to/save/img`)
  - `js` for .js (full path `/path/to/save/js`)
  - `css` for .css (full path `/path/to/save/css`)
*/
scrape({
  urls: ['http://example.com'],
  directory: '/path/to/save',
  subdirectories: [
    {directory: 'img', extensions: ['.jpg', '.png', '.svg']},
    {directory: 'js', extensions: ['.js']},
    {directory: 'css', extensions: ['.css']}
  ]
});
```

#### defaultFilename
String, filename for index page. Defaults to `index.html`.

#### prettifyUrls
Boolean, whether urls should be 'prettified', by having the `defaultFilename` removed. Defaults to `false`.

#### ignoreErrors
Boolean, if `true` scraper will continue downloading resources after error occurred, if `false` - scraper will finish process and return error. Defaults to `true`.

#### urlFilter
Function which is called for each url to check whether it should be scraped. Defaults to `null` - no url filter will be applied.
```javascript
// Links to other websites are filtered out by the urlFilter
scrape({
  urls: ['http://example.com/'],
  urlFilter: function(url){
    return url.indexOf('http://example.com') === 0;
  },
  directory: '/path/to/save'
});
```

#### filenameGenerator
String (name of the bundled filenameGenerator). Filename generator determines path in file system where the resource will be saved.

###### byType (default)
When the `byType` filenameGenerator is used the downloaded files are saved by extension (as defined by the `subdirectories` setting) or directly in the `directory` folder, if no subdirectory is specified for the specific extension.

###### bySiteStructure
When the `bySiteStructure` filenameGenerator is used the downloaded files are saved in `directory` using same structure as on the website:
- `/` => `DIRECTORY/example.com/index.html`
- `/about` => `DIRECTORY/example.com/about/index.html`
- `//cdn.example.com/resources/jquery.min.js` => `DIRECTORY/cdn.example.com/resources/jquery.min.js`

```javascript
scrape({
  urls: ['http://example.com/'],
  urlFilter: (url) => url.startsWith('http://example.com'), // Filter links to other websites
  recursive: true,
  maxRecursiveDepth: 10,
  filenameGenerator: 'bySiteStructure',
  directory: '/path/to/save'
});
```

#### updateMissingSources
Boolean, if `true` scraper will set absolute urls for all failing `sources`, if `false` - it will leave them as is (which may cause incorrectly displayed page).
Also can contain array of `sources` to update (structure is similar to [sources](#sources)).
Defaults to `false`.
```javascript
// update all failing img srcs with absolute url
scrape({
  urls: ['http://example.com/'],
  directory: '/path/to/save',
  sources: [{selector: 'img', attr: 'src'}],
  updateMissingSources: true
});

// download nothing, just update all img srcs with absolute urls
scrape({
  urls: ['http://example.com/'],
  directory: '/path/to/save',
  sources: [],
  updateMissingSources: [{selector: 'img', attr: 'src'}] 
});

```

#### requestConcurrency
Number, maximum amount of concurrent requests. Defaults to `Infinity`.


#### plugins
Object with `.apply` method. 
Can be used to configure or change scraper behavior. `.apply` method takes one argument - `registerAction` function which allows to customize saving resource or generating filename or handling http response etc.
You can add multiple plugins which register multiple actions. Plugins will be applied in order they were added to options.
All actions should be regular or async functions. Scraper will call actions of specific type in order they were added and use result (if supported by action type) from last action call.

List of supported actions with detailed descriptions and examples you can find below.
```javascript
class MyPlugin {
	apply(registerAction) {
		registerAction('beforeStart', async ({options}) => {});
		registerAction('afterFinish', async () => {});
		registerAction('error', async ({error}) => {console.error(error)});
		registerAction('beforeRequest', async ({resource, requestOptions}) => ({requestOptions}));
		registerAction('afterResponse', async ({response}) => response.body);
		registerAction('onResourceSaved', ({resource}) => {});
		registerAction('onResourceError', ({resource, error}) => {});
		registerAction('saveResource', async ({resource}) => {});
		registerAction('generateFilename', async ({resource}) => {})
	}
}

scrape({
  urls: ['http://example.com/'],
  directory: '/path/to/save',
  plugins: [ new MyPlugin() ]
});
```

##### beforeStart
Action `beforeStart` is called before downloading is started. It can be used to initialize something needed for other actions.
Parameters: 
* options - scraper options, normalized options object passed to scrape function
```javascript
registerAction('beforeStart', async ({options}) => {});
```

##### afterFinish
Action afterFinish is called after all resources downloaded or error occurred. Good place to shut down/close something initialized and used in other actions.
No parameters.
```javascript
registerAction('afterFinish', async () => {});
```

##### error
Action error is called when error occurred.
Parameters: 
* error - Error object
```javascript
registerAction('error', async ({error}) => {console.log(error)});
```

##### beforeRequest
Action beforeRequest is called before requesting resource. You can use it to customize request options per resource, for example if you want to use different encodings for different resource types or add something to querystring.
Parameters:
* resource - [Resource](https://github.com/website-scraper/node-website-scraper/blob/master/lib/resource.js) object
* requestOptions - default options for [request](https://github.com/request/request#requestoptions-callback) module or options returned by previous beforeRequest action call

Should return custom options for [request](https://github.com/request/request#requestoptions-callback) module.
If multiple actions `beforeRequest` added - scraper will use `requestOptions` from last one.
```javascript
// Add ?myParam=123 to querystring for resource with url 'http://example.com'
registerAction('beforeRequest', ({resource, requestOptions}) => {
	if (resource.getUrl() === 'http://example.com') {
		return extend(requestOptions, {qs: {myParam: 123}});
	}
	return {requestOptions};
});
```

##### afterResponse
Action afterResponse is called after each response, allows to customize resource or reject its saving.
Parameters:
* response - response object of [request](https://github.com/request/request) module

Should return resolved `Promise` if resource should be saved or rejected with Error `Promise` if it should be skipped.
Promise should be resolved with:
* `string` which contains response body
* or object with properties `body` (response body, string) and `metadata` - everything you want to save for this resource (like headers, original text, timestamps, etc.), scraper will not use this field at all, it is only for result.

If multiple actions `afterResponse` added - scraper will use result from last one.
```javascript
// Do not save resources which responded with 404 not found status code
registerAction('afterResponse', ({response}) => {
	if (response.statusCode === 404) {
			return Promise.reject(new Error('status is 404'));
		} else {
			// if you don't need metadata - you can just return Promise.resolve(response.body)
			return Promise.resolve({
				body: response.body,
				metadata: {
					headers: response.headers,
					someOtherData: [ 1, 2, 3 ]
			}
		});
	}
});
```

##### onResourceSaved
Action onResourceSaved is called each time after resource is saved (to file system or other storage with 'saveResource' action). 
Parameters:
* resource - [Resource](https://github.com/website-scraper/node-website-scraper/blob/master/lib/resource.js) object

Scraper ignores result returned from this action and does not wait until it is resolved
```javascript
registerAction('onResourceSaved', ({resource}) => console.log(`Resource ${resource.url} saved!`));
```

##### onResourceError
Action onResourceError is called each time when resource's downloading/handling/saving to was failed
Parameters:
* resource - [Resource](https://github.com/website-scraper/node-website-scraper/blob/master/lib/resource.js) object
* error - Error object

Scraper ignores result returned from this action and does not wait until it is resolved
```javascript
registerAction('onResourceError', ({resource, error}) => console.log(`Resource ${resource.url} has error ${error}`));
```

##### generateFilename
Action generateFilename is called to determine path in file system where the resource will be saved. 
Parameters:
* resource - [Resource](https://github.com/website-scraper/node-website-scraper/blob/master/lib/resource.js) object

Should return:
* filename - String, relative to `directory` path for specified resource

If multiple actions `generateFilename` added - scraper will use result from last one.

Default plugins which generate filenames: [byType](https://github.com/website-scraper/node-website-scraper/blob/master/lib/plugins/generate-filenamy-by-type-plugin.js), [bySiteStructure](https://github.com/website-scraper/node-website-scraper/blob/master/lib/plugins/generate-filenamy-by-site-structure-plugin.js)
```javascript
// Generate random filename 
const crypto = require('crypto');
registerAction('generateFilename', ({resource}) => {
  return {filename: crypto.randomBytes(20).toString('hex')};
});
```

##### saveResource
Action saveResource is called to save file to some storage. Use it to save files where you need: to dropbox, amazon S3, existing directory, etc. By default all files are saved in local file system to new directory passed in `directory` option (see [SaveResourceToFileSystemPlugin](https://github.com/website-scraper/node-website-scraper/blob/master/lib/resource-saver/index.js)).
Parameters:
* resource - [Resource](https://github.com/website-scraper/node-website-scraper/blob/master/lib/resource.js) object

If multiple actions `saveResource` added - resource will be saved to multiple storages.
```javascript
registerAction('saveResource', async ({resource}) => {
  const filename = resource.getFilename();
  const text = resource.getText();
  await saveItSomewhere(filename, text);
});
```

## result
Array of [Resource](https://github.com/website-scraper/node-website-scraper/blob/master/lib/resource.js) objects containing:
- `url`: url of loaded page
- `filename`: filename where page was saved (relative to `directory`)
- `children`: array of children Resources

## Log and debug
This module uses [debug](https://github.com/visionmedia/debug) to log events. To enable logs you should use environment variable `DEBUG`.
Next command will log everything from website-scraper
```bash
export DEBUG=website-scraper*; node app.js
```

Module has different loggers for levels: `website-scraper:error`, `website-scraper:warn`, `website-scraper:info`, `website-scraper:debug`, `website-scraper:log`. Please read [debug](https://github.com/visionmedia/debug) documentation to find how to include/exclude specific loggers.
