# Lenses MCP Training Workshop Hands-On Labs
<!-- v. 4 -->

## Lab Outline

Lab 1: Familiarize Yourself With Lenses UI

Lab 2: Configure Your MCP Server

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


### Lab 2: Configure Your MCP Server

Step 1: Connect your Lenses MCP server to your LLM. 

Your instructor will provide connection instructions specific to the LLM you are using. The general process is the same regardless of which LLM client you choose: provide the MCP server URL to your LLM's MCP connector settings.

For example in Claude.ai you would go to Settings > Connectors > "Add custom connector" and fill in the URL. Other LLM clients will have similar configuration screens.

![configure claude](/images/configure-claude1.jpg)

Once connected you can now use your LLM to query and communicate with Lenses and its underlying data streams. 

### Add section here about connecting Context7 for Lenses docs and adding a directive to your LLM to use such docs???? 

### Lab 3: Explore Data With Your LLM

In this lab you'll use your LLM and the Lenses MCP connection to perform common developer tasks: understanding unfamiliar data, validating schemas before building consumers, and generating test data requirements.

#### Lab 3A: Schema Discovery & Documentation

**Scenario:** You've just joined a team that works with airline data streams. Before you can contribute, you need to understand what data is available and how the topics relate to each other.

**Step 1:** Ask your LLM to survey the available airline-related topics in your environment.

```
Using your Lenses MCP connection, list all topics in the staging environment that appear to be related to airline or flight data. For each topic, tell me:
- The topic name
- Number of partitions
- Whether it has a schema registered
```

**Step 2:** Once you have a list of topics, pick 2-3 that seem related and ask your LLM to examine their schemas in detail.

```
Examine the schemas for [topic1] and [topic2] in staging. 
Give me a field-by-field breakdown including data types.
Do these topics share any common fields that could be used to join or correlate events?
```

**Step 3:** Have your LLM generate a quick data dictionary you could share with your team.

```
Based on what you've learned about these airline topics, generate a markdown data dictionary I could add to our team's documentation. Include topic names, descriptions of what each topic likely contains, key fields, and any relationships between topics.
```

This is exactly how developers use MCP in practice — rapid exploration of unfamiliar data landscapes without having to manually click through dozens of topics in a UI.

---

#### Lab 3B: Data Validation for a New Consumer

**Scenario:** You've been assigned to build a new microservice that consumes financial transaction data. Before writing any code, you need to understand the actual data format, edge cases, and potential gotchas.

**Step 1:** Start by sampling real messages to understand what you'll be working with.

```
I'm building a consumer for the credit-card-transactions topic in the dev environment. 
Sample 100 recent messages and give me a summary:
- What fields are present in every message?
- Are there any null or missing fields I'll need to handle defensively?
- What are the data types for each field?
```

**Step 2:** Understand the value ranges and distributions you'll need to handle.

```
Looking at those same messages from credit-card-transactions:
- What's the range of transaction amounts?
- What are all the different values you see for status or transaction_type fields?
- Are there any fields with unexpected or potentially problematic values?
```

**Step 3:** Generate test fixtures for your consumer code.

```
Based on your analysis of credit-card-transactions, give me 3 representative JSON messages I can use as test fixtures:
1. A "happy path" typical transaction
2. An edge case with minimum/maximum values
3. A message with any optional or nullable fields set to null
```

This workflow helps you write robust consumer code from day one, rather than discovering edge cases in production.

---

#### Lab 3C: Building Test Data Requirements

**Scenario:** Your team needs integration tests that use realistic data. Rather than inventing fake data that might not match production patterns, you'll analyze real streams to build accurate test data specifications.

**Step 1:** Pick a topic and have your LLM analyze the actual patterns in the data.

```
Analyze the sea_vessel_position_reports topic in dev. Sample 50 messages and tell me:
- What are the realistic ranges for latitude and longitude?
- What vessel_type values appear and how frequently?
- What's the typical format and length of vessel identifiers?
- Are there any fields that follow specific patterns (like timestamps or codes)?
```

**Step 2:** Ask your LLM to identify edge cases that your tests should cover.

```
Based on your analysis of sea_vessel_position_reports, what edge cases should our integration tests cover? 
Look for:
- Boundary values (min/max coordinates, speeds, etc.)
- Rare but valid values in enum-like fields
- Any data quality issues we should test our error handling against
```

**Step 3:** Have your LLM generate a test data specification document.

```
Generate a test data specification for sea_vessel_position_reports that our team can use to build realistic mock data. Format it as a markdown document with:
- Field names and types
- Valid value ranges with examples
- Required vs optional fields
- Edge cases to include in test suites
- Sample valid and invalid messages
```

This approach ensures your test data reflects actual production patterns, making your integration tests far more valuable. 

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
