 # Preparation
In this week's workshop we will demonstrate how to ingest logs from the existing Node.js movie application into your Elasticsearch Service deployment using Filebeat. 

# Prerequisites
In this Exercise you will use the deployment credentials you downloaded during the Intro Workshop. 

In case you don't have the credentials, you should create a new Deployment and download the file in a safe place.

 # Setup
As mentioned above we will continue working on the existing movie app, but we will clone a new repository into a new folder. The new repository already contains the fixed solution from the apm workshop.
 
Please install the application and have it running before the workshop, by following the instructions below:  

1. Clone the application
    ```
    git clone https://github.com/mgiota/apm-movie-demo-app.git logs-movie-demo-app
    cd logs-movie-demo-app
    ```

2. Open the new `logs-movie-demo-app` into your Editor

3. Make sure to update `movie-backend/src/db.js` with the existing mysql user and password:
    ```
    const knex = require('knex')({
        client: 'mysql',
        connection: {
            host: '127.0.0.1',
            port: 3306,
            user: '<mysql-user>', // replace with mysql user
            password: '<mysql-password>', // replace with mysql user password
            database: 'moviedemo',
        }
    });
    ```
3. Make sure to update `movie-backend/src/index.js` with your APM Server secret token and server url.

        ```
        apm.start({
        // Override service name from package.json
        // Allowed characters: a-z, A-Z, 0-9, -, _, and space
        serviceName: '',

        // Use if APM Server requires a token
        secretToken: '<your-apm-server-secret-token>',

        // Set custom APM Server URL (default: http://localhost:8200)
        serverUrl: '<your-apm-server-url>',
        })
        ```

4. Start the backend service in development mode. Don't forget to stop the previously running nodejs app from the apm workshop
    ```
    cd movie-backend
    nvm use
    npm install
    npm run dev
    ```

5. Ensure that the backend application is reachable via http request by making a request to `http://localhost:3001/genres`

If you get stuck no worries! We will start the demo by briefly recapping above steps together. If you have any questions before the workshop, feel free to post them in the Slack channel.

# Resources

* [ELK Stack for Log Management](https://www.linkedin.com/pulse/guide-use-elastic-stackelk-stack-log-management-dharmik-joshi/)
* [Filebeat overview](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-overview.html)
* [Filebeat modules](https://www.youtube.com/watch?v=K-jVrLMOd-g)


