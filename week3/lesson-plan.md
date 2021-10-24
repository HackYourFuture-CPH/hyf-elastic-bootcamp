# Week  3: Topic Logs
This guide demonstrates how to `ingest logs` from the movie Node.js  application you built during the APM workshop and deliver them  into your Elasticsearch deployment `using Filebeat`.

You are going to learn how to:
1. use `winston logger` to send logging events based on HTTP requests in a custom file
2. use a Node.js HTTP request application to generate traffic to your application
3. setup Filebeat
4. send the Node.js logs to Elasticsearch
5. create log visualizations in Kibana



## Pre-requisites:
Ensure to follow the [Preparation](./preparation.md) guideline and have the example app installed and running.

Even if you didn't complete the exercise from the APM workshop, you will still be able to follow along with this Exercise, since you will clone a new repository that already contains the fixed apm solution. 

## Lesson Plan:

1\. Add logging to your nodejs application

Navigate to the movie-backend folder and install a couple of dependencies.

```
cd movie-backend
npm install winston express-winston @elastic/ecs-winston-format got
```

Under movie-backend create a `utils` folder with a `logger.js` file inside it and save it with the contents found [here](https://raw.githubusercontent.com/mgiota/apm-movie-demo-app/logs/movie-backend/src/logger.js).

 
Open `index.js` to make use of the logger we just created. Add following code after the line where port number is defined:

```
const logger = require('./logger').logger;
const errorLogger = require('./logger').errorLogger;
app.use(logger);
```

You will also create a middleware that logs the errors of the pipeline. The error logger needs to be added AFTER the express routers and BEFORE any of your custom error handlers (express.handler). You should add following code after the line you connect apm middleware.
```
app.use('/error', function(req, res, next) {
  // here we cause an error in the pipeline so we see express-winston in action.
  return next(new Error("This is an error and it should be logged to the console"));
});

// express-winston errorLogger makes sense AFTER the router.
app.use(errorLogger);
```

Let's test what we've done so far with a a couple http requests. 
* Open your browser and navigate to `http://localhost:3001/movies`. You should see a `logs.json` file under a newly created logs folder. 
* Open your `db.js` file and change the password to a wrong one. Refresh your browser and verify that a new error will appear in your logs.json file. Don't forget to undo the change to the db.js file.


2\. Create a Node.js HTTP request application

In this step, you’ll create a Node.js application that sends HTTP requests to your web server.

Under movie-backend/src folder, create a file `_webrequests.js` and paste the contents of this [file](https://raw.githubusercontent.com/mgiota/apm-movie-demo-app/logs/movie-backend/src/_webrequests.js).

This Node.js app generates HTTP requests with a random method of type GET, POST, or PUT, and a random from request header using various pretend email addresses. The requests are sent at random intervals between 1 and 10 seconds.

In a new terminal run `node _webrequests.js`. 

After the script has run for about 30 seconds, enter CTRL + C to stop it. Have a look at your Node.js logs/logs.json file. It should contain some entries written in a JSON format with ECS fields. This allows for easy parsing and analysis, and for standardization with other applications. 

3\. Set up Filebeat


4\. Send the Node.js logs to Elasticsearch


5\. Create log visualizations in Kibana

## Advanced Topics
* [Integration with APM tracing](https://www.elastic.co/guide/en/ecs-logging/nodejs/current/winston.html?baymax=rec&rogue=rec-1&elektra=guide#winston-apm)

## See final solution
To see the final solution including the logs instrumentation, checkout the `logs` branch (`git checkout logs`) from [https://github.com/mgiota/apm-movie-demo-app](https://github.com/mgiota/apm-movie-demo-app).

## Elastic Documentation
* [ECS Logging with winston](https://www.elastic.co/guide/en/ecs-logging/nodejs/current/winston.html)
* [Everything you Always Wanted to Know about Filebeat](https://www.youtube.com/watch?v=ykuw1piMGa4)
* [Filebeat quickstart: installation and configuration](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html)
