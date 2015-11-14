# S3renity.js
A powerful S3 toolbelt that gives you access to batch operations like forEach, map, reduce, filter, as well as a promise-based wrapper around the S3 api.

## Use Cases
- Quickly prototype MapReduce jobs
- Clean or organize dirty log files
- Perform sync or async functions over each file with forEach, map, reduce, and filter
- Many more...

## Getting started
Install S3renity  
```bash
npm install s3renity --save
```

In your code:
```javascript
var S3renity = require('s3renity');

var s3renity = new S3renity({
  access_key_id: 'your access key',
  secret_access_key: 'your secret key'
});

var folder = 's3://<bucket>/path/to/folder/';

var fn = s3object => console.log('this is an s3 object!', s3object);

// map func over every line separated string
s3renity
  .encode('utf8')  // (optional) this is default
  .context(folder) // sets the directory in s3
  .split('\n')     // (optional) split each object by a delimiter
  .forEach(fn)     // function to perform over each s3 object (or line if split is used)
  .then(result => ...do stuff...)
  .catch(e)
```

## Instructions
S3renity has the concept of a working context, which defines the files or log entries you are working with.  The working context is set by ```s3renity.context()```.  By calling that on a valid s3 path, the working context is set to all the files with that key prefix (in that directory).  From there, you can perform batch operations.  For example:  
```javascript
s3renity
  .context(dir)
  .forEach(body => console.log('do something with file body')
  .then(s => ...do something else...)
  .catch(e => ...handle error...);
```

It is also possible for the working context to set to the lines in the files by calling ```split()```.  Suppose you wanted to iterate over every line in every file in a S3 directory.  You could do something like:  
```javascrpit
s3renity
  .context(dir)
  .split('\n')
  .forEach(line => console.log('do something with line'))
  .then(s => ...do something else...)
  .catch(e => ...handle error);
```

## Batch Functions
- forEach
- map
- reduce
- filter

**forEach(func[, isAsync])**  
Perform ```func``` on every item in the working context.  If ```split``` is called, ```func``` takes a string.  Otherwise, it takes an S3 object.  ```func``` either performs a synchronous action on the argument, or returns a promise.
```javascript
s3renity
  .context(path)
  .forEach(func)
  .then(..)
  .catch(..)
  
s3renity
  .context(path)
  .split('\n')
  .forEach(func)
  .then(_ => ...)
  .catch(e)
```

**map(func[, isAsync])**  
Perform ```func``` on every item in the working context, replacing each in place.  *This function is destructive*.  If ```split``` is called, ```func``` takes a string.  Otherwise, it takes an S3 object.  ```func``` is either synchronous that returns the new object (or item) or returns a promise that resolves to the new object.
```javascript
s3renity
  .context(path)
  .map(func)
  .then(_ => ...)
  .catch(e)
  
s3renity
  .context(path)
  .split('\n')
  .map(func)
  .then(_ => ...)
  .catch(e)
```

**reduce(func[, isAsync])**
Reduce the working context to a single value with ```func```. ```func``` takes three arguments: ```previousValue```, ```currentValue```, and ```key``` (the key of the current S3 object), and returns the updated object.

```javascript
s3renity
  .context(path)
  .reduce(func)
  .then(result => ...)
  .catch(e => ...)

s3renity
  .context(path)
  .split('\n')
  .reduce(func)
  .then(result => ...)
  .catch(e => ...)
```

**filter(func)**  
Filter the working context with ```func```, removing all objects or entries that don't pass the test.  *This function is destructive*.  If ```split``` is called, ```func``` takes a string.  Otherwise, it takes an S3 object.  ```func``` is either synchronous or returns a promise, and returns false if the item should be filtered.
```javascript
s3renity
  .context(path)
  .filter(func)
  .then(_ => ...)
  .catch(e)
  
s3renity
  .context(path)
  .split('\n')
  .filter(func)
  .then(_ => ...)
  .catch(e)
```

## Extra Functionality
- keys
- get
- put
- delete
- join
- clean
- write
- splitObject

**keys()**  
Get all the object keys in the working context.
```javascript
s3renity
  .context(path)
  .keys()
  .then(keys => ...)
  .catch(e => ...)
```

**get(arg1[, arg2])**  
Get an object from S3 either by specifying a valid key, or separate them into two arguments key and bucket.
```javascript
s3renity
  .get('bucket', 'path/to/object')
  .then(object => ...)
  .catch(e => ...)

s3renity
  .get('s3://path/to/object')
  .then(object => ...)
  .catch(e => ...)
```

**put(bucket, key, body)**  
Put an object in S3.
```javascript
s3renity
  .put(bucket, key, body)
  .then(_ => ...)
  .catch(e => ...)
```

**delete(bucket, key)**  
Delete an object or list of objects from S3.  Key can be a string key, the object to delete, or an array of keys to delete.
```javascript
s3renity.delete('bucket', 'path/to/file.txt')
s3renity.delete('bucket', ['s3://bucket/path/to/thing', 's3://bucket/path/to/thing2'])
```

**join(delimiter)**  
Concatenate, or "join" the objects in the working context into one file, and direct the output to an S3 path or a local file.  Output can also be an array of paths.  Under the hood, join deferrs a promise and returns ```this```, so you need to call ```output()``` after for it to run.
```javascript
s3renity
  .context(path)
  .join('\n')
  .output('output.txt')
  .then(_ => ...)
  .catch(e)

s3renity
  .context(path)
  .join('\n')
  .output(['output.txt', 's3://bucket/path/to/file1', 's3://bucket/path/to/file2'])
  .then()
```

**clean()**  
Removes empty files in the working context
```javascript
s3renity
  .context(path)
  .clean()
  .then(_ => ...)
  .catch(e => ...)
```

**write(body, targets)**  
Output the working context to a file or location in s3. Targets can be a string (single target) or an array of targets.  Targets can be local files or valid S3 paths.
```javascript
var text = 'blah';
s3renity.write(text, ['s3://bucket/path/to/file.txt', 'localfile.txt']);
```

**splitObject(bucket, key[, delimiter, encoding])**  
Split a single (text) object in S3 by a delimiter. Default delimiter is \n and encoding is utf8.
```javascript
s3renity
  .context(path)
  .splitObject('bucket', 'path/to/object', '\t');
  .then(result => ...)
  .catch(e => ...)
```

## Advanced Examples
Perform an asynchronous db lookup for every line of JSON in every file in the working context.
```javascript
const lookupId = line => {
  line = JSON.parse(line);
  return new Promise((success, fail) => {
    db.lookup(line.id).then(success).catch(fail);
  });
};

s3renity
  .context(path)
  .split('\n')
  .forEach(lookupId)
  .then(..)
  .catch(..)
```

Add a field to each log entry and then concatonate all the files into one and save it locally.
```javascript
s3renity
  .context(path)
  .split('\n')
  .map(entry => {
    var temp = JSON.parse(entry);
    if (temp.timestamp == null) {
     temp.timestamp = Date.now()/1000|0;
    }
    return temp; 
  })
  .then(_ => s3renity.join('\n').then(b => write('output.txt'))
  .catch(err => console.log(err));
```

## More Examples
Check out the examples file https://github.com/littlstar/s3renity/blob/master/examples.js
