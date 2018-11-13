# Exploring Server-Side Development with Twilio

## Description

In this lab, you'll have a chance to build a few tools using Twilio: a service that supports SMS texting and voice, among other things,
over the Internet.

Twilio's main use is to allow web applications to incorporate texting or automated voice call capabilities. This can include, for example,
two-factor authentication or interactive chatbot-type systems over SMS.

## Create an Account

Go to the Twilio home page and sign up for an account. You'll need to enter your own phone number and receive a validation code to prove
that you're a real person.

Twilio's service is not free, but the trial account gets you about $15 of credit. Sending a text costs, like, $.0075, and getting an activated phone number is $1, so you'll be able to send close to 2000 texts on the trial, which should be more than enough for us. Another
practical limitation of the intro trial: you're only allowed to text your own phone number that you used in the account setup process.

## Get Your Number

First, create a new project. From your logged-in console page, choose "Products", then select "Programmable SMS" and "Phone Numbers". Click on the "Continue" button at the bottom of the page to create the workspace. You can choose any name for the workspace; I made mine "CMS 450 Demo".

Once you're on the main project dashboard, scroll down and click on "Get a Number" under the "Phone Numbers" section. You'll be offered a number (I think it's in the same area code as your sign-up number by default), which you can accept.

You're now ready to write some code.

## Send a Text

You're going to write a nodejs backend.

Log-in to Cloud 9, navigate to your CMS450 directory (or create it with `mkdir` if you don't have one). Make a new project directory:

```
mkdir twilio_lab
cd twilio_lab
```

Install the Twilio node package. If you don't have node on your Cloud 9 account, look back at the earlier lab and go through the setup instructions for it.

```
npm install twilio --save
```

Now create a new file

```
touch send_text.js
```

Put the following code into `send_text.js`:

```
// Twilio Credentials
const accountSid = 'YOUR ACCOUNT SID FROM YOUR DASHBOARD GOES HERE';
const authToken = 'YOUR AUTH TOKEN FROM YOUR DASHBOARD GOES HERE';

// Require the Twilio module and create a REST client
const client = require('twilio')(accountSid, authToken);

client.messages.create(
  {
    to: '+1xxxxxxxxxx',
    from: '+1yyyyyyyyyy',
    body: 'BEEP BOOP THIS IS A COMPUTER TEXTING YOU HUMAN',
  },
  (err, message) => {
    console.log(message.sid);
  }
);
```

There are four things you have to fill in:

1. Your account SID string, which is on the upper right of your dashboard.

2. You account authentication token, which is in the same place.

3. Replace the `x`'s with your personal phone number (the one you used in the account creation process). Keep the leading `+1` in place.

4. Replace the `y`'s with the number Twilio generated for you. Again, keep the leading `+1`.

The code is pretty simple: it uses the `twilio` package to create a `client`, which has a `create` method that does the work of contacting Twilio's servers for you. The first input to `create` is a JS object specifying the phone numbers involved in the text and the message. The second input is a function (written in the anonymous arrow style) that runs when the request completes.

Run the script at the command prompt:

```
node send_text.js
```

Change the message and run it again.

## Texting II: The Human Texts Back

Okay, now we're going to set up a server that can both receive and respond to text requests.

Here's how this works:

- When a user texts the number associated with your Twilio app, Twilio will receive that text.

- It then automatically fires off an HTTP request to a server URL that you specify for handling incoming requests. We'll set this up
in the next section.

- The server's text-handling URL is bound to a bit of JavaScript code that runs when the server receives the HTTP request. The code can
access the content of the message and send a text back to the user, using the same Twilio API calls as the last example.

### Create the Server App

First, create a new node project within your `twilio_lab` directory.

```
npm init
```

You can accept all of the default options except the `entry_point` name: change that to `app.js`, like in our previous projects.

Install express:

```
npm install express --save
```

And create the `app.js` file:

```
touch app.js
```

### The Code

Put the following code in `app.js`:

```
var http = require('http');
var express = require('express');
var twilio = require('twilio');

var app = express();

app.post('/sms', function(req, res) {
  console.log('Received message');

  const MessagingResponse = require('twilio').twiml.MessagingResponse;
  const response = new MessagingResponse();
  response.message('GREETINGS HUMAN PLEASED TO MEET YOU');

  res.writeHead(200, {'Content-Type': 'text/xml'});
  res.end(response.toString());
});

http.createServer(app).listen(8080, function () {
  console.log("Express server listening on port 8080");
});
```

Let's break this down.

The first three lines import the required libraries. The fourth,

```
var app = express()
```

initializes an application using the `express` framework.

The next block of code is the most important.

```
app.post('/sms', function(req, res) {
```

This line declares a URL named `/sms` that can receive requests via the HTTP `POST` method, which is used to send data to a server.
When the server receives a valid HTTP request targeting the `/sms` path, it runs the specified function.

**THIS NEXT PART IS IMPORTANT**.

These associations between HTTP URLs and functions are called **routes**. The idea of routes is core to RESTful web programming.

- When a user wants to do something like retrieve a record, create an account, delete some content, or invoke a function, the client-side HTML page sends a request to the appropriate URL on the server.

- The server receives the HTTP request and invokes the associated function, which does some work to satisfy the user's request.

If you look back at our previous projects, you'll see a line like,

```
app.get('/', (req, res) => res.sendFile(path.join(__dirname, '/index.html')));
```

This line creates a route for the `/` URL (the root of the page). The associated function sends back the root `index.html` page.

### Tell Twilio About Your Server

Go to your project dashboard and navigate to the phone numbers page. Click on the one active number (the one you got from Twilio during
the setup phase) and navigate down to "Messaging".

In the field that says "A MESSAGE COMES IN" put

```
https://YOURWORKSPACENAME-YOURUSERNAME.c9users.io:8080/sms
```

Make sure that you have the `/sms` bit at the end: that's the route you want Twilio to target.

### Fire It Up

Go up to the "Share" tab in your Cloud 9 editor. Verify that the "Public" box is checked next to your Application URL. This will allow
Twilio to send requests your server without requiring it to log in (which it can't do). I think this is checked by default if you're
using a public Cloud 9 workspace.

Run the server.

```
node app.js
```

Once your server is up, text your Twilio number. It will text you back.

You can modify the code to collect some extra information about the incoming message. The `req` parameter of the function contains
the body of the HTTP POST message. To access it, first install the `body-parser` package:

```
npm install body-parser --save
```

Next, add the following lines **after** you define `app = express()`.

```
var bodyParser = require('body-parser');
app.use(bodyParser.json()); // support json encoded bodies
app.use(bodyParser.urlencoded({ extended: true })); // support encoded bodies
```

Recall that JSON is "JavaScript Object Notation". It's a way of representing JS objects (which are key-value maps), in text form.

Add the following lines at the beginning of the `/sms` route function:

```
  console.log('From: ' + req.body.From);
  console.log('Body: ' + req.body.Body);
```

`req.body` is the JS object holding the contents of the HTTP POST message. `From` is the number of the incoming message and `Body` is
the message contents.

## The Voice

Let's play another trick by creating a route that can receive and respond to voice calls.

Go into `app.js` and add a new route for `/voice`. Copy the function from the `/sms` route, then change the messaging lines in the
middle to this:

```
  const VoiceResponse = require('twilio').twiml.VoiceResponse;
  const response = new VoiceResponse();
  response.say('Hello, human!', { voice: 'alice' });
```

The code is almost identical to the previous example, but will now generate a text-to-speech synthesized voice response. Make sure to
keep the code at the end of the function that sends back the HTML response.

Next, go back into your Twilio phone number settings and update the field for "A CALL COMES IN" to use your application URL with the new `/voice` route.

Stop your server with CTRL-C if it's still running from the previous example. Save your changes and re-run the server 
(changes don't take effect unless your stop and re-run the server).

Call your Twilio phone number. You'll hear a short message about having a trial account, then be prompted to press a button. Press
any digit and then welcome your new machine overlord.

## Mongo and Mongoose

You've built a server that can receive and respond to text messages. However, you application has no **persistence**: there is no
way to save data between requests. Therefore, the final part of this lab will add database support to the server.

Our example database application will log all SMS texts sent to your Twilio phone number. We'll use two tools, MongoDB and Mongoose,
to build it.

MongoDB is an "open-source document database". It belongs to a class of what are sometimes called **NoSQL databases**.

You may recall that many classic databases, like MySQL, Postgres, and Oracle, make use of the **relational data model**. In a relational
DB, the data is conceptually modeled as a collection of 2-D tables, where each row in the table is a thing and the columns represent the
attributes of a thing.

```
        columns are the attributes of a thing
 -----------------------------------------------------
|                                                     |
v                                                     v
=======================================================
|   ID    |   LastName    |  FirstName   |  EMailAddr |  
=======================================================
|         |               |              |            |  <--- each row is one thing
-------------------------------------------------------
|         |               |              |            |
-------------------------------------------------------
```

SQL, the "Structured Query Language", is a programming syntax for coding lookups and searches in a relational database.

"NoSQL" databases got their name because, originally, they did not focus on supporting the same classic use cases as the relational model. Rather than using 2-D tables, NoSQL databases store data as looser collections of (key, value) pairs, kind of like a big
hash table.

Given that (key, value) pair objects are an integral part of JavaScript, the ability to quickly set up a database that could easily
store and retrieve JS objects in JSON or a similar format without the complexity of a traditional DB quickly caught on in the web
programming world.

Since that time, the term **document database** has also caught on for similar applications.

There are many NoSQL databases out there. We'll use MongoDB, which is one of the most popular. We'll pair it with another tool called
Mongoose, which provides a nicer object-oriented front-end to the lower-level MongoDB database operations.

This section will be pretty bare-bones, but it will equip you with enough working knowledge to think about building more sophistiated 
data-driven applications, and to further explore the MongoDB and Mongoose documentation.

### Install MongoDB

The first step is to install MongoDB, then set up a script that runs it with some necessary parameters. To install, use our old buddy
`sudo apt-get install`:

```
sudo apt-get install -y mongodb-org
```

Next, enter the following commands. The first makes a directory called `data` that will serve as the database location. The second 
copies a command into the a script named `mongod`. The command includes some setting flags, notably `$IP`, the IP address of your
current server instance. The `--nojournal` flag stops MongoDB from allocating a 2 GB journal file, which would blow up your disk quota. The last line uses `chmod` to adjust the permissions of the script.

```
mkdir data
echo 'mongod --bind_ip=$IP --dbpath=data --nojournal --rest "$@"' > mongod
chmod a+x mongod
```

Finally, run the `mongodb` process, which will start up and begin listening for connections on port 27017.

```
./mongod
```

You now have a running database.

### The Mongoose Strikes

Install the `mongoose` module using `npm`:

```
npm install mongoose --save
```

Now add some code to connect Mongoose to the running MongoDB instance. Place this **before** your `/sms` route to keep things organized:

```
// Mongoose connection to MongoDB
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/test');

var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function() {
  console.log('Mongoose connected to DB.');
});
```

Start your server like normal:

```
node app.js
```

You should see a message saying `Mongoose connected to DB`. You may get a warning about a URL strin parser; that's okay and you can ignore it.

### Mongoose Workflow

**JavaScript objects** are the **basic units of data storage** in a Mongo + Mongoose database. To store some data, you first package it 
into an object, then invoke some appropriate functions to save the object in the DB.

As you'll recall, JS objects are fundamentally (key, value) maps that associate names with values. Therefore, you can view a 
Mongo + Mongoose data item as a "blob" of named data properties.

However, you can't just chuck any object you want into the DB (at least, we're not going to let you do that--there are **rules**). You must first define a **schema** that specifies what names and their types you'd like a data object to contain.

**Step 1: Create the Schema**  Modify your Mongoose setup code to the following. This code will create a data model for saving
texts in the DB.

```
// Mongoose connection to MongoDB
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/test');

var Text;

var db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function() {
  console.log('Mongoose connected to DB.');
});


// Define a model for data objects
// from: the input number
// body: the body of the test message
var textSchema = new mongoose.Schema({
  from: String,
  body: String
});

// Compile the schema to create a model constructor
var Text = mongoose.model('Text', textSchema);
```

The last line takes the `Schema` and compiles it to a special function that Mongoose calls a **model**. The model function is a
constructor for DB objects.

**Step 2: Update the `/sms` route to store data objects**  Modify the code for `/sms` to construct and save a data object with the contents of every text:

```
app.post('/sms', function(req, res) {
  console.log('From: ' + req.body.From);
  console.log('Body: ' + req.body.Body);

  // Log the message in the DB
  var newText = new Text({ from: req.body.From,
                           body: req.body.Body});

  newText.save(function (err, newText) {
    if (err) return console.error(err);
    console.log('Record saved to DB.')
  });

  // Construct and send SMS response
  const MessagingResponse = require('twilio').twiml.MessagingResponse;
  const response = new MessagingResponse();
  response.message('Thank you. Your message has been recorded.');

  res.writeHead(200, {'Content-Type': 'text/xml'});
  res.end(response.toString());
});
```

The important new functions are:

- `new Text`, which uses the model constructor to create a new data object. Notice that the input to
the constructor is a JS object (enclosed in `{}`) that specifies the names and their properties.

- `save`, which actually invokes the back-end DB operations to save an object in the database. Notice that `save` is a **method** of
`newText`.

Try running the server again and sending a few texts. You will see messages saying that they've been saved to the DB, but you can't see
the contents of the DB, which leads to...

**Step 3: Write a route to view the contents of the DB**

Add one more route to your project, which will dump all of the DB contents to the console:

```
app.get('/data', function(req, res) {

  // Print out all messages in DB
  Text.find(function (err, texts) {
    if (err) return console.error(err);
    console.log(texts);
  });

  // Send back a generic response
  res.writeHead(200, {'Content-Type': 'text/xml'});
  res.end("OK");

});
```

A few points:

- This code uses the HTTP GET method, specified by `app.get`.

- The important function to look up all values in the DB is `find`. It returns a list of all data objects generated by the `Text` model.

- When the `find` completes it runs the specified callback function. The results of the call are saved in the array `texts`.

- The last lines send a generic response message back to the client. It could be more interesting if the response actually contained the
data from the query, eh?

Fire up another terminal window to test your implementation:

```
curl https://YOURWORKSPACENAME-YOURUSERNAME.c9users.io:8080/data
```

The `curl` command sends a basic HTTP GET request to the specified route, which triggers the code to print the DB contents to the 
console.

## Next

This lab has given you a high-level overview of routes, interacting with third-party APIs, and storing data in a NoSQL database.

A few final thoughts:

- Reading documentation and searching for examples are essential when doing this kind of work.

- At the same time, be prepared for the possiblity that the docs you find might be wrong or out of date. I had to fix multiple errors
that were in the official Twilio documentation to get their example code to run, due to the doc still using an out-of-date version of
their own API.

- There's one piece that we're still missing: sending a request **from a client page** to the server, getting some data back, and using
that data to update the client page.

