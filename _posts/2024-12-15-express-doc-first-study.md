# Express

This is my notes from the official doc

[Ref](https://expressjs.com/)

## What is Express?

Node.js web app framework (open source MIT)

## Installing

Req:
- Express 4.x requires Node.js 0.10 or higher
- Express 5.x requires Node.js 18 or higher

Initialize Node.js package manager
```bash
npm init -y
```

Use npm to install express as dependency
```bash
npm install express
```

Default entry point is `app.js` in root project directory

## Simplest example

Edit `app.js`, instance express and use it's abilities

```js
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})
```

Reponds 404 Not Found for non-existent paths

Note, `req` and `res` are Node.js's objects

Use Node.js to run `app.js`
```bash
node app.js
```

## Basic routing

AKA define endpoints (URI + HTTP request method)

This is the declaration template
```js
app.METHOD(PATH, HANDLER)
```

Note: HANDLER is a function with `req` and `res` parameters

Example
```js
app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.post('/', (req, res) => {
  res.send('Got a POST request')
})

app.put('/user', (req, res) => {
  res.send('Got a PUT request at /user')
})

app.delete('/user', (req, res) => {
  res.send('Got a DELETE request at /user')
})
```

## Doc has collection of example repos

Use this to learn how to use Express. Has things like
- Auth
- Cookies
- So much more...

[Ref](https://expressjs.com/en/starter/examples.html)
