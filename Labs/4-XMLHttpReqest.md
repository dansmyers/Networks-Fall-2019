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

