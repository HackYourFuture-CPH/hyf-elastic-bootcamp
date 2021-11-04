# Week  3: Topic Logs
This guide demonstrates how to `ingest logs` from the movie Node.js  application you built during the APM workshop and deliver them  into your Elasticsearch deployment using `Filebeat`.

You are going to learn how to:
1. use `winston logger` to send logging events based on HTTP requests in a custom file
2. use a Node.js HTTP request application to generate traffic to your application
3. setup Filebeat
4. send Node.js logs to Elasticsearch
5. create log visualizations in Kibana



## Pre-requisites:
Ensure to follow the [Preparation](./preparation.md) guideline and have the example app installed and running.

Even if you didn't complete the exercise from the APM workshop, you will still be able to follow along with this Exercise, since you will clone a new repository that already contains the fixed apm solution. 

If you don't have the deployment credentials from your Intro Workshop, you can reset the `elastic` user password [here](https://www.elastic.co/guide/en/cloud/current/ec-password-reset.html#ec-password-reset). Keep the deployment credentials somewhere safe, because we are going to use them later on.

## Lesson Plan:

###  1. Add logging to your nodejs application

Navigate to the movie-backend folder (if you are not already there) and install a couple of dependencies.

```
cd movie-backend
npm install winston @elastic/ecs-winston-format got
```

