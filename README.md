# jhttp-client

node.js promise-based http client

## features
- HTTP & HTTPS
- GET, POST, PUT, DELETE, HEAD, PATCH
- built-in [cookie manager](https://www.npmjs.org/package/cookie-manager)
- return string as response `body`
- return jQuery like object as response `body` ([cheerio](https://www.npmjs.org/package/cheerio))
- return json object as response `body`
- return buffer as response `body`
- support file upload or other arbitrary HTTP data transfer

## install
```javascript
npm install jhttp-client
```

## usage
```javascript

var HTTPC = require("jhttp-client");
var httpClient = HTTPC();

httpClient.request("google.com")
.then(function( response ){

  console.log(response.request.url);
  console.log(response.status);
  console.log(response.headers);
  console.log(response.body);    

})
.fail(function( response ){
  
  console.log(response.request.url);
  console.log(response.status);
  console.log(response.headers);
  console.log(response.body);
  console.log(response.text);

});

```

---

## Options
```javascript
// global options
var httpClient = HTTPC( options );

// per-request options
// will extend global options
httpClient.request( options );
```

### `options` [object]
```javascript
{
  auth               : '',
  accept             : 'text/html, text/plain, application/json, */*',
  output             : 'string',
  expect             : 200,
  followRedirect     : true,
  useCookie          : true,
  method             : "get",
  url                : "example.com",
  data               : false,
  charset            : "utf-8",
  proxy              : '',
  ssl                : {
    rejectUnauthorized : false,
  },
  headers            : {
    'user-agent'       : '',
    'Accept-Encoding'  : 'gzip;q=0.9, deflate;q=0.5, identity;q=0.2'
  },
  agent          : {
    keepAlive          : true
  },
  timeout      : 60000,
  log          : false,
  stream       : false
}

// above are the default values
```

**options.method** "get", "post", "put", "delete", "head", "patch".

**options.url** "http://domain.com/path" or "https://domain.com" or "domain.com" or "http://unix:/the/socket:/uri/path?par=stuff".

**options.accept** will be used in `Accept` headers.

- `options.accept = "*/*"` will be overriden by <br />`options.output = "json"` or `options.output = "$"`.

**options.output** "string", "buffer", "json", "$". The `$` will output jQuery-like object.

- `options.output = "json"` will change `options.accept` from `*/*` to `application/json`.

- `options.output = "$"` will change `options.accept` from `*/*` to `text/html`.

**options.expect** HTTP status to expect. Will *reject* the *promise* if not fulfilled.

- Use `options.expect = false` to accept every HTTP status

**options.charset** will be used in `Accept` and `Accept-Charset` headers.

**options.followRedirect**. If `true`, will not *reject* the *promise* on redirect status (3**).

**options.useCookie** will save cookie and use them for future requests with the same domain.

- `options.useCookie` Will be overriden by `options.headers.cookie` value.

**options.auth** basic HTTP auth: "user:password".

**options.proxy** To make request through a proxy, i.e. http://123.345.543.234:8080

**options.stream** Use stream for request body (boolean).

**options.timeout** Timeout used in client.

**options.ssl** is used in `tls.connect()`. Read more in the [documentation](http://nodejs.org/api/https.html#https_https_request_options_callback).

**options.headers.accept** will override **options.accept** regardless of **options.output**.

**options.headers["accept-charset"]** will override **options.charset**.

- `accept-encoding` headers will always be `gzip;q=0.9, deflate;q=0.5, identity;q=0.2`, unless overriden with `options.headers["accept-encoding"]`.

---

## Stream
Use **options.stream** to use stream in handling response body.
```js
var req = httpClient.request({
  url: 'http://example.com',
  stream: true
});

req.stream
.on('redirect',function(url){
  
  

}).on('headers',function(heads){
  
  console.log(heads);

}).on('data',function(c){

    process.stdout.write(c);

}).on('end',function(){

    console.log('--------------------------');
    console.log('All data has been printed');

});

// You can still use promise for 
req.then(function( res ){
  
  // res.body will be empty
  console.log( res.body );

})

```

## Data Transfer

Use `options.data` object to transfer data. Below are some example of data transfers.

**NOTE:** `GET` request will not use the `options.data` attribute.

#### HTML form upload
The following will create an `application/x-www-form-urlencoded` data transfer:
```javascript
{
  content:{
    name: 'JohnDoe',
    occupation: 'Awesome Staff'
  }
}
```

Will create:
```text
POST /path HTTP/1.1
Host: domain.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 39

name=JohnDoe&occupation=Awesome%20Staff
```

A `multipart/form-data` data transfer can also be created:
```javascript
{
  content: {
      name: 'JohnDoe',
      occupation: 'Awesome Staff'
  },
  file:[
    { 
      // will read the file and send it
      name:'uploaded1',
      filename: 'some/file.txt' 
    },
    {
      // construct your own
      name:'uploadedFile',
      filename: 'file2.jpg',
      mime: 'image/jpg',
      content: 'jpeg strings.....'
    },
    {
      // use automatic MIME based on `filename` extension
      name:'uploadedFile',
      filename: 'file3.jpg',
      content: 'jpeg strings.....'
    }
  ]
}
```

Will create:
```text
POST /path HTTP/1.1
Host: domain.com
Content-Type: multipart/form-data; boundary=o98aywaw74eyo
Content-Length: 2234766

--o98aywaw74eyo
Content-Disposition: form-data; name="name"

JohnDoe
--o98aywaw74eyo
Content-Disposition: form-data; name="occupation"

Awesome Staff
--o98aywaw74eyo
Content-Disposition: form-data; name="uploaded1"; filename="file.txt"
Content-Type: text/plain

text here
--o98aywaw74eyo
Content-Disposition: form-data; name="uploadedFile"; filename="file2.jpg"
Content-Type: image/jpg

file2.jpg content here ...
--o98aywaw74eyo
Content-Disposition: form-data; name="uploadedFile"; filename="file3.jpg"
Content-Type: image/jpg

file3.jpg content here ...
--o98aywaw74eyo--
```


#### Arbitrary Data Transfer
You can construct everything yourself:
```javascript
{
  type : 'application/xml',
  content: '<some>XML</some>'
}
```
Or, using stream:
```javascript
{
  type : 'text/csv',
  filename: '/home/some/path/file.csv'
}
```
Remember that using `options.data.type` will override `Content-Type` header.

Use it only to send strings with `options.data.content` or `options.data.filename`, unless you know what you're doing..



cheers,

[jujiyangasli.com](http://jujiyangasli.com)
