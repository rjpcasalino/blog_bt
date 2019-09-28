---
layout: post
title: Docker, Go, and Heroku
---

Deploying a simple Gin Go app turned out to be surprisingly annoying today. I'm not sure if I was looking in the wrong
place for the correct documentation or if I'm just an aging fool. Either way, I thought it wise to commit to
writing what I did. Perhaps some wayward soul will come across this post in a few odd years when none of it will
matter anyway.

Heroku binds to a random port each time your app spins up. Make your Go service look for an env var named `PORT`.
The pitfall here to is the colon before the port number. Thus: `":" + os.Getenv("PORT")`.

Log into Heroku via their CLI tool and the documentation gets clearer. Although, they do forget to mention to pass
the app name you're looking to deploy.
```
heroku container:push web --app <appname>
```

next:

```
heroku container:release web --app <appname>
```
