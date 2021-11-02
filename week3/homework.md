# Homework

## Exercise 1 - Create Visualizations
In this Exercise you will create a new vertical bar chart, which visualizes the count of records broken down by the HTTP response code:

- Create a chart of type `Bar vertical`
- Drag and drop the `@timestamp` field onto the visualization builder
- Drag and drop the `http.response.status_code` field onto the `Break down by` area
- Save the new Visualization into your Dashboard

## Exercise 2 - Enable MySQL Filebeat module [Optional]

We slightly touched the concept of Filebeat modules in this workshop, so this Exercise is optional. 

However, if you want to know how to enable MySQL Filebeat module to parse and collect MySQL logs, go ahead with the Exercise.


- From the Filebeat installation directory you should run
```
./filebeat modules enable mysql
```

- To figure out the path of mysql logs in your OS, connect to a MySQL client and type following commands:

```
SHOW VARIABLES LIKE '%general_log%';  
SHOW VARIABLES LIKE '%slow_query_log%'; 
```

If general_log and slow_query_log appear to be OFF, you should enable them by running:

```
SET GLOBAL general_log = 'ON';  
SET GLOBAL slow_query_log = 'ON'; 
```

- Modify the settings in `modules.d/mysql.yml` with the paths you got in previous step. Make sure to remove the comment pound sign (#) from the lines where var.paths is defined. Please note that the value of the var.paths variable should be an array. Here's an example:

```
var.paths: ['/usr/local/var/mysql/*.log']
```


- Run filebeat 
```
./filebeat -e
```

- In your MySQL client make a couple of SQL queries. For example you can connect to your database and make a couple of select statements. Or you can run `SHOW VARIABLES LIKE '%general_log%';` a few more times to generate a couple of logs

- From the main Kibana menu click on `Discover`

- Search for the `event.dataset` field name and click on the plus icon to add this field as column

- Search for the `message` field and click on the plus icon to add this field as column

- Verify that you can see mysql logs


