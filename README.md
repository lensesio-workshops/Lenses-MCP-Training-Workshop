# Lenses MCP Training Workshop Hands-On Labs
<!-- v. 3 -->

## Lab Outline

Lab 1: Familiarize Yourself With Lenses UI

Lab 2: Connect your AI with Lenses MCP

Lab 3: Explore Data With Your LLM

Lab 4: Write SQL Queries With Your LLM

Lab 5: Build a Poison Pill Filter With Your LLM

### Lab 1: Familiarize Yourself With Lenses UI

Step 1: Log in and explore. Your instructor will have assigned you a Lenses URL with login information. Go to that URL and login. You will see a welcome screen you can dismiss for now, but if you are brand new to Lenses this is a great resource for further information.

The first screen you see is the Environments screen. This is a listing of all the connected Kafka environments. For training purposes we will be working with an "on-prem" open source Kafka labedled Dev and a staging environment using Strimzi in the cloud. 

![environments screen cap](/images/initial-env-page.jpg)

Step 2: If you click on the next icon down on the right hand navigation bar that will take you to the Topics view. 

![topics picker image](/images/topics-picker.jpg)

This is the Topics view. It lists all the topics available across all your Kafka Environments. Note we are logged in here as Admin so we can view all topics. In the "real world" which topics appear here will depend on Lenses RBAC.

The Topics view is fully searchable by both topic name as well as key names from each topic's schema. Do a search for the keyword "longitude" - this will surface all topics with geographical fields in them. Click on the Search in Schema tick box to surface all the key name matches for each topic as well.

![topics search image](/images/topic-search.jpg)

Step 3: Let's take a deeper dive into the sea_vessel_position_reports topic by hovering over it in the list and then clicking on the SQL button on the right hand side that appears when we hover. 

![topics hover click sql studio](/images/sql-studio-jump.jpeg)

This moves us to the most commonly used aspect of Lenses - SQL Studio. It's designed for developers to interact with Lenses in a similar way they interact with code editors such as VS Code. By default when you click over from the topics view it will run a basic search in that topic showing the most current events across partitions in the topic.

![sql-studio-first-view](/images/sql-studio-first-view.jpeg)

You can also view schemas, topic configurations, and connected consumers side by side with searching the topics. We will be using other aspects of Lenses UI later on in the course, feel free to explore a bit on your own before we pick the lecture back up.


Step 5: Connect your Lenses MCP server to your LLM. Now our Lenses MCP server is fully connected to Lenses. We just have one last step: connect your LLM to the MCP server. 

Your instructor will provide connection instructions specific to the LLM you are using. The general process is the same regardless of which LLM client you choose: provide the MCP server URL to your LLM's MCP connector settings.

For example in Claude.ai you would go to Settings > Connectors > "Add custom connector" and fill in the URL. Other LLM clients will have similar configuration screens.

![configure claude](/images/configure-claude1.jpg)

Once connected you can now use your LLM to query and communicate with Lenses and its underlying data streams. 

### Add section here about connecting Context7 for Lenses docs and adding a directive to your LLM to use such docs???? 

### Lab 3: Explore Data With Your LLM

Step 1: Now that your LLM is all hooked up, let's have it help us explore the data flowing through Kafka. You will notice there are many different financial type topics on both of our clusters. Prompt your LLM to take a look at one of these topics to see what kinds of patterns of fraud it can spot. 

We are purposely leaving this part of the lab more open since your results will be non-deterministic. Any LLM will answer each question you pose in a slightly different way. But to get you started here are a few example prompts:

```
using your lenses mcp connection can you scan the most recent 100 transactions from the paypal-transactions topic in the staging environment for signs of possible fraud?
```
For these types of open-ended operations it's best to keep your sample sizes small to keep from running out of compute tokens.

![claude results](/images/claude-results.jpg)

Step 2: Get your LLM to follow up on its initial findings. Are there geographical anomalies in the data? Sales types that don't fit the vendors? Explore away. 

### Lab 4: Write SQL Queries With Your LLM

In the previous lab we let our LLM explore the data freely. Now let's get more specific and ask it to write targeted SQL queries for us that we can run in Lenses SQL Studio.

One of the most powerful aspects of pairing an LLM with Lenses is that it can inspect topic schemas through the MCP connection, understand what fields are available, and then write accurate SQL for you — no need to memorize the schema yourself.

Step 1: Let's start with a geographical query. The `nyc_yellow_taxi_trip_data` topic contains pickup and dropoff coordinates for NYC taxi trips. Ask your LLM to write a query that finds trips in a specific borough. Here's an example prompt:

