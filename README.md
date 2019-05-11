[hapi](https://hapijs.com) is a battle-tested, full-featured, framework for building web applications and services with Node.js. With integrated support for essentials like authentication, caching and validation, and a powerful plugin system, hapi is ideal for projects and teams of any size.

This course will introduce hapi and guide you through some of hapi's core features.

At the end you'll be able to program basic hapi app and deploy it to `now.sh`.


# Getting Started

Let's start off by creating a new project and install hapi. Run the following in your terminal to get started:

```
# Create a new directory for the project
$ mkdir hapi-api && cd hapi-api

# Initialise a new Node project
$ npm init -y

# Install hapi.js
$ npm install @hapi/hapi

# Create a new server file
$ touch index.js

# Install nodemon
$ npm i nodemon -g

# Run server with nodemon
$ nodemon index.js
```

We're making use of nodemon to start our server in watch mode.

The first thing we need to do is create a server. Thankfully, this is easy with Node.js!

## Creating a Server

A very basic hapi server looks like the following:

```
# index.js
const Hapi = require('@hapi/hapi');

const init = async () => {
    const server = Hapi.server({
        port: 3000,
        host: 'localhost',
    });

    await server.start();
    console.log('Server running on %s', server.info.uri);
};

process.on('unhandledRejection', err => {
    console.log(err);

    process.exit(1);
});

init();
```

First, you require hapi. Then you initialize a new `Hapi.server()` with connection details containing a port number to listen on and the host information. After that you start the server and log that it's running.

## Adding Routes

After you get the server up and running, its time to add a route that will display "Hello World!" in your browser.

```
# index.js
const Hapi = require('@hapi/hapi');

const init = async () => {
    const server = Hapi.server({
        port: 3000,
        host: 'localhost',
    });

    server.route({
        method: 'GET',
        path:'/',
        handler: (request, h) => 'Hello World!',
    });

    await server.start();
    console.log('Server running on %s', server.info.uri);
};

process.on('unhandledRejection', (err) => {
    console.log(err);

    process.exit(1);
});

init();
```

Save the above as `index.js` and start the server with the command `nodemon index.js`. Now you'll find that if you visit `http://localhost:3000` in your browser, you'll see the text 'Hello, World!'.

The `method` property can be any valid HTTP method, array of HTTP methods, or an asterisk to allow any method.

The `path` property defines the path including parameters. It can contain optional parameters, numbered parameters, and even wildcards.

The `handler` function performs the main business logic of the route and sets the response. The `handler` must return a value, a promise, or throw an error.

## Parameters

We're also able to take this further with the request and h parameters. Let's add the ability to pass parameters into our URL:

```
# index.js
const Hapi = require('@hapi/hapi');

const init = async () => {
    const server = Hapi.server({
        port: 3000,
        host: 'localhost',
    });

    server.route({
        method: 'GET',
        path:'/',
        handler: (request, h) => 'Hello World!',
    });

    server.route({
      path: '/{id}',
      method: 'GET',
      handler: (request, h) => {
        return `Product ID: ${encodeURIComponent(request.params.id)}`;
      }
    });

    await server.start();
    console.log('Server running on %s', server.info.uri);
};

process.on('unhandledRejection', (err) => {
    console.log(err);

    process.exit(1);
});

init();
```

In our small example, we're imagining that we have an API that returns a particular product. Whenever the user requests `http://localhost:3000/123`, they'll get back:

```
Product ID: 123
```

This is because the `request.params` object contains any params that we've set-up in our path.

Notice that we surrounded the `id` inside of two braces: `{id}`, this tells hapi that we intend for a user to replace that part of the URL as a param.

At the same time, we also kept the original route _without_ the `id`. This shows that we can have multiple routes that target a similar base pattern and they won't override one another. Each one gets more specific, and if it doesn't match a particular route, it'll look back in the stack until one is matched.


## Plugins

hapi has an extensive and powerful plugin system that allows you to very easily break your application up into isolated pieces of business logic, and reusable utilities. You can either add an existing plugin to your application, or create your own.

### Loading a plugin

Plugins can be loaded one at a time, or as a group in an array, by the server.register(plugins, [options]) method, for example:

```
const start = async function () {
    const server = Hapi.server();

    // load one plugin
    await server.register(require('myplugin'));

    // load multiple plugins
    await server.register([require('myplugin'), require('yourplugin')]);
};
```

To pass `options` to your plugin, we instead pass an object with plugin and options keys, such as:

```
const start = async function () {
    const server = Hapi.server();

    await server.register({
        plugin: require('myplugin'),
        options: {
            message: 'hello'
        }
    });
};
```

These objects can also be passed in an array:

```
const start = async function () {
    const server = Hapi.server();

    await server.register([{
        plugin: require('plugin1'),
        options: {}
    }, {
        plugin: require('plugin2'),
        options: {}
    }]);
};
```

## Using authentication plugin

We will use node.js module (Hapi plugin) [hapi-auth-jwt2](https://github.com/dwyl/hapi-auth-jwt2) that lets you use JSON Web Tokens (JWTs) for authentication in your Hapi.js web application.

If you are totally new to JWTs, we wrote an introductory post explaining the concepts & benefits: 
[learn json web tokens](https://github.com/dwyl/learn-json-web-tokens)

### Install from NPM

```
npm install hapi-auth-jwt2 --save
```

### Example

This basic usage example should help you get started:

```
const Hapi = require('@hapi/hapi');

const people = { // our "users database"
    1: {
      id: 1,
      name: 'Jen Jones'
    }
};

// bring your own validation function
const validate = async function (decoded, request) {
    // do your checks to see if the person is valid
    if (!people[decoded.id]) {
      return { isValid: false };
    }
    else {
      return { isValid: true };
    }
};

const init = async () => {
  const server = new Hapi.Server({ port: 8000 });
  // include our module here ‚Üì‚Üì
  await server.register(require('hapi-auth-jwt2'));

  server.auth.strategy('jwt', 'jwt',
  { key: 'NeverShareYourSecret',          // Never Share your secret key
    validate: validate,            // validate function defined above
    verifyOptions: { algorithms: [ 'HS256' ] } // pick a strong algorithm
  });

  server.auth.default('jwt');

  server.route([
    {
      method: "GET", path: "/", config: { auth: false },
      handler: function(request, reply) {
        reply({text: 'Token not required'});
      }
    },
    {
      method: 'GET', path: '/restricted', config: { auth: 'jwt' },
      handler: function(request, reply) {
        reply({text: 'You used a Token!'})
        .header("Authorization", request.headers.authorization);
      }
    }
  ]);
  await server.start();
  return server;
};


init().then(server => {
  console.log('Server running at:', server.info.uri);
})
.catch(error => {
  console.log(error);
});
```

You can also find the _quick demo_ example in [hapi-auth-jwt2](https://github.com/dwyl/hapi-auth-jwt2)

## How to deploy your app to now.sh

[ZEIT Now](https://zeit.co/now) is a cloud platform for serverless deployment. It enables developers to host websites and web services that deploy instantly, scale automatically, and require no supervision, all with minimal configuration.

Run this to install `now` globally:

```
npm i -g now
```

To deploy your app to the cloud create in the your app directory a file named `now.json` that contains:

```
{
  "version": 2,
  "builds": [
    { "src": "index.js", "use": "@now/node-server" }
  ],
  "routes": [
    { "src": "/(.*)", "dest": "/index.js" }
  ]
}
```

And run the command:

```
$ now
```

As the result you'll get the url which you can use to access the app.

If you need more information about `now`, follow the guide through [Now documentation](https://zeit.co/docs/v2/deployments/official-builders/node-js-server-now-node-server)

