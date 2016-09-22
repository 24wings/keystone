# Introduction

`This guide is currently being updated to match the new Yeoman Generator.`TK

KeystoneJS makes it easy to build database-driven websites, applications and APIs in node.js.

Under the hood, KeystoneJS uses the [express.js](expressjs.com) web server framework, and a [MongoDB](mongodb.com) database via the [mongoose](mongoosejs.com) object modelling framework.

## Simple vs. Flexible

Keystone is designed to make complicated things simple, without limiting the power or flexibility of node.js or the frameworks it is built on.
This guide will show you how to build a KeystoneJS website using the default project structure and options.
To learn more about how things work under the hood, and how you can extend or replace features, we strongly recommend reading the [source code](https://github.com/keystonejs/keystone).

## Prerequisites
1. Before you begin, make sure you have [Node.js](nodejs.org) and [MongoDB](mongodb.com/download) installed.
2. You'll need a reasonable working knowledge of Javascript to use KeystoneJS, as well as familiarity with basics such as database concepts, and using node / npm etc.
3. In the guide we'll also be using JadeTK for our view templates and [LESS](lesscss.org/) for our CSS templates. In your own project you can use any template language you like; see [using other template languages](TK) (below) for more information.

## Production vs. Development

Keystone applies different settings in production and development modes. The environment will default to development, so you should set the `NODE_ENV` environment variable to `production` on your production servers for better performance.

Your app can detect which environment it is running in by calling `keystone.get('env')`.

# Getting Started

## Using the Yeoman Generator

The easiest way to get started with KeystoneJS is to use our new Yeoman Generator.

We're still updating our getting started guide to reflect this; in the meantime head over to the [KeystoneJS Yeoman Generator](https://github.com/keystonejs/generator-keystone) page and follow the instructions there.

Running `yo keystone` will take all the work out of following the guide below, so once you've got your new project, continue reading to follow along with what it did.

## Starting a new project

Create a new directory for your project, then add the following two files which are responsible for setting up your project and starting your webserver.

**package.json**
This file describes our project for npm, including the fact that it depends on `keystone`.
```
{
  "name": "my-project",
  "version": "0.0.1",
  "private": true,
  "dependencies": {
    "keystone": "latest",
    "underscore": "latest"
  },
  "engines": {
    "node": "0.10.x",
    "npm": "1.3.x"
  },
  "scripts": {
    "start": "web.js"
  }
}
```

> NOTE
> Note we're also requiring the underscore library, because we'll use some of its functionality later. You can use any other packages you like from npm in your Keystone application by adding them to your package.json.

**web.js**
This is the script that will run our keystone website.
```
var keystone = require('keystone');
keystone.init({

  'name': 'My Project',

  'favicon': 'public/favicon.ico',
  'less': 'public',
  'static': ['public'],

  'views': 'templates/views',
  'view engine': 'jade',

  'auto update': true,
  'mongo': 'mongodb://localhost/my-project',

  'session': true,
  'auth': true,
  'user model': 'User',
  'cookie secret': '(your secret here)'

});

require('./models');

keystone.set('routes', require('./routes'));

keystone.start();
```

Now, in your console, run `npm install` from the root folder of your project (where package.json is) to install Keystone.

> NOTE
> For more information about the options Keystone supports, see the configuration guide.
> NOTE
> Note: your web script won't run yet, because it is including models and routes that have not been set up yet. It assumes you follow the conventions in this guide.

### BYO Express and Mongoose

If you want to require Express or Mongoose in your application, instead of having Keystone provide and configure them completely, you can do so.

Include them in the dependencies list for your project, then provide them to Keystone using the `app` and `mongoose` options respectively.
```
var express = require('express'),
    mongoose = require('mongoose'),
    app = express(),
    keystone = require('keystone');

    keystone.set('app', app);
    keystone.set('mongoose', mongoose);
```

> NOTE
> The `keystone.connect()` method, previously used to set your own Express/Mongoose instances, is now deprecated and will be removed in future versions of Keystone. Due to changes in Express 4, `keystone.connect()` no longer works as expected. Please use the Keystone `app` and `mongoose` options in its place.

## Project Structure

With your package and web scripts in place, it's time to scaffold out containers for the rest of your app. Create the following directory structure:

```
|--lib
|  Custom libraries and other code
|--models
|  Your application's database models
|--public
|  Static files (css, js, images, etc.) that are publicly available
|--routes
|  |--api
|  |  Your application's api controllers
|  |--views
|  |  Your application's view controllers
|  |--index.js
|  |  Initialises your application's routes and views
|  |--middleware.js
|  |  Custom middleware for your routes
|--templates
|  |--includes
|  |  Common .jade includes go in here
|  |--layouts
|  |  Base .jade layouts go in here
|  |--mixins
|  |  Common .jade mixins go in here
|  |--views
|  |  Your application's view templates
|--updates
|  Data population and migration scripts
|--package.json
|  Project configuration for npm
|--web.js
|  Main script that starts your application
```

We also recommend that your application will be simpler to build and maintain if you mirror the internal structure of your `routes/views` and `templates/views` directories as much as possible.

### Structure

This guide assumes you follow the recommendations above, however Keystone doesn't actually enforce any structure, so you're free to make changes to suit your application better.

# Models

Before you can start your Keystone app, you'll need some data models.

We're going to start with the `User` model, which is special - we need it so that Keystone can do authentication and session management.

Create the following two files in the `/models` folder:

**models/index.js**
This is the script that includes your models. It doesn't need to export anything.
```require('./users.js');```

**models/users.js**
This script initialises the `User` model. It doesn't need to export anything, but the model *must* be registered with Keystone.
```
var keystone = require('keystone'),
    Types = keystone.Field.Types;

var User = new keystone.List('User');

User.add({
    name: { type: Types.Name, required: true, index: true },
    email: { type: Types.Email, initial: true, required: true, index: true },
    password: { type: Types.Password, initial: true },
    canAccessKeystone: { type: Boolean, initial: true }
});

User.register();
```

## Authentication and Session Management

For Keystone to provide authentication and session management to your application, it needs to know a few things (which we've now configured).

To recap:

- The option `user model` must be the name of the Model that Keystone should look in to find your users. If you use a different model name, be sure to set the option correctly.
- If you want your application to support session management, set the `session` option to true. Loading sessions incurs a small overhead, so if your application doesn't need sessions you can turn this off.
- Keystone has built-in views for signing in and out. To enable them, set the `auth` option to true. You can also implement custom signin and signout screens in your applications' views.
- Sessions are persisted using an encrypted cookie storing the user's ID. Make sure you set the `cookie secret` option to a long, random string.
- The user model must have a `canAccessKeystone` property (which can be a virtual method or a stored boolean) that says whether a user can access Keystone's Admin UI or not. \*Note\* If you choose to use a virtual method setting the value in mongodb directly will not authenticate correctly. A virtual method is useful when the criteria for access is more complex. See [Mongoose virtuals](mongoosejs.com/docs/guide.html#virtuals).

### More on Data Models

For more information on how to set up your application's models, and the full documentation for lists and fields, see the [database guide](TK).

# Routes & Views

Usually, the easiest and clearest way to configure the logic for different routes (or views) in your application is to set up all the bindings single file, then put any common logic (or middleware) in another file.

Then, the controller for each route you bind goes in its own file, organised similarly to the template that renders the view.

Keystone's `importer` and Express's middleware support makes this easy to set up.

## Setting up your Routes and Middleware

### Route index controller

First, create a `routes/index.js` file. This is where we bind your application's URL patterns to the controllers that load and process data, and render the appropriate template.

**routes/index.js**

This script imports your route controllers and binds them to URLs.
```
var keystone = require('keystone'),
    middleware = require('./middleware'),
    importRoutes = keystone.importer(__dirname);

// Common Middleware
keystone.pre('routes', middleware.initErrorHandlers);
keystone.pre('routes', middleware.initLocals);
keystone.pre('render', middleware.flashMessages);

// Handle 404 errors
keystone.set('404', function(req, res, next) {
    res.notfound();
});

// Handle other errors
keystone.set('500', function(err, req, res, next) {
    var title, message;
    if (err instanceof Error) {
        message = err.message;
        err = err.stack;
    }
    res.err(err, title, message);
});

// Load Routes
var routes = {
    views: importRoutes('./views')
};

// Bind Routes
exports = module.exports = function(app) {

    app.get('/', routes.views.index);

}
```


### Stepping through the route controller index

- Load `keystone`, the `middleware.js` file (below), and create an `importer` for the current directory
- Bind middleware (below) that
	- Initialises our basic error handlers
	- Initialises common local variables for our view templates
	- Retrieves flash messages from session before the view template is rendered
- Tell Keystone how to handle `404` and `500` errors
- Use the importer to load all the route controllers in the `/routes/views` directory
- Export a method that binds the index route controller to `GET` requests on the root url `/`
	- The `app` argument to this method our express app, so anything you can do binding routes in express, you can do here.
- Additional route controllers that you add to your app should be added using `app.get`, `app.post` or `app.all` under your root controller.

## Common Route Middleware

Putting your common middleware in a separate `routes/middleware.js` file keeps your route index nice and clean. If your middleware file gets too big, it's a good idea to restructure any significant functionality into custom modules in your projects `/lib` folder.

Now we'll add the basic middleware to get your app up and running with default behaviours:

**routes/middleware.js**
This script includes common middleware for your application routes
```
var _ = require('underscore'),
    keystone = require('keystone');

/**
    Initialises the standard view locals.
    Include anything that should be initialised before route controllers are executed.
*/
exports.initLocals = function(req, res, next) {

    var locals = res.locals;

    locals.user = req.user;

    // Add your own local variables here

    next();

};

/**
    Inits the error handler functions into `res`
*/
exports.initErrorHandlers = function(req, res, next) {

    res.err = function(err, title, message) {
        res.status(500).render('errors/500', {
            err: err,
            errorTitle: title,
            errorMsg: message
        });
    }

    res.notfound = function(title, message) {
        res.status(404).render('errors/404', {
            errorTitle: title,
            errorMsg: message
        });
    }

    next();

};

/**
    Fetches and clears the flashMessages before a view is rendered
*/
exports.flashMessages = function(req, res, next) {

    var flashMessages = {
        info: req.flash('info'),
        success: req.flash('success'),
        warning: req.flash('warning'),
        error: req.flash('error')
    };

    res.locals.messages = _.any(flashMessages, function(msgs) { return msgs.length }) ? flashMessages : false;

    next();

};
```

### Middleware functions

Keystone expects middleware functions to accept the following arguments:

- req - an express request object
- res - an express response object
- next - the method to call when the middleware has finished running (including any internal callbacks)

### Flash message support (no, not that flash)

Keystone includes support for 'flashing' messages to your visitors via session. The default setup above supports four categories, all of which can be styled differently:

- info
- success
- warning
- error

You can easily support other types of messages by updating your middleware and the .jade template that renders them (which we'll get to below).

To use flash messages in your route controllers, do this:

`req.flash('info', 'Some information!');`

Messages use session so they survive redirects, and will only be displayed to the user once, making them perfect for status messages (e.g. "Your changes have been saved") or form validation (e.g. "Please enter a valid email address").

Some Keystone features (such as the Update Handler) can automatically generate flash messages for you, and expect the categories above to be available.

## Your First View

Now we're going to set up your first route controller (for the index page), and the template it will render.

The importer (above) expects the directory you ask it for to include `.js` or `.coffee` files that export a single method accepting the following arguments:

- req - an express request object
- res - an express response object
Our first view controller is going to be very simple - just rendering a template. Create an `routes/views/index.js` file like this:

**routes/views/index.js**
The route controller for our home page view
```
var keystone = require('keystone');

exports = module.exports = function(req, res) {

    var view = new keystone.View(req, res);

    view.render('index');

}
```

## Templates

Now, for the template our route will `render`. The render method looks in the `views` directory specified in our `web.js`, which we set to `/templates/views`.

In this guide, we're going to use **Jade**TK for our templates. To learn more about Jade, visit [jade-lang.org](jade-lang.org), or check out the great [live syntax documentation](naltatis.github.io/jade-syntax-docs/) to learn by example.

First up, create `templates/views/index.jade`:

**templates/views/index.jade**
The template for our home page view
```
extends ../layouts/base

block content
    h1 Hello World
```

Jade comes with some great features to simplify templates - including using layouts that define regions. We're going to use a layout called `../common/templates/layout/base.jade`, which is included on the first line of the file above:

**templates/layouts/base.jade**
The base layout for our view templates
```
include ../mixins/flash-messages

doctype html
html
    head
        meta(charset="utf-8")
        meta(name="viewport", content="initial-scale=1.0,user-scalable=no,maximum-scale=1,width=device-width")

        title= title || 'My Keystone Website'
        link(rel="shortcut icon", href="/favicon.ico", type="image/x-icon")
        link(href="/styles/site.min.css", rel="stylesheet")

        block css
        block head
    body

        #header My Keystone Website

        #body

            block intro

            +flash-messages(messages)

            block content

        #footer Powered by <a href='http://keystonejs.com', target='_blank'>KeystoneJS</a>.

    script(src='/js/lib/jquery/jquery-1.10.2.min.js')

    block js
```

We're also going to create a `templates/mixins/flash-messages.jade` template to include the `flash-messages` mixin. Including mixins in your layout or view templates is a great way to keep your layout and view files clean, and re-use mixins across multiple views.

**templates/mixins/flash-messages.jade**
Our flash-messages mixin
```
mixin flash-messages(messages)
    if messages
        #flash-messages.container
            each message in messages.info
                +flash-message(message, 'info')
            each message in messages.success
                +flash-message(message, 'success')
            each message in messages.warning
                +flash-message(message, 'warning')
            each message in messages.error
                +flash-message(message, 'danger')

mixin flash-message(message, type)
    div(class='alert alert-' + type)
        if utils.isObject(message)
            if message.title
                h4= message.title
            if message.detail
                p= message.detail
            if message.list
                ul
                    each item in message.list
                        li= item
        else
            = message
```

### Using other template languages

KeystoneJS supports any [template language supported by express](expressjs.com/en/api.html).

Use the `view engine` option to specify the template language you want to use (it will default to `jade`).

If you want to use a custom template engine, set the `custom engine` option as well. For instance, [ejs](embeddedjs.com/) is supported by express by default, but you might want to use [ejs.locals](github.com/RandomEtc/ejs-locals) as a template engine in order to benefit from get extensions.
```
// Modified web.js to use the ejs-locals custom template engine.
var keystone = require('keystone');
var engine   = require('ejs-locals');
keystone.init({
  ...
  'custom engine': engine,
  'view engine': 'ejs',
  ...
});
```

## Public Assets

You'll want to add your own css, javascript, images and other files to your project. In the examples above, we're including `/styles/site.min.css` and `/js/lib/jquery-1.10.2.min.js`.

Keystone will serve any static assets you place in the public directory. This path is specified in `web.js` by the `static` option.

It will also automatically generate `.css` or compressed `.min.css` files when a corresponding `.less` file is found in the public folder, as specified in `web.js` by the `less` option. For more information on LESS, see [lesscss.org](lesscss.org).

# Running Your App

You're now (almost) ready to run your app! Before we do, though, we should add a **User** so you can sign in to Keystone's Admin UI.

## Writing Updates

To do this, we're going to create an update script, which Keystone will automatically run before starting the web server.

Keystone's automatic update functionality is enabled in `web.js` by the auto `update option`.

When the option is set to `true`, Keystone will scan the `updates` directory for `.js` files, each of which should export a method accepting a single argument:

- next - the method to call when the update has finished running (including any internal callbacks)

Updates are ordered using [Semantic Versioning](semver.org), and Keystone will only run them once (successfully executed updates are stored in your database, in a collection called `app_updates`).

Update file names should match the pattern `x.x.x-description.js` - anything after the first hyphen is ignored, so you can describe the update in the filename.

So to automatically add a new Admin User when your app first launches, create a `updates/0.0.1-admin.js` file:

**updates/0.0.1-admin.js**
Update script to add the first admin (change to your own name, email and password)
```
var keystone = require('keystone'),
    User = keystone.list('User');

exports = module.exports = function(done) {

    new User.model({
        name: { first: 'Admin', last: 'User' },
        email: 'admin@keystonejs.com',
        password: 'admin',
        canAccessKeystone: true
    }).save(done);

};
```

> NOTE
> You probably don't want to store your real password in the code, so it's a good idea to set the default password to something simple, then sign in and change it using Keystone's Admin UI after you start your app for the first time.

## Starting Keystone

Now you're ready to run your application, so execute the following in your project's main folder:

`node web`

Keystone will automatically apply the update, and then start a web server on the default port, 3000.

To see your home page, point your browser at [localhost:3000](localhost:3000). You should see our **Hello World!** message.

To sign in to Keystone's Admin UI, go to [localhost:3000/keystone](localhost:3000/keystone). Use the email and password you put in the update script above to sign in, and you'll be redirected to Keystone's home page.

## Next Steps

... you're done! Well, not really. It's time to start building your app now. For more information on list options and the field types Keystone supports, browse the [database guide](TK).

You should also [Follow @KeystoneJS on Twitter](twitter.com/keystonejs) for news and updates, [Star KeystoneJS on GitHub](github.com/keystonejs/keystone), and discuss this guide (or anything KeystoneJS related) on the [KeystoneJS Google Group](groups.google.com/d/forum/keystonejs).

We've also got more [Examples and Sample Code](TK) for you on the examples page.

Enjoy using KeystoneJS!