```
Using your Lenses MCP connection, look at the schema for the nyc_yellow_taxi_trip_data topic in the dev environment. Then write me a Lenses SQL Studio query that finds all taxi trips that were picked up in Brooklyn. Use the pickup latitude and longitude coordinates and a bounding box for Brooklyn's approximate boundaries. 
```

Your LLM should inspect the schema, identify the `pickup_latitude` and `pickup_longitude` fields, and write something like:

```sql
SELECT *
FROM nyc_yellow_taxi_trip_data
WHERE pickup_latitude > 40.57 
  AND pickup_latitude < 40.74
  AND pickup_longitude > -74.04
  AND pickup_longitude < -73.85
```

Copy the SQL your LLM gives you and run it in Lenses SQL Studio to verify the results. Try changing the borough — ask for Manhattan, Queens, or the JFK airport area (hint: `RateCodeID = 2` is the JFK flat rate).

Step 2: Now let's query the financial data. We have several financial topics generated by our data generators including `credit-card-transactions` and `paypal-transactions`. Ask your LLM to write queries that surface interesting patterns. Here are some example prompts:

```
Look at the schema for credit-card-transactions in the dev environment. Write me a Lenses SQL Studio query that finds all transactions over $500 at gas stations. Those seem suspicious to me.
```

```
Write me a SQL Studio query for the paypal-transactions topic in dev that finds transactions where the amount is greater than 1000 and the city is not in the United States.
```

Step 3: Let's try the maritime data. The `sea_vessel_position_reports` topic contains AIS (Automatic Identification System) position data from ships in Scandinavian waters. Ask your LLM to help you find specific vessel activity:

```
Look at the sea_vessel_position_reports topic schema in dev. Write me a Lenses SQL Studio query that finds all vessels that appear to be stationary — where the Speed field is 0 or very close to 0. 
```

Step 4: Try writing your own prompts. Here are some ideas to get you thinking:

- Find taxi trips with unusually high tip percentages (tip_amount vs fare_amount)
- Query the `telecom_italia_data` topic to find high-traffic cell tower grid squares
- Look for credit card transactions in specific merchant categories
- Find sea vessels transmitting specific AIS message types

The key takeaway here is that your LLM can read schemas, understand field semantics, and write valid Lenses SQL — saving you from having to memorize topic structures across dozens of data sources. When your query doesn't return what you expect, paste the results back to your LLM and ask it to refine the SQL.

### Lab 5: Build a Poison Pill Filter

Step 1: Using Lenses SQL Studio let's do a search for bad records or poison pills in our nyc_yellow_taxi_trip_data. Specifically we are going to look for negative fare amounts since even a cancelled trip will have at worst a fare amount of zero. 

Open up the Lenses UI SQL Studio and find the nyc_taxi_trip_data in your dev environment. Run the following SQL search to see if these types of events exist:

```
SELECT *
FROM nyc_yellow_taxi_trip_data
WHERE fare_amount < 0
```
![sql taxi results](/images/sql-taxi-results.jpg)

Step 2: Now that we know these events exist and are fairly common let's get our LLM to write us some SQL Processor statements that will do this filtering for us. 

Here's an example prompt:

```
There are events with errors (poison pills) in the nyc_yellow_taxi_trip_data topic on our dev kafka cluster. events with fare_amount < 0
Write a lenses sql processor to filter out these poison pills into their own separate topic. this sql processor should create two new topics. One with just the poison pills called nyc_taxi_dlq and another called nyc_taxi_filtered.
Be careful. SQL Processors use a different SQL syntax than SQL Studio in lenses. It's stream processing so keep that in mind when you double check the SQL you're going to write.
```

Note that last statement is because LLMs can confuse the SQL Studio syntax with the SQL Processor syntax — which is mostly similar but there are a few key differences.

![claude processor](/images/claude-processor.jpg)

Hopefully your LLM will give you a result that looks similar to this. If it seems way off base send it back to work with a reminder that SQL Processor syntax is different than SQL Studio syntax. 

Step 3: Once your LLM has come up with the proper SQL for your processor, drill down into your Dev cluster and go to the App section. 

![apps-picker](/images/apps-picker.jpg)

In the Apps screen click on "Create SQL Processor." Give it a name like "taxi-trips-poison-pill-filter" and then copy and paste in the SQL from your LLM. 

Note! Sometimes your LLM will forget to include ```SET defaults.topic.autocreate=true;``` in the SQL statement so leave that in the pre-filled box if necessary. 

Once you've got the SQL copied and there's no listed errors. Click on the "Create Processor" button.

![sql processor create](/images/sql-proc-create.jpg)

Once the processor is created click on the Start Processor button to kick it off. Once it's up and running go to the Topology View to see it represented there. 

![topology view](/images/topology-view.jpg)

Then go to the topics viewer and check out your two new topics. 
