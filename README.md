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
