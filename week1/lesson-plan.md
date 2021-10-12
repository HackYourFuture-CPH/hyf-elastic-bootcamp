# Week 1 : Topic APM

## Pre-requisites:

Ensure to follow the [Preparation](./preparation.md) guideline and have the example app installed and running. 

## Lesson Plan:

We are going to instrument the shared example app with Elastic APM agents and take a look at the collected data and how to use them. 

1. Add APM instrumentation to the backend application

    Add `elastic-apm-node` to the backend app to enable Elastic APM auto-instrumentation. 

  * Add following to the very top of `movie-backend/src/index.js`:
    ```
    const apm = require('elastic-apm-node').start({
      // Use if APM Server requires a token
      secretToken: '<your-apm-server-secret-token',

      // Set custom APM Server URL (default: http://localhost:8200)
      serverUrl: '<your-apm-server-url>',
    })
    ```
  * Make a couple of requests to the backend application, for example by using `curl` or `postman`. 

    The available routes are:
    * `GET /genres`

      Example Request: `curl http://localhost:3001/genres`

    * `GET /directors`

      Example Request: `curl http://localhost:3001/directors`

    * `GET /movies`

      query string parameters: `artist=<artist.name>` (replace `artist.name` with the artist to query for)

      Example Requests: 
      * `curl "http://localhost:3001/movies?genre=documentary"`
      * `curl "http://localhost:3001/movies?genre=comedy"`
      * `curl "http://localhost:3001/movies?genre=comdey&genre=documentary"`
      * `curl "http://localhost:3001/movies?director=Emilia%20Huber&director=Anthony%20De%20Place&genre=documentary"`

  * Navigate to the APM UI in Kibana (_Observability/APM_). You should now see the service `movie-backend`. Familiarize yourself with the APM UI.

2. Solve the N+1 problem
   
    Navigate to `movie-backend/src/db.js` and comment/uncomment the `fetchMovie` function with the second, more performant one. 

3. Add manual Instrumentation

* Error capturing: 
  Uncaught errors and exceptions are automatically captured by Elastic APM. Capturing handled errors must be done manually.
  Add `apm.captureError(err)` in the `catch` block in the `movies` router in `movie-backend/src/api/movies.js`:
  ```
  const apm = require('elastic-apm-node')

  router.get("/", async (request, response, next) => {
    apm.setTransactionName("GET /movies")
    try {
      ...
    } catch (err) {
      if (err instanceof db.MovieNotFoundError) {
        apm.captureError(err)
        return response.json([])
      }
      next(err)
    }
  });
  ```
  Make a couple of request leading to an empty result. For instance call `http://localhost:3001/movies?genre=Action`.

  Navigate to the `movie-backend` service in the APM UI and checkout the _Errors_ section. 

* Add labels

  Navigate to `movie-backend/src/api/movies.js` and add `apm.SetLabel` as follows:
  ```
  router.get("/", async (request, response, next) => {
    apm.setTransactionName("GET /movies")
    try {
      // only allow one genre to be requested
      if (Array.isArray(request.query.genre)) {
        throw new ParamsNotAllowedError("Only one genre allowed")
      }
      // fake a server side error for demo purposes
      if (request.query.genre && request.query.genre === "fail") {
        throw `Server temporarily not available`
      }
      if (request.query.genre) {
        apm.setLabel("genre", request.query.genre.toLowerCase())
      }
      const movies = await db.fetchMovie(request.query);
      return response.json(movies)
    } catch (err) {
      ...
    }
  });
  ```

  Go to Kibana/Dev Tools and run following query:
  ```
  GET apm-*-transaction/_search
  {
    "aggs": {
      "artist_type_queries": {
        "terms": {
          "field": "labels.genre",
          "size": 20
        }
      }
    },
    "size": 0
  }
  ```

4. Add APM instrumentation to the frontend application 

    Add `@elastic/apm-rum` to the frontend app to enable Elastic APM auto-instrumentation. 

  * Add following to the `movie-frontend/src/index.js`:
    ```

    import { init as initApm } from '@elastic/apm-rum'

    let apm = initApm({
      // Set required service name (allowed characters: a-z, A-Z, 0-9, -, _, and space)
      serviceName: 'movie-frontend',

      // Set custom APM Server URL (default: http://localhost:8200)
      serverUrl: '<apm-server-url>',

      // Set service version (required for sourcemap feature)
      serviceVersion: 'v0.1.0',

      breakdownMetrics: true
    })

    // group development transactions into one bucket 
    // to avoid spamming the APM UI in development mode
    apm.observe('transaction:end', function (transaction) {
      if (transaction.name.endsWith(".json")) {
        transaction.name = "development"
      }
    })
    ```

  * Wrap the react components with apm:
    * `movie-frontend/src/components/MovieList.js`

      Add import statement: 
      ```
      import { withTransaction } from '@elastic/apm-rum-react'
      ```

      Wrap export with transcation:
      ```
      export default withTransaction('MovieList', "component")(MovieList);
      ```
    Do the same for the other components:
    * `movie-frontend/src/components/Genre.js`
    * `movie-frontend/src/components/Director.js`
    * `movie-frontend/src/components/Movie.js`

    NOTE: change `MovieList` in the above `export` statement to the according component name.

  * Navigate to the browser, and click around.
  * Navigate to the services APM UI in Kibana (_Observability/APM_). You should now also see the `movie-frontend`. Familiarize yourself with it.
  * Navigate to _Observability/APM/Traces_. Choose the trace `Click - div` and investigate the trace sample. You should now see a sample showing a distributed trace. 
  * Navigate to the User Experience Dashboard in Kibana (_Observability/User Experience_). Familiarize yourself with it.

  NOTE: You can also add manual instrumentation to the react app using the `@elastic/apm-rum` agent. 

## Advanced Topics
* Agent Configuration Management for Sampling
* Sourcemaps
* Annotations (add significant events, such as deployments)

## See final solution with Elastic APM
To see the final solution including the apm instrumentation, checkout the `apm` branch (`git checkout apm`) from [https://github.com/simitt/apm-movie-demo-app](https://github.com/simitt/apm-movie-demo-app).

## Elastic Documentation
* [Get Started with the APM app](https://www.elastic.co/guide/en/kibana/current/apm-getting-started.html)
* [APM Overview](https://www.elastic.co/guide/en/apm/get-started/current/index.html)
* [APM Agents](https://www.elastic.co/guide/en/apm/agent/index.html)
  * [APM Node.js Agent Reference](https://www.elastic.co/guide/en/apm/agent/nodejs/current/index.html)
  * [APM RUM JavaScript Agent Reference](https://www.elastic.co/guide/en/apm/agent/rum-js/current/index.html)
* [APM Server](https://www.elastic.co/guide/en/apm/server/current/index.html)
* [User Experience App](https://www.elastic.co/guide/en/observability/current/user-experience.html)