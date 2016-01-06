---
layout: default
version: 2
---

## Getting started
------------------

**package.json**

{% highlight javascript %}
{
  "dependencies": {
    "express-restify-mongoose": "^2.0.0",
    "mongoose": "^4.0.0"
  }
}
{% endhighlight %}

**From the command line**

{% highlight javascript %}
npm install express-restify-mongoose --save
{% endhighlight %}

### Express 4 app

<div class="row">
  <div class="col-md-6">
    <p>This snippet...</p>
{% highlight javascript %}
var express = require('express')
var bodyParser = require('body-parser')
var methodOverride = require('method-override')
var mongoose = require('mongoose')
var restify = require('express-restify-mongoose')
var app = express()
var router = express.Router()

app.use(bodyParser.json())
app.use(methodOverride())

mongoose.connect('mongodb://localhost/database')

restify.serve(router, mongoose.model('Customer', new mongoose.Schema({
  name: { type: String, required: true },
  comment: { type: String }
})))

app.use(router)

app.listen(3000, function () {
  console.log('Express server listening on port 3000')
})
{% endhighlight %}
  </div>
  <div class="col-md-6">
    <p>...automatically generates those endpoints.</p>
{% highlight apache %}
GET http://localhost/api/v1/Customers/count
GET http://localhost/api/v1/Customers
PUT http://localhost/api/v1/Customers
POST http://localhost/api/v1/Customers
DELETE http://localhost/api/v1/Customers

GET http://localhost/api/v1/Customers/:id
GET http://localhost/api/v1/Customers/:id/shallow
PUT http://localhost/api/v1/Customers/:id
POST http://localhost/api/v1/Customers/:id
DELETE http://localhost/api/v1/Customers/:id
{% endhighlight %}
  </div>
</div>

### Usage with [request](https://www.npmjs.com/package/request)

{% highlight javascript %}
var request = require('request')

request({
  url: '/api/v1/Model',
  qs: {
    query: JSON.stringify({
      $or: [{
        name: '~Another'
      }, {
        $and: [{
          name: '~Product'
        }, {
          price: '<=10'
        }]
      }],
      price: 20
    })
  }
})
{% endhighlight %}

## Querying
-----------

All the following parameters (sort, skip, limit, query, populate, select and distinct) support the entire mongoose feature set.

> When passing values as objects or arrays in URLs, they must be valid JSON

### Sort

{% highlight apache %}
GET /Customers?sort=name
GET /Customers?sort=-name
GET /Customers?sort={"name":1}
GET /Customers?sort={"name":0}
{% endhighlight %}

### Skip

{% highlight apache %}
GET /Customers?skip=10
{% endhighlight %}

### Limit

Only overrides `options.limit` if the queried limit is lower

{% highlight apache %}
GET /Customers?limit=10
{% endhighlight %}

### Query

Supports all operators ($regex, $gt, $gte, $lt, $lte, $ne, etc.) as well as shorthands: ~, >, >=, <, <=, !=

{% highlight apache %}
GET /Customers?query={"name":"Bob"}
GET /Customers?query={"name":{"$regex":"^(Bob)"}}
GET /Customers?query={"name":"~^(Bob)"}
GET /Customers?query={"age":{"$gt":12}}
GET /Customers?query={"age":">12"}
GET /Customers?query={"age":{"$gte":12}}
GET /Customers?query={"age":">=12"}
GET /Customers?query={"age":{"$lt":12}}
GET /Customers?query={"age":"<12"}
GET /Customers?query={"age":{"$lte":12}}
GET /Customers?query={"age":"<=12"}
GET /Customers?query={"age":{"$ne":12}}
GET /Customers?query={"age":"!=12"}
{% endhighlight %}

### Populate

Works with create, read and update operations

{% highlight apache %}
GET/POST/PUT /Invoices?populate=customer
GET/POST/PUT /Invoices?populate={"path":"customer"}
GET/POST/PUT /Invoices?populate=[{"path":"customer"},{"path":"products"}]
{% endhighlight %}

### Select

`_id` is always returned unless explicitely excluded

{% highlight apache %}
GET /Customers?select=name
GET /Customers?select=-name
GET /Customers?select={"name":1}
GET /Customers?select={"name":0}
{% endhighlight %}

### Distinct

{% highlight apache %}
GET /Customers?distinct=name
{% endhighlight %}

## Reference
------------

### serve

{% highlight javascript %}
restify.serve(router, model[, options])
{% endhighlight %}

**router**: express.Router() instance (Express 4), app object (Express 3) or server object (restify)

**model**: mongoose model

**options**: object <span class="label label-primary">type</span><span class="label label-success">default</span><span class="label label-info">version</span>

