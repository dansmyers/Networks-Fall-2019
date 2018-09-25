# Meme Generator

## Overview

In this lab, you're going to build a meme generator using front-end web development techniques.

The goal of this lab is **not** to memorize every single feature and function that's used to build the app. Rather, we want to emphasize
a few key points:

- The core technique of giving elements `id` values and accessing their properties using `document.getElementById()`.

- Laying out a page using Bootstrap's grid system.

- Using Bootstrap's built-in features for input boxes, buttons, and radio toggles.

- Assigning functions to run on button clicks and other events.

- Using a new element, `canvas`, for drawing.

Everything you'll create today will be **pure front-end HTML, CSS, and JavaScript**. All of the code fits into about 100 lines on a single
page. Therefore, the page can respond to user input, but doesn't have the ability to save data between session. Our next topic will be
adding in a back-end framework that can store and retrieve data in response to browser requests.

The lab should take about an hour. Once you finish you can make some memes and experiment with adding new features to the application.

## Initialization

Create a new directory and a new node project.

```
$ cd cms450/front_end
$ mkdir meme_gen
$ cd meme_gen
$ npm init

Go through the npm init process
Remember to change the entry point name from index.js to app.js

$ npm install express --save
$ cp ../hello_app/app.js .
$ touch index.html
```

Open `index.html` and you're ready to start coding. Here's a shell Bootstrap page. Copy and paste it into your `index.html`.

```
<!doctype html>
<html>
    
    <head>
        
        <!-- Required meta tags -->
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Bootstrap CSS -->
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
        
    </head>
    
    
    <body>

        <!-- Recall: all Bootstrap content must be enclosed in a container div -->
        <div class="container">
            
            <!-- Container is made of a series of rows -->
            <div class="row">
                
                <!-- Each row is divided into 12 units -->
                <!-- col-md-12 makes a column that spans all 12 units -->
                <div class="col-md-12">
                    OHAI DERE.
                </div>   <!-- /col-md-12 -->  
                
            </div> <!-- /row -->
            
        </div> <!-- /container -->
        
        <!-- Optional JavaScript -->
        <!-- jQuery first, then Popper.js, then Bootstrap JS -->
        <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
    </body>
    
</html>
```

Save the file, then run your node server.

```
$ node app.js
```

Visit your workspace's page, whcih is available by right-clicking (or CTRL + click on Mac) on your workspace name and choosing Info.

## Bootstrap Overview

Recall some basic facts about the Bootstrap grid system.

- Grid elements are created by making `div` tags and giving them `class` properties that describe the layout. Boostrap has a class
for almost everything you want to do.

- All Bootstrap grid elements must be enclosed in a `container` div. The container automatically pulls the content in from the left
and right edges a little bit.

- Within the container, the grid is organized into a series of `div` sections with `class="row"`.

- Each row is divided into 12 columns that span the full width of the container. You can group these 12 units into columns by creating
additional `div` elements within the `row`. In this example, the `col-md-12` class makes a column that spans all 12 units of its row. You can subdivide the 12 units in any way you want, such 6-6, 4-4-4, 3-3-3-3, 8-4, 4-8, 11-1, etc., but **the units of all columns in 
the row must sum up to 12**.

- The `md` portion of the column class name controls the device size at which Bootstrap's grid stacking behavior takes effect (it can also be `sm`, `lg`, or `xl`). This is not important for us, but lets designers have more control over how the page appears on different-
sized mobile devices.

Let's modify the grid system. Change the single `col-md-12` `div` into two `div`s, each with `class=col-md-6`. This will create two
equal-width columns spanning the row. Put some text in each one and refresh your page.
