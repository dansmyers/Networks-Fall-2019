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

- The server's text handling URL is bound a bit of JavaScript code that runs when the server receives the HTTP request. The code can
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

In the field that says "A MESSAGE COME IN" put

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








