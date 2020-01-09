name: inverse
layout: true
class: center, middle, inverse
---

**Creative Coding I: Design & Communication - Winter Term 19/20**

Prof. Dr. Lena Gieseke | l.gieseke@filmuniversitaet.de | Film University Babelsberg *KONRAD WOLF*

Wednesdays, 10.00-13.30, Room 2.044, House 7


<!-- 

h or ?: Toggle the help window
j: Jump to next slide
k: Jump to previous slide
b: Toggle blackout mode
m: Toggle mirrored mode.
c: Create a clone presentation on a new window
p: Toggle PresenterMode
f: Toggle Fullscreen
t: Reset presentation timer
<number> + <Return>: Jump to slide <number>

 -->

---
layout: false

## Topics

Continuing the development of a webpage, which recreates a [Fridge Magnetic Poetry Kit](https://magneticpoetry.com) on a p5 canvas.

.center[![wordkit_zombie](../img/09/wordkit_zombie.jpg)]

---

## Topics

A *Reizwortgeschichte* app...


---

### Fridge Poetry App - The Story

--

* Everybody sees at all times the same poetry board
* There is *one common state* of the magnets on all connected clients

--

**Goal 1**: If a magnet is moved, the same magnet moves on all connected clients as well  

--

**Goal 2**: The locations of all magnets are saved in a database and if a magnet moves, its database entry is updated  

--

**Goal 3**: Put everything online

---

### Fridge Poetry App - The Story

* Everybody sees at all times the same poetry board
* There is one common state of the magnets on all connected clients

**Goal 1**: If a magnet is moved, the same magnet moves on all connected clients as well ✔️ 🎉  
  
**Goal 2**: The locations of all magnets are saved in a database and if a magnet moves, its database entry is updated  
  
**Goal 3**: Put everything online

???

Goal 1: If a magnet moves, it moves on all connected clients as well  

* Bilateral server-client communication
* Clients need to send data back to the server
* The server needs to update all other clients when it received data from a client

---

### Package Installations

#### 1. Initializing the Node Environment

To configure the development environment, execute the following command inside your project folder (`fridgePoetryClass`).

```node
npm init
```

--

Now we have a `package.json` file, which tracks the packages we install for the project. 


???

TASK: Show package.json 

---

### Package Installations

#### 2. Packages

```bash
npm install --save express
```

Express is a micro web application framework for node.js.

```bash
npm install --save socket.io
```

socket.io is a package to manage WebSockets.

```bash
npm install --save mongoose
```

Mongoose provides a straight-forward, schema-based solution to model application data for a mongodb with node.js.

???
As we are using it also on the client side, we need to include the javascript library in the index.html:


---

## Web Applications

### Server

```js

// Loading the modules
const express = require('express');

// Listen to port 300
const port = process.env.PORT || 3000;
const server = express()
                .use(express.static('public'))
                .listen(port, () => console.log(`Listening on ${port}`));
```

???

This example creates a web server that listens for any kind of HTTP request on the URL http://127.0.0.1:3000/ .

The example uses the express library to handle the requests.

Please not that this code should go into a separate server file, e.g. `server.js`, not the sketch file, we used to work with.

--

Start the server with the terminal in your project folder with

```js
node server.js
```

???


The :3000 part is the [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) port. You can consider these ports as communication endpoints on a particular IP address (in the case of localhost - 127.0.0.1). 

You can imagine it as the following. In an apartment building, there is one address for multiple apartments. The address is a host, e.g. the localhost. Each apartment has its own mailbox, hence each mailbox is a port.

---

### WebSockets

--

WebSockets is a protocol that allows a bilateral synchronous exchange between a client and a server.  

Socket.io is a library based on this protocol to make the use of WebSockets easier. It works on every platform, browser or device, focusing equally on reliability and speed.

---

### WebSockets

.center[<img src="../img/09/socket.png" style="width:50%;">  [[Medium]](https://medium.com/@noufel.gouirhate/build-a-simple-chat-app-with-node-js-and-socket-io-ea716c093088)]

In the classic web, a client requests to a server and a server responds sending it back the data.   

---

### WebSockets

With WebSockets, the server can send data to the client, but the client can too! A WebSocket is a communication pipe open in two directions.

.center[<img src="../img/09/sockets_communication.png" style="width:50%;">  [[Medium]](https://medium.com/@noufel.gouirhate/build-a-simple-chat-app-with-node-js-and-socket-io-ea716c093088)]

???

A possible scenario can look like the following.

* At a client side a change happened (magnet is moved) and
* the client sends the new position back to the server,
* which then sends the new postion to all other clients.

We have to program this behavior ourself and socket.io gives us the commands for it.

---

### WebSockets

A basic client-server communication is setup as the following.

```js
// server.js

const express = require('express');

// TODO 3a
const socketIO = require('socket.io');

const port = process.env.PORT || 3000;
const server = express()
                .use(express.static('public'))
                .listen(port, () => console.log(`Listening on ${port}`));

// TODO 3b
// Creating a new communication pipe
// based on the given server and port
const io = socketIO(server);

io.on('connection', socket => console.log('Client connected'));
```

--

The above code however only sets up the communication pipe for the server, not the client. We still need to connect the client as well.

---

### WebSockets

As the client is using the library as well, we need to dd the socket.io library to the `index.html`:

```html
<!-- index.html -->

<script src="/socket.io/socket.io.js"></script>
```

Add to the sketch file `fridgepoetry.js` as first code line a socket.io object:

```js
// fridgepoetry.js

let socket = io();
```

---

### WebSockets

#### Bilateral Communication

Send out (*emit*) a event from the client side:

```js
// fridgepoetry.js

let socket = io();
socket.emit('sayingHello');
```

--

Receive that event on the server side and react to it:

```js
// server.js

...

const io = socketIO(server);
io.on('connection', socket => 
{
    console.log('Client connected')

    socket.on('sayingHello', () => console.log('Got the hello'));
});
```

---

### WebSockets

#### Bilateral Communication

Remember that we are heavily using the arrow function notation, which is the same as defining functions and passing them as arguments:

```js
io.on('connection', socket => 
{
    console.log('Client connected')

    socket.on('sayingHello', () => console.log('Got the hello'));
});
```

```js

io.on('connection', connectionLive);

function connectionLive(socket)
{
    console.log('Client connected')

    socket.on('sayingHello', receivedHello);

    function receivedHello()
    {
        console.log('Got the hello');
    }
}

```

---

### WebSockets

#### Bilateral Communication

We could also add data to sent from the client to the server:

```js
// fridgepoetry.js

let socket = io();
let data = 'data, data, so much data...';
socket.emit('sayingHello', data);
```

```js
// server.js

...

const io = socketIO(server);
io.on('connection', socket => 
{
    console.log('Client connected')

    socket.on('sayingHello', data => console.log(`Got: ${ data }`));
});
```

[[4]](#4-source-build-a-simple-chat-app-with-nodejs-and-socketio) [[6]](#6-socketio-documentation)


<!----------------------------------------------------------------------------->
<!-------------------------------- NEW SECTION -------------------------------->
<!----------------------------------------------------------------------------->

---

## The Fridge Poetry App - Implementation

### Moved Magnet

--

The client emits a signal and sends the current magnet and its location

```js
// fridgepoetry.js

if(this.dragged)
{
    ...

    // TODO 6: 
    // 6a. Create a magnet object
    let data =
    {
        index: this.index,
        x: this.x,
        y: this.y
    }
    // 6b. and send it to the server 
    // (emit a signal) 
    socket.emit('clientMagnetMove', data);
}
```

---

## The Fridge Poetry App - Implementation

### Moved Magnet

The server receives it:

```js
// server.js

io.on('connection', socket =>
{
    ... 

    // TODO 7
    // Receive moving signal from a client
    socket.on('clientMagnetMove', (data) =>
    {
        console.log('Moved:', data);
    });

    ...
}
```

---

## The Fridge Poetry App - Implementation

### Moved Magnet

#### Broadcast to All Clients

--

Now the server should send (*broadcast*) the move to all other clients.  

The broadcast methods send the data to all clients, except the one with which the server just communicated. 

--

```js
// server.js

...

    // Receive moving signal from a client
    socket.on('clientMagnetMove', (data) =>
    {
        // Send the movement data to all
        // other clients
        // TODO 8
        socket.broadcast.emit('serverBroadcastMagnetMove', data);
    });
...
```

---

#### Broadcast to All Clients

This event needs to be received from the clients. As we created the index from the array position of the magnet, we can use the index in turn as index to access the array.

```js
// fridgepoetry.js

function setup() 
{
    ...

    // TODO 9
    // Receive signal from server that
    // a magnet moved
    socket.on('serverBroadcastMagnetMove', (data) =>
    {
        console.log('I received:', data);

        // Set the position of the magnet
        poetry.magnets[data.index].x = data.x;
        poetry.magnets[data.index].y = data.y;
    });

    ...
```

???

TASK: Show console log, and moving magnets in two browser windows

---
template:inverse

## Goal 1  ✔️  🎉


???

TASK: Everybody should see the magnet moving in different screens

---

### Fridge Poetry App

#### Goal 2: The locations of all magnets are saved in a database and if a magnet moves, its database entry is updated  

--

* A database and a connection from the server to it

--

* A representation of a magnet in the database: id, location

--

* If the magnets are not yet saved in the database, an initial saving of all magnets with random locations

--

* When a new client connects, the server sends it the current locations of all magnets and the board is drawn accordingly

--

* When a magnet is moved its database entry is updated accordingly

---
template:inverse

## Database

---

## Database

A database stores data in an organized way so that it can be searched and retrieved later.

Overall, there are two standardized database models: relational and non-relational models.

[[1]](#1-wikipedia-database)

???

TASK: What is the difference?

Relational Database

* A table is much like a spreadsheet, in that it's made up of rows and columns. All rows have the same columns, and each column contains the data itself. You can think of a tables in the same way that you would of a table in Excel.
* Popular SQL databases are MySQL, PostgreSQL and Oracle among many others.

Non-Relational Database

* Non-relational databases have grown in popularity because they were designed to overcome the limitations of relational databases in dealing with *big data* demands. 
* Think of non-relational databases more like file folders, assembling related information of all types. 
* Non-relational databases offer greater flexibility than their traditional counterparts. If your data requirements aren’t clear at the outset or if you’re dealing with massive amounts of unstructured data, you may not have the luxury of developing a relational database with clearly defined schema. 
* On of the most popular NoSQL system is MongoDB - the system we are going to use.

---

### Non-Relational Database

#### MongoDB

MongoDB is a document-oriented database with JSON-like documents in dynamic schemata instead of relational tables. [Mongoose](https://mongoosejs.com/) is a javascript package, which provides a straight-forward, schema-based solution to model your application data.  

MongoDB works with the concept of *collections* and *documents*.

##### Collection

A Collection is a group of MongoDB documents, like a folder. It is the equivalent of an relational database table. 

##### Document

A document is a set of key-value pairs of properties and functionalities and are declared in a *schema*. 

---

### Non-Relational Database

#### MongoDB

We define how to save the data in a collection with key-value pairs in a schema.

A schema for a blog post document could look as the following.

```js
let blogSchema = new mongoose.Schema(
{
    title:  String,
    author: String,
    body:   String,
    date: Date,
    hidden: Boolean,
    meta: {
        likes: Number,
        favs:  Number
    }
});

```

???

Documents have a dynamic schema. Dynamic schema means that documents in the same collection do not need to have the same set of fields or structure, and common fields in a collection's documents may hold different types of data.

TASK: Go to objects in javascript script

---

##### Mongoose

A simple example with mongoose might look as the following.

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/test', {useNewUrlParser: true});


```

[[8]](#8-mongoose-documentation)

---


##### Mongoose

A simple example with mongoose might look as the following.

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/test', {useNewUrlParser: true});

// Defining the data structure, schema
let catSchema = new mongoose.Schema(
{
    name: String
});


```

[[8]](#8-mongoose-documentation)


---


##### Mongoose

A simple example with mongoose might look as the following.

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/test', {useNewUrlParser: true});

// Defining the data structure, schema
let catSchema = new mongoose.Schema(
{
    name: String
});

//  Defining a model with the schema
const Cat = mongoose.model('Cat', catSchema);

```

--

Models are responsible for creating and reading documents from the underlying MongoDB database.

--

The first argument is the *singular* name of the collection your model is for. Mongoose automatically looks for the plural version of your model name, in our case a `magnets` collection.

[[8]](#8-mongoose-documentation)

???

Models are fancy constructors compiled from Schema definitions. An instance of a model is called a document. Models are responsible for creating and reading documents from the underlying MongoDB database.

Models are defined using the Schema interface. The Schema allows you to define the fields stored in each document along with their validation requirements and default values. In addition, you can define static and instance helper methods to make it easier to work with your data types, and also virtual properties that you can use like any other field, but which aren't actually stored in the database (we'll discuss a bit further below).

Schemas are then "compiled" into models using the mongoose.model() method. Once you have a model you can use it to find, create, update, and delete objects of the given type.

Note: Each model maps to a collection of documents in the MongoDB database. The documents will contain the fields/schema types defined in the model Schema.

https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/mongoose



---


##### Mongoose

A simple example with mongoose might look as the following.

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/test', {useNewUrlParser: true});

// Defining the data structure, schema
let catSchema = new mongoose.Schema(
{
    name: String
});

//  Defining a model with the schema
const Cat = mongoose.model('Cat', catSchema);

// Creating a new instance of that schema
const kitty = new Cat({ name: 'Zildjian' });

```

[[8]](#8-mongoose-documentation)

---

##### Mongoose

A simple example with mongoose might look as the following.

```js
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/test', {useNewUrlParser: true});

// Defining the data structure, schema
let catSchema = new mongoose.Schema(
{
    name: String
});

//  Defining a model with the schema
const Cat = mongoose.model('Cat', catSchema);

// Creating a new instance of that schema
const kitty = new Cat({ name: 'Zildjian' });

// Saving the instance to the database
kitty.save();
```


---

##### Mongoose

Some query examples:

--

```js
// Find a specific entry:
Cat.find((err, dataDb) => {
    ...
};
```

--

```js
// Find all entries based on the condition name: 'fluffy'
Cat.find({ name: 'fluff' }, (err, dataDb) => {
    ...
};
```

--

```js
// Find one entry with that condition
Cat.findOne({ name: 'fluff' }, (err, dataDb) => {
    ...
};
```

[[8]](#8-mongoose-documentation)

---

### Non-Relational Database

#### MongoDB

#### mLab

In order to access the data of a MongoDB online, we need a service that hosts our database. For this we are going to use mLab.

[mLab](https://mlab.com) provides databases as a service for MongoDB. They are a commercial service but have certain capabilities for free. These are sufficient for our purposes.

[[7]](#7-tutorialspoint-mongodb-tutorial) [[8]](#8-mongoose-documentation)

???

Databases and database systems are an ongoing development and research topic in Computer Science. If you study Computer Science you most likely have at least one lecture solely dedicated to databases.

We will not get into databases but we just use one in it simplest and most straight-forward implementation. However, databases are actually a fascinating topic and if you are ever needing a more sophisticated setup I encourage you to investigate the topic further.

---

### Connect to the Database

Add the following code at the beginning of your server file in order to connect to your mongoDB Atlas collection.

```js
// server.js
...
// TODO 10a
const mongoose = require('mongoose');

// DATABASE 
// TODO 10b
const dbUrl = `mongodb+srv://<user>:<password>
                @cluster0-hgrys.mongodb.net/
                <collection>?retryWrites=true&w=majority`;

// Connecting
mongoose.connect(dbUrl,
                {useNewUrlParser: true, useUnifiedTopology: true},
                (err) =>
{
    if (err) return console.log('Error:', err);
    else console.log('Connected to:', dbUrl);
});
```

---

### The Schema of the Magnets

--

We identify a magnet with its index and save the location of the magnet.

--

```js
// server.js

// TODO 11
let MagnetSchema = new mongoose.Schema(
{
    index: Number,
    x: Number,
    y: Number,
});

let Magnet = mongoose.model('magnet', MagnetSchema);
```



[[10]](#10-mongoose-documentation-models)

---

## The Fridge Poetry App - Implementation

To work with the database with the magnets we need to consider two scenarios:

???

TASK: Which ones?

--

1. The website is called for the first time and there are no magnets saved yet in the database (this happens only once)

--

2. The magnet exists in the database

---

### Initialization of the Magnets

We create a new communication exchange, which either

* initializes the database when a client is connected and ready or
* sends the current position of all magnets to the newly connected client.

---

Check if the client is done with its setup:

```js
// fridgepoetry.js

function setup()
{
    ...

    // TODO 12
    socket.emit('clientSetupReady');
}
```

---

Then, the general logic for this differentiating wether the database needs to be initialized looks as follows.

```js
// server.js

io.on('connection', socket =>
{

...

    // TODO 13
    socket.on('clientSetupReady', () =>
    {
        console.log('Client ready');
















    });
...

```

---

Then, the general logic for this differentiating wether the database needs to be initialized looks as follows.

```js
// server.js

io.on('connection', socket =>
{

...

    // TODO 13
    socket.on('clientSetupReady', () =>
    {
        console.log('Client ready');

        // Get all data from the Magnet model collection
        Magnet.find((err, dataDb) =>
        {











        });
    });
...

```
---

Then, the general logic for this differentiating wether the database needs to be initialized looks as follows.

```js
// server.js

io.on('connection', socket =>
{

...

    // TODO 13
    socket.on('clientSetupReady', () =>
    {
        console.log('Client ready');

        // Get all data from the Magnet model collection
        Magnet.find((err, dataDb) =>
        {


            if(dataDb.length == 0)
            {


            }
            else
            {


            }
        });
    });
...

```

---

Then, the general logic for this differentiating wether the database needs to be initialized looks as follows.

```js
// server.js

io.on('connection', socket =>
{

...

    // TODO 13
    socket.on('clientSetupReady', () =>
    {
        console.log('Client ready');

        // Get all data from the Magnet model collection
        Magnet.find((err, dataDb) =>
        {
            if (err) return console.error(err);

            if(dataDb.length == 0)
            {
                console.log('Init Database');
                //...
            }
            else
            {
                console.log('Init Client');
                //...
            }
        });
    });
...

```

???


---

#### Initialization of the Database

If there is no data in the database yet, we want to save the current random state of the client.

--

```js
// server.js

// TODO 14
console.log('Init Database');
socket.emit('serverAsksForMagnetData');
```

---

#### Initialization of the Database

```js
// fridgepoetry.js

function setup() 
{
    ...

    // TODO 15
    socket.on('serverAsksForMagnetData', () =>
    {
        console.log('Sending all magnet data');
        ...
    });
}
```

---

#### Initialization of the Database

```js
// fridgepoetry.js

function setup() 
{
    ...

    // TODO 15
    socket.on('serverAsksForMagnetData', () =>
    {
        console.log('Sending all magnet data');

        for (let i = 0; i < poetry.magnets.length; i++)
        {
            let data =
            {
                index: poetry.magnets[i].index,
                x: poetry.magnets[i].x,
                y: poetry.magnets[i].y
            }
            ...
        }
    });
}
```

---

#### Initialization of the Database

```js
// fridgepoetry.js

function setup() 
{
    ...

    // TODO 15
    socket.on('serverAsksForMagnetData', () =>
    {
        console.log('Sending all magnet data');

        for (let i = 0; i < poetry.magnets.length; i++)
        {
            let data =
            {
                index: poetry.magnets[i].index,
                x: poetry.magnets[i].x,
                y: poetry.magnets[i].y
            }
            // Make use of the already existing pipe
            socket.emit('clientMagnetMove', data);
        }
    });
}
```


---

#### Initialization of the Database

Now let's update the `clientMagnetMove` function for either an empty database or for updating a specific magnet data entry.


---

#### Initialization OR Updating of the Database


```js
// server.js

    // Receive moving signal from a client
    socket.on('clientMagnetMove', (data) =>
    {
        ...

        // TODO 16
        // Update the position in the database
        // Find the magnet by its index
        Magnet.findOne({index:Number(data.index)},
                        (err, dataDb) =>
        {
            if (err) return console.error(err);

            ...
        });
    });
```

---

#### Initialization OR Updating of the Database


```js
// server.js

    // Receive moving signal from a client
    socket.on('clientMagnetMove', (data) =>
    {
        ...

        // TODO 16
        // Update the position in the database
        // Find the magnet by its index
        Magnet.findOne({index:Number(data.index)},
                        (err, dataDb) =>
        {
            if (err) return console.error(err);

            if(dataDb !== null) // update
            {
                dataDb.x = data.x;
                dataDb.y = data.y;
                dataDb.save();
            }
            ...
        });
    });
```

---

```js
// server.js

    // Receive moving signal from a client
    socket.on('clientMagnetMove', (data) =>
    {
        ...

        // TODO 16
        // Update the position in the database
        // Find the magnet by its index
        Magnet.findOne({index:Number(data.index)},
                        (err, dataDb) =>
        {
            if (err) return console.error(err);

            if(dataDb !== null) // update
            {
                dataDb.x = data.x;
                dataDb.y = data.y;
                dataDb.save();
            }
            else // initialize
            {
                let tmpMagnet = new Magnet(data);
                tmpMagnet.save((err, element) =>
                {
                    if (err) return console.error(err);
                    console.log('New element saved', element);
                });
            }
        });
    });
```

---

#### Initialization of a Client

For the scenario that the magnets already exist in the databse, we want to sent position of all magnets saved in the database to a newly connected client.

```js
// server.js
...

    // TODO 17
    console.log('Init Client');
    socket.emit('serverSendsDbData', dataDb);
...
```

---

#### Initialization of a Client

```js
// fridgepoetry.js

function setup() 
{
    ...

    // TODO 18
    // Receive signal from server that
    // it sending all magnet data
    // from the database
    socket.on('serverSendsDbData', (data) =>
    {
        ...
    });
```

---

#### Initialization of a Client

```js
// fridgepoetry.js

function setup() 
{
    ...

    // TODO 18
    // Receive signal from server that
    // it sending all magnet data
    // from the database
    socket.on('serverSendsDbData', (data) =>
    {
        for (let i = 0; i < data.length; i++) 
        {
            ...
        }
    });
```

---

#### Initialization of a Client

```js
// fridgepoetry.js

function setup() 
{
    ...

    // TODO 18
    // Receive signal from server that
    // it sending all magnet data
    // from the database
    socket.on('serverSendsDbData', (data) =>
    {
        for (let i = 0; i < data.length; i++) 
        {
            // We can't be sure that data array from the
            // database has the same order as the magnets
            // array. We need to identify each element
            // by its index.
            let index = fridge.magnets.findIndex(obj => 
            {
                return obj.index === data[i].index
            });

            fridge.magnets[index].index = data[i].index;
            fridge.magnets[index].x = data[i].x;
            fridge.magnets[index].y = data[i].y;
        }
    });
```

???

Javascript array findIndex() is an inbuilt function that returns an index of the first item in an array that satisfies the provided callback function. Otherwise, it returns -1, indicating no element passed the test.  The findIndex() method executes a function once for each item present in an array.

---
template:inverse

## Goal 2 ✔️ 🎉

???

https://www.ted.com/talks/simone_giertz_why_you_should_make_useless_things
https://www.youtube.com/channel/UC3KEoMzNz8eYnwBC34RaKCQ/videos
https://www.youtube.com/watch?v=ab47XHidvwQ&t=259s
https://www.ted.com/talks/lucy_cooke_sloths_the_strange_life_of_the_world_s_slowest_mammal


---

## Online Deployment

To discuss all the options and requirements for hosting a website professionally is out of scope of this class. 

We will simply use the free *hobby* hosting option of the [Heroku](https://www.heroku.com/what) platform.

---

### Heroku

Heroku is a cloud platform as a service supporting several programming languages including Java, Node.js, Scala, Clojure, Python, PHP, and Go.  

For this reason, Heroku is said to be a *polyglot* platform as it lets the developer build, run and scale applications in a similar manner across all the languages.

--

Heroku works nicely with Git - you can deploy web applications by pushing your code to a Heroku/git repo.

---

### Heroku

#### [Setup](https://devcenter.heroku.com/articles/getting-started-with-nodejs#set-up)

**Sign up on the [Heroku site](https://www.heroku.com/).**

---

### Heroku

#### [Setup](https://devcenter.heroku.com/articles/getting-started-with-nodejs#set-up)

Once you have created a user login, you need to configure you system once to work with Heroku correctly.

--

* Have git installed and setup (which all of you already have)

--

* Download the [Installer](https://devcenter.heroku.com/articles/getting-started-with-nodejs#set-up) for Heroku with Node.js

???

Mac folks: think about using [Homebrew](https://brew.sh/), which I have mentioned in the Setup Chapter already

--

* Execute in the terminal / command line `heroku login`

--

    * This command opens your web browser to the Heroku login page. If your browser is already logged in to Heroku, simply click the Log in button displayed on the page.
    * This authentication is required for the heroku and git commands to work correctly.


---

### Heroku

#### Deployment

There are two steps for deploying an app with heroku:

--

1. Initialize and commit its local git repository

--

2. Initialize and commit its online heroku repository


---

### Heroku

##### Git

* Navigate with the terminal into your `fridgePoetryClass` folder  
    (REMEMBER THAT YOU CAN NOT BE INSIDE AN GIT ENVIRONMENT ALREADY)

* `git init`
* `git add .`
    * If you are sure that the folder is clean and you want to commit all files!  
  
* `git commit -m "Initial Commit"`


---

### Heroku

##### Heroku

* Create a heroku app with: `heroku create` or `heroku create <your_name>`

--
    * `heroku create` will give your app a random default name
    * If you want to define the name yourself consider that <your_name> must start with a letter, end with a letter or digit and can only contain lowercase

--

* Now you can deploy your code with: `git push heroku master`

--

* `heroku open` opens the app, which can be found under the url `https:/<your_name>.herokuapp.com/`

In the heroku world, e.g. on its dashboard, an app is called *dyno*. 

---

### Heroku

#### Workflow

--

If you made changes to the code and you want to update the online heroku app

1. `add` and `commit` to update the git repository of the app
2. `git push heroku master` to update the heroku online repository of the app

--

Please note that for the free hobby deployment of the app, the app falls asleep after 30min of inactivity and waking it up again (loading its webpage) might take a moment (usually less than a minute).


---
template:inverse

## Goal 3 ✔️  🎉

---
template:inverse

## The Fridge Poetry App ✔️  👏🏻  👏🏻  👏🏻 

???

https://www.ted.com/talks/simone_giertz_why_you_should_make_useless_things
https://www.youtube.com/channel/UC3KEoMzNz8eYnwBC34RaKCQ/videos
https://www.youtube.com/watch?v=ab47XHidvwQ&t=259s
https://www.ted.com/talks/lucy_cooke_sloths_the_strange_life_of_the_world_s_slowest_mammal





---
template:inverse

## The Story Telling App

---

## The Story Telling App

### Why?

???

TASK: What are we missing?

--

So far we only used *one* page. 

???
and still relied on p5

--

Now we want to 

--

* have multiple, nested pages, and

--

* use modern and pure javascript.

---

## The Story Telling App

The app will be an implementation of a *Reizwortgeschichte*.

--

This means that a user is given three words, which then must be used in a story.

???

TASK: Show App

---

## The Story Telling App

The app needs to

--

* Save a variety of words (*Reizwörter*) in a database
    * Give the option to enter new words

--

* Show three randomly chosen words from the database

--

* Give the option to enter and save a story (text) 

--

* Show a randomly chosen story from the database

---

## The Story Telling App

The app has three main pages:

--

1. The startpage (also called landing page), which shows a story and the words the story is using.

--

2. A page to add a story based on three randomly chosen words.

--

3. A page to add a new word to the database.

--

We save in a mongo database

* the words, and
* the stories.


---

## The Story Telling App

### Implementation

For the implementation we will make further use of the Express Web Framework and its functionalities.

--

The underlying logic of the steps is in different frameworks (such as Python's Django) more or less the same.

???

First, I am going to explain all the theory and then we apply it for the story telling app.

---

## The Story Telling App

### Implementation

Express provides methods to 

???

Remember:
Skript: In a traditional data-driven website, a web application waits for HTTP requests from the web browser (or other client). When a request is received the application works out what action is needed based on the URL pattern and possibly associated information contained in POST data or GET data. Depending on what is required it may then read or write information from a database or perform other tasks required to satisfy the request. The application will then return a response to the web browser, often dynamically creating an HTML page for the browser to display by inserting the retrieved data into placeholders in an HTML template.  
https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction

--

* how to process the different HTTP request methods, which are called *verbs* (GET, POST, SET, etc.)

--

* use *URL patterns*, which are called *routes*
    * `localhost:5000/stories/add`
    * `localhost:5000/stories/all`

--

* use *templates* to render html pages, which are called *views*


???

Skript: Express provides methods to specify what function is called for a particular HTTP verb (GET, POST, SET, etc.) and URL pattern ("Route"), and methods to specify what template ("view") engine is used, where template files are located, and what template to use to render a response. You can use Express middleware to add support for cookies, sessions, and users, getting POST/GET parameters, etc. You can use any database mechanism supported by Node (Express does not define any database-related behaviour).  
https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction

---
template:inverse

## HTTP Request Methods

---

## The Story Telling App

### HTTP request methods

Request methods indicate which action should be performed for a given client request. 

???

Skript: HTTP defines a set of request methods to indicate the desired action to be performed for a given resource. Although they can also be nouns, these request methods are sometimes referred as HTTP verbs.  
https://developer.mozilla.org/de/docs/Web/HTTP/Methods

--

The most important methods are

* `GET`
    * Used to request data from a specified resource, it always should only *retrieve* data.

--

* `POST`
    * Used to send data to a server to create/update a resource.

???

TASK: What is the difference to WebSockets?  

WebSockets enable the server and client to send messages to each other at any time, after a connection is established, without an explicit request by one or the other. This is in contrast to HTTP, which is traditionally associated with the challenge-response principle — where to get data one has to explicitly request it. In more technical terms, WebSockets enable a full-duplex connection between the client and the server.  

https://blog.stanko.io/do-you-really-need-websockets-343aed40aa9b

Information exchange mode of WebSocket is bidirectional. Means, server can push information to the client (which does not allow direct HTTP). HTTP communication can only be initiated by a client (is this true?)


---

### HTTP request methods

#### GET

```js
//app_example01.js

let express = require('express');
let app = express();

// A route definition
app.get('/', (req, res) => res.send('Hello World!'));

// Have the app listen to port 5000
const port = process.env.PORT || 5000;
app.listen(port, () => console.log(`Listening on ${port}`));
```

--

The `app.get()` method specifies a callback function that will be invoked whenever there is an HTTP GET request with a path `('/')` relative to the site root. 

--

The callback function always takes a request and a response object as arguments.

--

[`send()`](https://expressjs.com/en/4x/api.html#res.send) simply returns the string "Hello World!".

???

* An express object is typically called app

https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction


---
template:inverse

## Routes

---

## The Story Telling App

### Routes

Routes allow you to match particular patterns of characters in a URL to different functionalities, called *route handlers*.

```js
//app_example02.js

let express = require('express');
let app = express();

// A route definition
app.get('/', (req, res) => res.send('My Landing Page'));
app.get('/about', (req, res) => res.send('My About Page'));

// Have the app listen to port 5000
const port = process.env.PORT || 5000;
app.listen(port, () => console.log(`Listening on ${port}`));
```


---

## The Story Telling App

### Routes

Routes are typically defined in a separate file for each main route (the prefix of that route).

--

For example, we might have 

* `localhost:5000/stories` (the main route)
* `localhost:5000/stories/add` (sub-route)
* `localhost:5000/stories/all` (sub-route)

These routes we collect in a file `stories.js` in a `routes` folder.

---

## The Story Telling App

### Routes

```
.
├── app.js
└── routes
    └── stories.js

```

--

All routes we define in `routes/stories.js` will be in regard to an url with the filename, meaning `localhost:5000/stories`:

* `/`
    * (the stories main route)
    * Refers to `localhost:5000/stories`
* `/add`
    * (sub-route)
    * Refers to `localhost:5000/stories/add`
* `/all`
    * (sub-route)
    * Refers to `localhost:5000/stories/all`

---

## The Story Telling App

### Routes

#### Modules

Whenever we create separate files in JavaScript, these separate files are *modules*, with the name of the file. In this case we are create the module `stories`. We have to use certain functionalities to work with modules such as using `require()`.

--

For a module when have to specify what should be accessible, when requiring that module. We do so with exporting it, which is usually an object.

```js
module.exports = 'what ever it is we want to export';
```

---

## The Story Telling App

### Routes

```js
//stories.js

let express = require('express');
let router = express.Router();

// All routes under /stories
router.get('/', (req, res) => res.send('Stories Page'));
router.get('/all', (req, res) => res.send('All Stories Page'));
router.get('/add', (req, res) => res.send('Add Stories Page'));

// Defining what should be accessible when
// requiring the module
module.exports = router;
```

For creating routes, we often use the `express.Router` object directly (instead of importing the whole module).

---

## The Story Telling App

### Routes

Now that we have created a separate route file, we still need to use it in our main `app`.

```js
//app_example03.js

let express = require('express');
let app = express();

// Direct route definitions
app.get('/', (req, res) => res.send('My Landing Page'));
app.get('/about', (req, res) => res.send('My About Page'));

// Route modules
app.use('/stories', require('./routes/stories'));

// Have the app listen to port 5000
const port = process.env.PORT || 5000;
app.listen(port, () => console.log(`Listening on ${port}`));
```


---

### HTTP request methods

#### POST

--

We need some data generation, e.g. with a html form:

```html
<form action="/stories/add" method="POST">
    <textarea rows="10" cols="60" name="story" id="story" placeholder="Tell us your story..."></textarea>

    <input type="submit" value="Add">

</form>
```

Upon `submit` the form data is sent to `/stories/add` via `POST`. We need to define a route for that.

---

### HTTP request methods

#### POST

```js
//stories.js

router.post('/add', (req, res) => {

    let story = req.body.story;

    ...
});
```

---

### HTTP request methods

#### POST

```js
//stories.js

router.post('/add', (req, res) => {

    let story = req.body.story;

    let tmp = new Story({ story });

    tmp.save((err, element) => {

        if (err) return console.error(err);

        console.log('New element saved', element);
    });

    res.redirect('/stories/add');
});
```

--

For `let story = req.body.story;` to work we must add the bodyParser module to the app:

```js
//app.js
app.use(bodyParser.urlencoded({ extended: false }));
```

???

Example coming later once we know how to build html pages


---
template:inverse

## Views

---

## The Story Telling App

### Views

For rendering html site (*views*) we can use *template engines* or also called *view engines*. 

--

Templates allow you to specify the *structure* of an output document in a template, using *placeholders for data* that will be filled in when a page is generated. 

--

In our context, we use templates to create HTML, but template can also create other types of documents. 


---

## The Story Telling App

### Views

Express has support for a [number of template engines](https://github.com/expressjs/express/wiki#template-engines).

--

One of the most popular ones is [pug](https://pugjs.org/api/getting-started.html). Pug is very powerful but also fairly complex.
![pug](https://camo.githubusercontent.com/a43de8ca816e78b1c2666f7696f449b2eeddbeca/68747470733a2f2f63646e2e7261776769742e636f6d2f7075676a732f7075672d6c6f676f2f656563343336636565386664396431373236643738333963626539396431663639343639326330632f5356472f7075672d66696e616c2d6c6f676f2d5f2d636f6c6f75722d3132382e737667)


???

Skript: Template engines (referred to as "view engines" by Express) allow you to specify the structure of an output document in a template, using placeholders for data that will be filled in when a page is generated. Templates are often used to create HTML, but can also create other types of documents. Express has support for a number of template engines, and there is a useful comparison of the more popular engines here: Comparing JavaScript Templating Engines: Jade, Mustache, Dust and More.  
https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction

---

## The Story Telling App

### Views

Express has support for a [number of template engines](https://github.com/expressjs/express/wiki#template-engines).  

One of the most popular ones is [pug](https://pugjs.org/api/getting-started.html). Pug is very powerful but also fairly complex.  

We are going to use [express-handlebars](), 

--

> A Handlebars view engine for Express which doesn't suck.

--

which I find a good balance between functionality and complexity.

---

## The Story Telling App

### Views

To use the handlebars template engine we add to app.js:

```js
//app_example04.js
...

let exphbs  = require('express-handlebars');

...

// Set the template engine
app.engine('handlebars', exphbs());
app.set('view engine', 'handlebars');

app.get('/', (req, res) => res.render('index');

...
```


---

## The Story Telling App

### Views

```js
//app_example04.js

let express = require('express');
let exphbs  = require('express-handlebars');

let app = express();

// Set the template engine
app.engine('handlebars', exphbs());
app.set('view engine', 'handlebars');

// Direct route definitions
app.get('/', (req, res) => res.render('index'));
app.get('/about', (req, res) => res.send('My About Page'));

// Route modules
app.use('/stories', require('./routes/stories'));

// Have the app listen to port 5000
const port = process.env.PORT || 5000;
app.listen(port, () => console.log(`Listening on ${port}`));
```

---

## The Story Telling App

### Views

Now, we need to work with the following project structure:

```
.
├── app.js
└── routes
    └── stories.js
└── views
    ├── index.handlebars
    └── layouts
        └── main.handlebars

```

--

With

--

* `views/layouts/main.handlebars` as HTML wrapper page

--

* `views/index.handlebars` as HTML content for the landing page

---

## The Story Telling App

### Views

#### `views/layouts/main.handlebars`

The main layout is the HTML page wrapper, which can be reused for the different views of the app. `{{{body}}}` is used as a placeholder for where the content of a view will be rendered.

```html
{{!-- views/layouts/main.handlebars --}}
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Example App</title>
</head>
<body>

    {{{body}}}

</body>
</html>
```

---

## The Story Telling App

### Views

#### `views/index.handlebars`

The content for the app's index view which will be rendered into the main layout's {{{body}}}.

```html
{{!-- views/index.handlebars --}}

<h1>My Landing Page</h1>

```

---

## The Story Telling App

### Views

With template engines we can use placeholders (like variables) in the template, which only upon rendering will be filled with actual data send with the request.

--

```js
//app.js

app.get('/', (req, res) => res.render('index', { title: 'About dogs', message: 'Dogs rock!' }));
```

```html
{{!-- views/index.handlebars --}}

<h1>{{title}}</h1>

<p>{{message}}</p>

```

---

## The Story Telling App

### Views

Now we can add the html form to a view and have a look at the POST example again.


---

## The Story Telling App

The usage of view is also related to the [Model-View-Controller](https://developer.mozilla.org/en-US/docs/Glossary/MVC) program architecture.

--

The three parts of the MVC software-design pattern can be described as follows:

1. Model: defines what data the app should contain.
2. View: defines how the app's data should be displayed.
3. Controller: contains logic that updates the model and/or view in response to input from the users of the app.

---

## The Story Telling App

The MVC model when working with express, routes and views would implement:

--

* *Routes* to forward the supported requests (and any information encoded in request URLs) to the appropriate controller functions.

???

Routes map a URL to a controller, which is the action. 

--

* *Controller* functions to get the requested data from the models, create an HTML page displaying the data, and return it to the user to view in the browser.

--

* *Views* (templates) used by the controllers to render the data.

---

## The Story Telling App

![mvc_express](/img/09/mvc_express.png)[[MDN Express Tutorial]](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/routes)

---

## The Story Telling App

When working with Express however, we usually do not create separate controller files but only work with the routes.

--

This is possible as Express makes no assumptions in terms of structure or what components you use, as it is *unopinionated*.


Routes, views, static files, and other application-specific logic can live in any number of files with any directory structure.

---

## The Story Telling App

There is for example the option to use the [Express Application Generator](https://expressjs.com/en/starter/generator.html), which creates a modular app skeleton for you like the following:

```
.
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.pug
    ├── index.pug
    └── layout.pug

```

Be aware that these files are filled with all kinds of base code and rather cluttered (I usually don't use it).

---

## The Story Telling App

### Models

To separate models for database collections is strongly advised however.

--

You should create one models file in a `/models` folder for each mongoose model, e.g.

--

`models/Story.js`  
(by convention written with a capital letter)  

--

```
.
├── app.js
└── models
    └── Story.js
└── routes
    └── stories.js
└── views
    ├── index.handlebars
    └── layouts
        └── main.handlebars

```


---

## The Story Telling App

### Models

```js
let mongoose = require('mongoose');

let Schema = mongoose.Schema;

let StorySchema = new Schema({
    story: {type: String, required: true},
});

// Export model
module.exports = mongoose.model('Story', StorySchema);
```

---

## The Story Telling App

### Models

Then you can work with that model e.g. in a route file `word.js` with

```js
...

const Word = require('../models/Story');

router.get('/', (req, res) => {

    Story.find((err, dataDb) => {

        if (err) return console.error(err);

        res.render('words', { stories: dataDb });
    });
});
```

---
template:inverse

## Now we know all the theory for the story telling app.

# 🤯

