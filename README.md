# Lenses MCP Training Workshop Hands-On Labs

## Lab Outline

Lab 1: Familiarize Yourself With Lenses UI

Lab 2: Configure Your MCP Server

Lab 3: Explore Data With Claude

Lab 4: Build a Poison Pill Filter With Claude

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

### Lab 2: Connecting your LLM to Lenses MCP

Step 1: Connect your Lenses MCP server to Claude.ai.

Connect and login to claude.ai. Under your account go to Settings. Then go to Connectors and click on "Add custom connector"

Then it just a matter of giving your Lenses MCP connection an name and filling in the URL for your server you obtained from your instructor. Click "Add" when you're done. 

![configure claude](/images/configure-claude1.jpg)

Once connected you can now use Claude to query and communicate with Lenses and it's underlying data streams. 

### Add section here about connecting Context7 for Lenses docs and adding a directive to Claude to use such docs???? 

# Lab 3: Explore Data With Claude

In this lab you'll use Claude and the Lenses MCP connection to perform common developer tasks: understanding unfamiliar data, validating schemas before building consumers, and generating test data requirements.

## Lab 3A: Schema Discovery & Documentation

**Scenario:** You've just joined a team that works with airline data streams. Before you can contribute, you need to understand what data is available and how the topics relate to each other.

**Step 1:** Ask Claude to survey the available airline-related topics in your environment.

```
Using your Lenses MCP connection, list all topics in the staging environment that appear to be related to airline or flight data. For each topic, tell me:
- The topic name
- Number of partitions
- Whether it has a schema registered
```

**Step 2:** Once you have a list of topics, pick 2-3 that seem related and ask Claude to examine their schemas in detail.

```
Examine the schemas for [topic1] and [topic2] in staging. 
Give me a field-by-field breakdown including data types.
Do these topics share any common fields that could be used to join or correlate events?
```

**Step 3:** Have Claude generate a quick data dictionary you could share with your team.

```
Based on what you've learned about these airline topics, generate a markdown data dictionary I could add to our team's documentation. Include topic names, descriptions of what each topic likely contains, key fields, and any relationships between topics.
```

This is exactly how developers use MCP in practice - rapid exploration of unfamiliar data landscapes without having to manually click through dozens of topics in a UI.

---

## Lab 3B: Data Validation for a New Consumer

**Scenario:** You've been assigned to build a new microservice that consumes financial transaction data. Before writing any code, you need to understand the actual data format, edge cases, and potential gotchas.

**Step 1:** Start by sampling real messages to understand what you'll be working with.

```
I'm building a consumer for the credit_card_transactions topic in the dev environment. 
Sample 100 recent messages and give me a summary:
- What fields are present in every message?
- Are there any null or missing fields I'll need to handle defensively?
- What are the data types for each field?
```

**Step 2:** Understand the value ranges and distributions you'll need to handle.

```
Looking at those same messages from credit_card_transactions:
- What's the range of transaction amounts?
- What are all the different values you see for status or transaction_type fields?
- Are there any fields with unexpected or potentially problematic values?
```

**Step 3:** Generate test fixtures for your consumer code.

```
Based on your analysis of credit_card_transactions, give me 3 representative JSON messages I can use as test fixtures:
1. A "happy path" typical transaction
2. An edge case with minimum/maximum values
3. A message with any optional or nullable fields set to null
```

This workflow helps you write robust consumer code from day one, rather than discovering edge cases in production.

---

## Lab 3C: Building Test Data Requirements

**Scenario:** Your team needs integration tests that use realistic data. Rather than inventing fake data that might not match production patterns, you'll analyze real streams to build accurate test data specifications.

**Step 1:** Pick a topic and have Claude analyze the actual patterns in the data.

```
Analyze the sea_vessel_position_reports topic in dev. Sample 50 messages and tell me:
- What are the realistic ranges for latitude and longitude?
- What vessel_type values appear and how frequently?
- What's the typical format and length of vessel identifiers?
- Are there any fields that follow specific patterns (like timestamps or codes)?
```

**Step 2:** Ask Claude to identify edge cases that your tests should cover.

```
Based on your analysis of sea_vessel_position_reports, what edge cases should our integration tests cover? 
Look for:
- Boundary values (min/max coordinates, speeds, etc.)
- Rare but valid values in enum-like fields
- Any data quality issues we should test our error handling against
```

**Step 3:** Have Claude generate a test data specification document.

```
Generate a test data specification for sea_vessel_position_reports that our team can use to build realistic mock data. Format it as a markdown document with:
- Field names and types
- Valid value ranges with examples
- Required vs optional fields
- Edge cases to include in test suites
- Sample valid and invalid messages
```

This approach ensures your test data reflects actual production patterns, making your integration tests far more valuable.

### Lab 4: Build a Poison Pill Filter

Step 1: Using Lenses SQL Studio let's do a search for bad records or poison pills in our nyc_yellow_taxi_trip_data. Specifically we are going to look for negative fare amounts since even a cancelled trip will have at worst a fare amount of zero. 

Open up the Lenses UI SQL Studio and find the nyc_taxi_trip_data in your dev environment. Run the following SQL search to see if these types of events exist:

```
SELECT *
FROM nyc_yellow_taxi_trip_data
WHERE fare_amount < 0
```
![sql taxi results](/images/sql-taxi-results.jpg)

Step 2: Now that we know these events exist and are fairly common let's get Claude to write us some SQL Processor statements will that will do this filtering for us. 

Here's an example prompt:

```
There are events with errors (poison pills) in the nyc_yellow_taxi_trip_data topic on our dev kafka cluster. events with fare_amount < 0
Write a lenses sql processor to filter out these poison pills into their own separate topic. this sql processor should create two new topics. One with just the poison pills called nyc_taxi_dlq and another called nyc_taxi_filtered.
Be careful Claude. SQL Processors use a different SQL syntax than SQL Studio in lenses. It's stream processing so keep that in mind when you double check the SQL you're going to write.
```

Note that last statement is because Claude can confuse the SQL Studio syntax from the SQL Processor syntax - which is mostly similar but there are a few key differences.

![claude processor](/images/claude-processor.jpg)

Hopefully your Claude iteration will give you a result that looks similar to this. If he seems way off base send him back to work with a reminder that SQL Studio Processing is different than SQL Studio Syntax. 

Step 3: Once he's come up with the proper SQL for your processor. Drill down into your Dev cluster and go to the App section. 

![apps-picker](/images/apps-picker.jpg)

In the Apps screen click on "Create SQL Processor." Give it a name like "taxi-trips-poison-pill-filter" and then copy and paste in the SQL from Claude. 

Note! Sometimes Claude "forgets" to put ```SET defaults.topic.autocreate=true;``` in the SQL statement so leave that in pre-filled box if necessary. 

Once you've got the SQL copied and there's no listed errors. Click on the "Create Processor" button.

![sql processor create](/images/sql-proc-create.jpg)

Once the processor is created click on the Start Processor button to kick it off. Once it's up and running go to the Topology View to see it represented there. 

![topology view](/images/topology-view.jpg)

Then go to the topics viewer and check out your two new topics. 