Under movie-backend/src folder create a `utils` folder with a `logger.js` file inside it and save it with the contents found [here](https://raw.githubusercontent.com/mgiota/apm-movie-demo-app/logs/movie-backend/src/utils/logger.js).

 
Open `index.js` and add following lines of code after line 17 where you defined the port number:

```
const logger = require('./utils/logger').logger;
const expressRequestLogger = require('./utils/logger').expressRequestLogger;
const expressErrorLogger = require('./utils/logger').expressErrorLogger;
app.use(expressRequestLogger({ logger }))
```

 The error logger needs to be added AFTER the express routers and BEFORE any of your custom error handlers. You should add following code before the errorHandler function:
```
app.use('/error', function(req, res, next) {
  // here we cause an error in the pipeline.
  return next(new Error("This is an error"));
});

// Express error logger makes sense after the router
app.use(expressErrorLogger({ logger }));
```

Let's test what we've done so far: 
* Open your browser and navigate to `http://localhost:3001/movies`. You should see a `logs.json` file under a newly created logs folder. 
* Open your `db.js` file and change the password to a wrong one. Refresh your browser and verify that a new line with the message `error handling GET /error` will appear in your logs.json file. Don't forget to undo the change to the db.js file.


### 2. Create a Node.js HTTP request application

In this step, you’ll create a Node.js application that sends HTTP requests to your web server.

Under movie-backend/src folder, create `_webrequests.js` and save it with the contents found [here](https://raw.githubusercontent.com/mgiota/apm-movie-demo-app/logs/movie-backend/src/_webrequests.js).

This Node.js app generates HTTP random requests to the different endpoints (movies, genres, directors and error) and a random from request header using various pretend email addresses. The requests are sent at random intervals between 1 and 10 seconds.

In a new terminal make sure you are in the movie-backend folder and run:

```
node src/_webrequests.js
``` 

After the script has run for about 30 seconds, enter CTRL + C to stop it. Have a look at your Node.js logs/logs.json file. It should contain some entries written in a JSON format with ECS fields. This allows for easy parsing and analysis, and for standardization with other applications. 


### 3. Set up Filebeat

- Download [Filebeat](https://www.elastic.co/downloads/beats/filebeat) and unpack it in a folder under your documents. 
- Open the `filebeat.yml` configuration file for editing
- In the Elastic Cloud section remove the comment pound sign (#) from the lines cloud.id: and cloud.auth:
- For cloud.auth, add the username and password of your deployment (you should have downloaded a credentials file during the Intro Workshop)
- For cloud.id, add the deployment’s Cloud ID
- In the `filebeat.inputs` section, set `enabled: true` and set the path to the logs.json file of your app
- Add the following json decoding options:

```
  json.keys_under_root: true
  json.overwrite_keys: true
  json.add_error_key: true
  json.expand_keys: true
```

Here's how the filebeat.inputs option should look like:
```
filebeat.inputs:

# Each - is an input. Most options can be set at the input level, so
# you can use different inputs for various configurations.
# Below are the input specific configurations.

- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /Users/panagiotamitsopoulou/Documents/HYF/apm-movie-demo-app/movie-backend/logs/logs.json
    #- c:\programdata\elasticsearch\logs\*
  json.keys_under_root: true
  json.overwrite_keys: true
  json.add_error_key: true
  json.expand_keys: true
  ```

- Run following commands to test your configuration:
```
./filebeat test config
./filebeat test output
```

- Run the setup command to load predefined assets for parsing, indexing and visualizing your data in Kibana Dashboards:
```
./filebeat setup
```

The setup process takes a couple of minutes. If everything goes successfully you should see a confirmation message:

``` 
Loaded Ingest pipelines
```


### 4. Send the Node.js logs to Elasticsearch

It's time to send some logs into Elasticsearch!

Launch Filebeat by running following command from the Filebeat installation directory:

```
./filebeat -e
```

Let's verify that you can see the logs in Kibana:

**Logs app**
1. From the Kibana main menu click on the `Logs` option under Observability section
2. If you don't see any logs, update the time range to `Last 1 hour`
3. You can run again your node.js HTTP request application to generate more data

**Discover**
1. From the Kibana main menu click on the `Discover` option under Analytics section
2. On the dropdown menu on the left change the index pattern to `filebeat-*`
3. You should see the logs of your application


### 5. Create log visualizations in Kibana

Now it's time to create visualizations based on the logs data.

1. Open the Kibana main menu, click Dashboard and then Create dashboard.
2. Click Create visualization. The Lens visualization editor opens.
3. In the index pattern dropdown box, select filebeat-*, if it isn’t already selected.
4. In the CHART TYPE dropdown box, select Bar vertical stacked, if it isn’t already selected.
5. Check that the time filter is set to Last 1 hour.
6. From the Available fields list, drag and drop the `@timestamp` field onto the visualization builder.
7. Drag and drop the `http.request.headers.from` field onto the visualization builder.
8. A stacked bar chart now shows the relative frequency of each of the different urls used in our example, measured over time.
9. Click Save and return to add this visualization to your dashboard.

Let's create one more visualization.
1. Click Create visualization. The Lens visualization editor opens.
2. In the CHART TYPE dropdown box, select Donut.
3. From the list of available fields, drag and drop the `http.request.headers.from` field onto the visualization builder. A donut chart appears.
4. Click Save and return to add this visualization to your dashboard.
5. Click Save and add a title to save your new dashboard.

You now have a Kibana dashboard with two visualizations: a stacked bar chart showing the frequency of each HTTP request url over time and another donut chart showing the frequency of various HTTP from headers over time.

Feel free to play around and use different chart types or different field names in your charts.  

You now know how to monitor log files from a Node.js web application, deliver the log event data securely into an Elasticsearch Service deployment, and then visualize the results in Kibana in real time.

## See final solution
To see the final solution including the logs instrumentation, checkout the `logs` branch (`git checkout logs`) from [https://github.com/mgiota/apm-movie-demo-app](https://github.com/mgiota/apm-movie-demo-app).

## Elastic Documentation
* [Ingest Logs from Nodejs app using Filebeat](https://www.elastic.co/guide/en/cloud/current/ec-getting-started-search-use-cases-node-logs.html) 
* [ECS Logging with winston](https://www.elastic.co/guide/en/ecs-logging/nodejs/current/winston.html)
* [Integration with APM tracing](https://www.elastic.co/guide/en/ecs-logging/nodejs/current/winston.html?baymax=rec&rogue=rec-1&elektra=guide#winston-apm)
* [Everything you Always Wanted to Know about Filebeat](https://www.youtube.com/watch?v=ykuw1piMGa4)
* [Filebeat quickstart: installation and configuration](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation-configuration.html)
