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

### Lab 2: Configuring your MCP Server

Step 1: Create a role for your MCP server. Your MCP server will need a Lenses Service Account. We will need to configure a few settings in IAM to facilitate this. 

![iam button](/images/iam-pick.png)

First up we will need to create a role to define the limititations and permissions for our AI "user" to access Lenses via the MCP server. Click on Roles section at the top of the IAM screen. Then click on the orange "Add New Role" button in the middle of the screen. Call your roll MCP-admin. Lenses will automatically create a lower case version for the resource name. You can leave this, and fill in the description if you'd like. Click on create role. 

![iam create role](/images/create-role1.jpg)

Once your role is created click on it to modify it. Then click on Manage Permissions to go to the permissions editing screen. For simplicity purposes we are going to grant our MCP server admin rights to our Lenses cluster, but in the "real world" much better care should be given to limiting this sort of access. See Lenses documentation for details: https://docs.lenses.io/latest/user-guide/iam 

Copy and past the following into your mcp-admin permissions and then click Save Updates.

```
name: mcp-admin
policy:
  - action: '*'
    resource: '*'
    effect: allow
```
![iam save role settings](/images/iam-admin-save.jpg)

Step 2: Create a group for your MCP Server role. In the upper left hand corner of the IAM screen click on the "Groups" button to create a group for our MCP Server role. Then click on the orange button "Add a new group." Call it MCP-admin-group and let Lenses populate the Resource name and SSO mapping name for you. At the bottom be sure to select your mcp-admin role from the dropdown. Then click on the "Create group" button. 

![iam create group](/images/create-group.jpg)

Step 3: Create Service account for your MCP server. Now that we have the group and role configured we can add our Service Account. On the top left select "Service accounts" and then click on the "Add a new service account" button. Give your new service account a name: MCP-admin-sa. Then at the bottom add it to the group you just created: "MCP-admin-group" and then select a key expiry time. For simplicity we will set ours to 7 days - but obviously in the real world this should reflect your own organization's security policy. Then click on "Create service account" button. 

![iam create service account](/images/create-service-account.jpg)

Once the account is created Lenses will setup a service account key. We will need to provide this key to our MCP server to connect to Lenses. Be sure to copy your key to a text file before closing the screen if you don't have it for the next steps you'll need to come back and create an new service account to get a new key. When you have copied your key to a text document click on the "I have saved my key" button. 

Now we have everything we need to configure our MCP server to connect to our Lenses installation. 

Step 4: SSH in to your MCP server. Your instructor will provide you with the URL and .pem file to SSH in to your MCP server to configure it with your Lenses service account information. 

Using your SSH client of choice open up an SSH connection to your MCP server. Here's the SSH command to use for Mac and Linux

```
ssh -i <path to .pem file> ec2-user@<Public IP of your MCP server>
```

(Note you will need to change the permissions of the .pem file `chmod 400 <path to .pem file>`)

![ssh connect to server](/images/ssh-to-mcp.jpg)

Once you've connected switch to the /opt/lenses-mcp directory `cd /opt/lenses-mcp`. In that directory is a hidden file called .env - this is where you configure your MCP settings. Run the following command to open the .env file in the vi editor. `sudo vi .env` (Note if you don't like vi, you can install Nano. You will need to run `sudo yum install nano` to install it.) 

Now it's just a matter of filling out the form. Use your HQ URL provided by your instructor to fill out the form, then you will need to copy your Service Account key in as well. When it's all done it should look like this:

![ssh env file](/images/updated-env-file1.jpg)

Once you are done editing the file save your changes and exit your file editor. Then run the following command to restart the MCP server to make it reload the config file you just updated. `sudo systemctl restart lenses-mcp`

Step 5: Connect your Lenses MCP server to Claude.ai. Now our Lenses MCP server is fully connected to Lenses. We just have one last step: connect Claude AI to the MCP server. 

Connect and login to claude.ai. Under your account go to Settings. Then go to Connectors and click on "Add custom connector"

Then it just a matter of giving your Lenses MCP connection an name and filling in the URL for your server you obtained from your instructor. Click "Add" when you're done. 

![configure claude](/images/configure-claude1.jpg)

Once connected you can now use Claude to query and communicate with Lenses and it's underlying data streams. 

### Add section here about connecting Context7 for Lenses docs and adding a directive to Claude to use such docs???? 

### Lab 3: Explore Data With Claude

Step 1: Now that Claude is all hooked up, let's have him help us explore the data flowing through Kafka. You will notice there are many different financial type topics on both of our clusters. Prompt Claude to take a look at one of these topics to see what kinds of patterns of fraud he can spot. 

We are purposely leaving this part of the lab more open since your results will be non-deterministic. Claude or another LLM will answer each question you pose in a slightly different way. But to get you started here are a few example prompts:

```
using your lenses mcp connection can you scan the most recent 100 transactions from the paypal-transactions topic in the staging environment for signs of possible fraud?
```
For these types of open-ended operations it's best to keep your sample sizes small to keep from running out of compute tokens.

![claude results](/images/claude-results.jpg)

Step 2: Get Claude to follow up with his initial findings? Are there geographical anomolies in the data? Sales types that don't fit the vendors? Explore away. 

### Lab 4: Build a Poison Pill Filter

Step 1: Using Lenses SQL Studio let's do a search for bad records or poison pills in our nyc_yellow_taxi_trip_data. Specifically we are going to look for negative fare amounts since even a cancelled trip will have at worst a fare amount of zero. 

Open up the Lenses UI SQL Studio and find the nyc_taxi_trip_data in your dev environment. Run the following SQL search to see if these types of events exist:

```
SELECT *
FROM nyc_yellow_taxi_trip_data
WHERE fare_amount < 0
```
![sql taxi results](/images/sql-taxi-results.jpg)

Now that we know these events exist and are fairly common let's get Claude to write us some SQL Processor statements will that will do this filtering for us. 

Here's an example prompt:

```
There are events with errors (poison pills) in the nyc_yellow_taxi_trip_data topic on our dev kafka cluster. events with fare_amount < 0
Write a lenses sql processor to filter out these poison pills into their own separate topic. this sql processor should create two new topics. One with just the poison pills called nyc_taxi_dlq and another called nyc_taxi_filtered.
Be careful Claude. SQL Processors use a different SQL syntax than SQL Studio in lenses. It's stream processing so keep that in mind when you double check the SQL you're going to write.
```

Note that last statement is because Claude can confuse the SQL Studio syntax from the SQL Processor syntax - which is mostly similar but there are a few key differences.

![claude processor](/images/claude-processor.jpg)

Hopefully your Claude iteration will give you a result that looks similar to this. If he seems way off base send him back to work with a reminder that SQL Studio Processing is different than SQL Studio Syntax. 

Once he's come up with the proper SQL for your processor. Drill down into your Dev cluster and go to the App section. 

![apps-picker](/images/apps-picker.jpg)

In the Apps screen click on "Create SQL Processor." Give it a name like "taxi-trips-poison-pill-filter" and then copy and paste in the SQL from Claude. 

Note! Sometimes Claude "forgets" to put ```SET defaults.topic.autocreate=true;``` in the SQL statement so leave that in pre-filled box if necessary. 

Once you've got the SQL copied and there's no listed errors. Click on the Create "Processor Button."

![sql processor create](/images/sql-proc-create.jpg)
