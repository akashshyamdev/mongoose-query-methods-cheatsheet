# Cheatsheet for Mongoose Query Methods
A cheatsheet of 100+ methods you can use to implement common patterns in mongoose data-fetching(eg pagination).

### Base Example
We'll work with a couple of queries which have not been resolved.
Since each method returns the overall instance, we can chain methods

```javascript
const todosQuery = Todo.find({});
const userQuery = User.find({});
```

###Filtering By Value
Using the `where()` method in conjugation with some other methods, based on value we can filter out certain results.

The following methods are usually used with the `where()` method:

* `equals()`
* `gt()`
* `lt()`
* `gte()`
* `lte()`

```javascript
// Filter By Equal
userQuery.where('age').equals(13);

// Filter By Greater Than
userQuery.where('age').gte('18')

// Filter By Less Than
userQuery.where('age').lte('18');
```


###Performance
####cursor()
Cursors are the native way mongodb navigates through the database.
With mongoose, an array is returned as a result of `find()` but the native driver returns a `Cursor`.

Mongoose allows us to get the data in the form a cursor/stream because it may be more performant in some cases.

```javascript
// There are 2 ways to use a cursor. First, as a stream:
Thing.
  find({ name: /^hello/ }).
  cursor().
  on('data', function(doc) { console.log(doc); }).
  on('end', function() { console.log('Done!'); });

// Or you can use `.next()` to manually get the next doc in the stream.
// `.next()` returns a promise, so you can use promises or callbacks.
const cursor = Thing.find({ name: /^hello/ }).cursor();
cursor.next(function(error, doc) {
  console.log(doc);
});

// Because `.next()` returns a promise, you can use co
// to easily iterate through all documents without loading them
// all into memory.
const cursor = Thing.find({ name: /^hello/ }).cursor();
for (let doc = await cursor.next(); doc != null; doc = await cursor.next()) {
  console.log(doc);
}
```

###Sorting
####sort()
The field name is the key, and the value states whether it's ascending or descending. There are different ways to implement this:
```javascript
// -1 or 1
todosQuery.sort({ title: -1, description: 1 });

// 'ascending' or 'descending'
todosQuery.sort({ title: 'descending', description: 'ascending' });

// 'asc' or 'desc'
todosQuery.sort({ title: 'desc', description: 'asc' });

// shorthand - title is ascending and description is descending
todosQuery.sort('title -description');
```

###Pagination
We can use a combination of `limit` and `skip` to implement pagination easily.
Before that, let's look at how these two methods work.
####limit()
```javascript
todosQuery.limit(100);
```

####skip()
This will skip the first `x` results of the `find` query.
```javascript
todosQuery.skip(20);
```

####Combining To Implement Pagination
Let's say that we are on page `2` and each page contains 10 results.
We'll create a variable called `page` which is `2` and `number` which is `10`.

So, we'll skip the first `(page - 1) * number` which will send the results 11 and above.

```javascript
const page = 2;
const number = 10;

todosQuery.skip((page - 1) * number);
```

Until now, we've skipped the first 10 results, but we still show all of remaining results.
Therefore, let's use the `limit()` method to limit the number of results.

```javascript
const page = 2;
const number = 10;

// skip = 2-1 * 10 = first 10 results
// limit = 2 * 10 = first 20 results
// the results are limited to the first 11 to 20 results
todosQuery.skip((page - 1) * number).limit(page * 10);
```