#### prefix
<span class="label label-primary" title="type">string</span><span class="label label-success" title="default">/api</span>

Path to prefix to the REST endpoint

#### version
<span class="label label-primary" title="type">string</span><span class="label label-success" title="default">/v1</span>

API version that will be prefixed to the rest path. If prefix or version contains `/:id`, then that will be used as the location to search for the id

##### Example

Generates `/api/v1/Entities/:id/Model` and `/api/v1/Entities/Model` for all pertinent methods

{% highlight javascript %}
version: '/v1/Entities/:id'
{% endhighlight %}

#### idProperty
<span class="label label-primary" title="type">string</span><span class="label label-success" title="default">_id</span>

`findById` will query on the given property

#### restify
<span class="label label-primary" title="type">boolean</span><span class="label label-success" title="default">false</span>

Enable support for [restify](https://www.npmjs.com/package/restify) instead of [express](https://www.npmjs.com/package/express)

#### plural
<span class="label label-primary" title="type">boolean</span><span class="label label-success" title="default">false</span>

Automatically pluralize model names using [inflection](https://www.npmjs.com/package/inflection)

#### lowercase
<span class="label label-primary" title="type">boolean</span><span class="label label-success" title="default">false</span>

Whether to call `.toLowerCase()` on model names before generating the routes

#### name
<span class="label label-primary" title="type">string</span><span class="label label-success" title="default">model name</span>

Endpoint name

#### readPreference
<span class="label label-primary" title="type">string</span><span class="label label-success" title="default">primary</span>

Determines the MongoDB nodes from which to read. [Read more](http://mongoosejs.com/docs/api.html#query_Query-read)

#### private
<span class="label label-primary" title="type">array</span>

Array of fields which are only to be returned by queries that have private access

##### Example

Defined in options

{% highlight javascript %}
private: ['topSecret', 'fields']
{% endhighlight %}

Defined in mongoose schema

{% highlight javascript %}
new Schema({
  topSecret: { type: String, access: 'protected' },
  fields: { type: String, access: 'protected' }
})
{% endhighlight %}

#### protected
<span class="label label-primary" title="type">array</span>

Array of fields which are only to be returned by queries that have private or protected access

##### Examples

Defined in options

{% highlight javascript %}
protected: ['somewhatSecret', 'keys']
{% endhighlight %}

Defined in mongoose schema

{% highlight javascript %}
new Schema({
  somewhatSecret: { type: String, access: 'protected' },
  keys: { type: String, access: 'protected' }
})
{% endhighlight %}

#### lean
<span class="label label-primary" title="type">boolean</span><span class="label label-success" title="default">true</span>

Whether or not mongoose should use `.lean()` to convert results to plain old JavaScript objects. This is bad for performance, but allows returning virtuals, getters and setters.

#### findOneAndUpdate
<span class="label label-primary" title="type">boolean</span><span class="label label-success" title="default">true</span>

Whether to use findOneAndUpdate or first findById and then save, allowing document middleware to be called. For more information regarding mongoose middleware, [read the docs](http://mongoosejs.com/docs/middleware.html).

#### findOneAndRemove
<span class="label label-primary" title="type">boolean</span><span class="label label-success" title="default">true</span>

Whether to use findOneAndRemove or first findById and then remove, allowing document middleware to be called. For more information regarding mongoose middleware, [read the docs](http://mongoosejs.com/docs/middleware.html).

#### preMiddleware
<span class="label label-primary" title="type">function (req, res, next)</span>

Middleware that runs before [preCreate](#preCreate), [preRead](#preRead), [preUpdate](#preUpdate) and [preDelete](#preDelete). 

##### Example

{% highlight javascript %}
preMiddleware: function (req, res, next) {
  performAsyncLogic(function (err) {
    next(err)
  }
}
{% endhighlight %}

#### preCreate
<span class="label label-primary" title="type">function (req, res, next)</span><span class="label label-info">2.1</span>

Middleware that runs before creating a resource

{% highlight javascript %}
preCreate: function (req, res, next) {
  performAsyncLogic(function (err) {
    next(err)
  }
}
{% endhighlight %}

#### preRead
<span class="label label-primary" title="type">function (req, res, next)</span><span class="label label-info">2.1</span>

Middleware that runs before reading a resource

{% highlight javascript %}
preRead: function (req, res, next) {
  performAsyncLogic(function (err) {
    next(err)
  }
}
{% endhighlight %}

#### preUpdate
<span class="label label-primary" title="type">function (req, res, next)</span><span class="label label-info">2.1</span>

Middleware that runs before updating a resource

{% highlight javascript %}
preUpdate: function (req, res, next) {
  performAsyncLogic(function (err) {
    next(err)
  }
}
{% endhighlight %}

<span class="label label-info">new in 2.2</span>

When `findOneAndUpdate` is disabled, the document is made available which is useful for authorization as well as setting values

{% highlight javascript %}
findOneAndUpdate: false,
preUpdate: function (req, res, next) {
  if (req.erm.document.user !== req.user._id) {
    return res.sendStatus(401)
  }

  req.erm.document.set('lastRequestAt', new Date())

  next()
}
{% endhighlight %}

#### preDelete
<span class="label label-primary" title="type">function (req, res, next)</span><span class="label label-info">2.1</span>

Middleware that runs before deleting a resource

{% highlight javascript %}
preDelete: function (req, res, next) {
  performAsyncLogic(function (err) {
    next(err)
  }
}
{% endhighlight %}

<span class="label label-info">new in 2.2</span>

When `findOneAndRemove` is disabled, the document is made available which is useful for authorization as well as performing non-destructive removals

{% highlight javascript %}
findOneAndRemove: false,
preDelete: function (req, res, next) {
  if (req.erm.document.user !== req.user._id) {
    return res.sendStatus(401)
  }

  req.erm.document.deletedAt = new Date()
  req.erm.document.save().then(function (doc) {
    res.sendStatus(204)
  }, function (err) {
    options.onError(err, req, res, next)
  })
}
{% endhighlight %}

#### access
<span class="label label-primary" title="type">function (req[, done])</span>

Returns or yields 'private', 'protected' or 'public'. It is called on GET, POST and PUT requests and filters out the fields defined in [private](#private) and [protected](#protected)

##### Examples

Sync

{% highlight javascript %}
access: function (req) {
  if (req.isAuthenticated()) {
    return req.user.isAdmin ? 'private' : 'protected'
  } else {
    return 'public'
  }
}
{% endhighlight %}

Async

{% highlight javascript %}
access: function (req, done) {
  performAsyncLogic(function (err, result) {
    done(err, result ? 'public' : 'private')
  })
}
{% endhighlight %}

#### contextFilter
<span class="label label-primary" title="type">function (model, req, done)</span>

Allows request specific filtering

##### Example

{% highlight javascript %}
contextFilter: function (model, req, done) {
  done(model.find({
    user: req.user._id
  }))
}
{% endhighlight %}

#### postCreate
<span class="label label-primary" title="type">function (req, res, next)</span><span class="label label-info">2.0</span>

Middleware that runs after successfully creating a resource

{% highlight javascript %}
postCreate: function (req, res, next) {
  var result = req.erm.result         // object
  var statusCode = req.erm.statusCode // 201

  performAsyncLogic(function (err) {
    next(err)
  }
}
{% endhighlight %}

#### postRead
<span class="label label-primary" title="type">function (req, res, next)</span><span class="label label-info">2.0</span>

Middleware that runs after successfully reading a resource

{% highlight javascript %}
postRead: function (req, res, next) {
  var result = req.erm.result         // object / array
  var statusCode = req.erm.statusCode // 200

  performAsyncLogic(function (err) {
    next(err)
  }
}
{% endhighlight %}

#### postUpdate
<span class="label label-primary" title="type">function (req, res, next)</span><span class="label label-info">2.0</span>

Middleware that runs after successfully updating a resource

{% highlight javascript %}
postUpdate: function (req, res, next) {
  var result = req.erm.result         // object
  var statusCode = req.erm.statusCode // 200

  performAsyncLogic(function (err) {
    next(err)
  }
}
{% endhighlight %}

#### postDelete
<span class="label label-primary" title="type">function (req, res, next)</span><span class="label label-info">2.0</span>

Middleware that runs after successfully deleting a resource

{% highlight javascript %}
postDelete: function (req, res, next) {
  var result = req.erm.result         // undefined
  var statusCode = req.erm.statusCode // 204

  performAsyncLogic(function (err) {
    next(err)
  }
}
{% endhighlight %}

#### outputFn
<span class="label label-primary" title="type">function (req, res)</span>

Function used to output the result

##### Example

{% highlight javascript %}
outputFn: function (req, res) {
  res.status(req.erm.statusCode).json(req.erm.result)
}
{% endhighlight %}

#### onError
<span class="label label-primary" title="type">function (err, req, res, next)</span><span class="label label-success" title="default">send the entire mongoose error</span>

<div class="alert alert-warning">
  <i class="glyphicon glyphicon-alert"></i> Leaving this as default may leak information about your database
</div>

Function used to output an error

##### Example

{% highlight javascript %}
onError: function (err, req, res, next) {
  next(err)
}
{% endhighlight %}

### defaults

{% highlight javascript %}
restify.defaults(options)
{% endhighlight %}

**options**: same as above, sets this object as the defaults for anything served afterwards