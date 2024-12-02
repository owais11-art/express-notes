# express-notes

## Initialize node project
```
npm init -y
```

## Install Express
```
npm install express
```

## Create app and listen to port
```js
import express from 'express'

const app = express() // Creating an app

app.listen(3000, () => console.log("Server running in port 3000")) // starting server at port: 3000
```

## Starting of server

In terminal:

```
node [filename].js
```

## Auto restarting of server

In terminal:

```
node --watch [fileman].js
```
To make it more simple in `package.json` in `dev` add:
```js
"scripts": {
    "dev": "node --watch [filename].js"
}
```

then in terminal:

```
npm run dev
```

## Environment variables
Create .env file and add variables like this:
```
PORT = 8080
```
To use it(node version >= 20.6.0):
```js
const port = process.env.PORT || 3000
```
But for it to work we have to add `--env-file=.env` flag in dev script in `package.json`
```js
"scripts": {
    "dev": "node --watch --env-file=.env [filename].js"
}
```
To use it(node version < 20.6.0):

install `dotenv` package:
```
npm install dotenv
```
then

```js
require('dotenv').config({path='.env'})
const port = process.env.PORT || 3000
```

## Creating routes

```js
let posts = [
    {
        id: 1,
        title: "Post 1"
    },
    {
        id: 2,
        title: "Post 2"
    },
    {
        id: 3,
        title: "Post 3"
    }
]

app.get('/api', (req, res) => {
    res.json(posts)
})
```

### Dynamic routes

```js
app.get('/api/post/:id', (req, res) => {
    const id = parseInt(req.params.id) // Getting dynamic parameters using req.params objects
    res.json(posts.find(post => post.id === id))
})
```

To get queries use `req.query` object

To send specific status codes:

```js
app.get('/api/post/:id', (req, res) => {
    const id = parseInt(req.params.id) // Getting dynamic parameters using req.params objects
    res.status(200).json(posts.find(post => post.id === id))
})
```

### Using routes in different file

create folder `routes` and create `posts.js` in it

In `posts.js`

```js
const express = require('express')

const router = express.Router()

let posts = [
    {
        id: 1,
        title: "Post 1"
    },
    {
        id: 2,
        title: "Post 2"
    },
    {
        id: 3,
        title: "Post 3"
    }
]

router.get('/', (req, res) => {
    const limit = req.query.limit
    if(!isNaN(limit) && limit > 0){
        res.json(posts.slice(0, limit))
    } else res.json(posts)
})

router.get('/:id', (req, res) => {
    const id = parseInt(req.params.id)
    const post = posts.find(post => post.id === id)
    if(!post) res.status(404).json({message: "Post not found"})
    else res.json(post)
})

module.exports = router
```

In `main.js`

```js
const express = require('express')
const posts = require('./routes/posts') // Import posts
require('dotenv').config({path: '.env'})

const app = express()
const port = process.env.PORT || 3000

app.use('/api/posts', posts) // use posts

app.listen(port, () => console.log(`Server is running on port ${port}`))
```

## Making POST request

```js
router.post('/', (req, res) => {
    const body = req.body
    posts.push(body)
    res.status(201).json({message: "New Post Created"})
})
```

To parse body JSON use `express.json()` middleware in `main.js`

```js
app.use(express.json())
```

## Making PUT request

```js
router.put('/:id', (req, res) => {
    const id = parseInt(req.params.id)
    posts = posts.map(post => {
        if(post.id === id){
            return {
                id: parseInt(req.body.id),
                title: req.body.title
            }
        }
        return post
    })
    res.json(posts)
})
```

## Making DELETE request

```js
router.delete('/:id', (req, res) => {
    const id = parseInt(req.params.id)
    posts = posts.filter(post => post.id !== id)
    res.json(posts)
})
```

## Middlewares

Functions that are executed after request is made and before response in sent.

```js
function logger(req, res, next){
    /*
        Your code here...
    */
    next() // for moving to next middleware in stack
}

router.get('/', logger, (req, res) => {
    /*
        Your code here...
    */
})
```

To use middlewares at app level in `main.js`

```js
function logger(req, res, next){
    /*
        Your code here...
    */
    next() // for moving to next middleware in stack
}

app.use(logger) // will be executed for all routes
```

## Error Handling

In `middlewares/errorHandler.js`: 

```js
function errorHandler(err, req, res, next) {
    if(err.status) {
        res.status(err.status).json({
            message: err.message
        })
    }else {
        res.status(500).json({
            message: err.message
        })
    }
}

module.exports = errorHandler
```

To use it `main.js`

```js
/*
    your other imports  here...
*/
const errorHandler = require('./middlewares/errorHandler') // import error handler function.

/*
    your code here...
*/

app.use('/api/posts', posts)
app.use(errorHandler) // use it after routes

/*
    your code here...
*/
```

In routes:
```js
router.get('/:id', (req, res, next) => {
    /*
        your code here...
    */

   if(!post){
    const error = new Error("No post found")
    error.status = 404
    return next(error)
   }

   /*
        your code here...
    */
})
```
