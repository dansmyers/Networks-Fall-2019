# Using XMLHttpRequest

## Description

We spent the last lab writing a server that established multiple routes, with each route representing the listening point of a service
that the application could provide to clients.

What we haven't done, however, is allow our front-end web pages to contact those routes. This lab will show you to makes requests and pass 
data between the front-end and back-end of an application using `XMLHttpRequest`, the vanilla JavaScript feature for HTTP communication.

## Setup the Server and Page

Create a new directory (the name doesn't matter) and create a new node project:

```
npm init

(accept default names except for changing index.js to app.js)

npm install express --save
npm install body-parser --save (we'll use this later)
```

Create a basic `app.js` file with the following code:

```
const express = require('express');
const app = express();
const port = 8080;
var path = require('path');
app.use(express.static('public'));

// Return root page
app.get('/', (req, res) => res.sendFile(path.join(__dirname, '/index.html')));

// Run server
app.listen(port, () => console.log(`Example app listening on port ${port}!`));
```

This is the basic server we've used several times: it has one route that returns `index.html` when the browser requests `/`.

Now load up `index.html`:

```
<!DOCTYPE html>
<html>

    <head>

    </head>

    <body>
        <h1>Example</h1>


        <input id="input_box" type="text"> </input>

        <button id="button">Press Here</button>

        <div id="output_div"></div>

        <script>
            document.getElementById('button').onclick = function() {
                alert('Clicked!');
            }
        </script>

    </body>
</html>
```

The page has an input box and a button. Clicking the button fires off a little JavaScript that displays a message.

Run the server, load the page, and test the button.

## Client to Server

Create a new route on the server:

```
// Receive a request from the server
app.post('/submit', function(req, res) {

    console.log("Request received: " + parameter);

    var data = {message: 'hello, world!'};

    res.setHeader('Content-Type', 'application/json');
    res.json(data);
});
```

This route is tied to the URL `/submit`. It uses the HTTP `POST` method.

The most important part is what it returns: 

- `data` is a JS object with one field (`message`)

- The final two lines convert that object into a text-based representation that can be the body of an HTTP response

This style of sending JS objects as HTTP text is called *JavaScript Object Notation* (JSON), and it's a preferred way of passing
information between clients and servers on the modern web.

Now add some code on the client to send an `XMLHttpRequest`. Replace the existing script block with the following:

```
        <script>
            document.getElementById('button').onclick = function() {
            
                var textbox = document.getElementById('input_box');
                var text = textbox.value;
                var data = {parameter: text};

                // Send the request to the server
                var xmlhttp = new XMLHttpRequest();

                xmlhttp.onload = function() {
                    alert("Response received!");

                    var data = JSON.parse(this.responseText);
                    console.log(data);
                };

                xmlhttp.open("POST", "/submit");
                xmlhttp.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
                xmlhttp.send(JSON.stringify(data));
            }
        </script>
```

## Server Response

Here's the final version of the server code:

```
// Global count variable
var count = 1;

// Receive a request from the server
app.post('/submit', function(req, res) {

    var parameter = req.body.parameter;
    console.log("Request received: " + parameter);

    var data = {message: 'hello, world!', count: count};
    count++;

    res.setHeader('Content-Type', 'application/json');
    res.json(data);
});
```
