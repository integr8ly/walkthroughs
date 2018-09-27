# Prerequisites

- The installation has already been performed on an OpenShift cluster as per the installation document https://github.com/integr8ly/installation/blob/master/README.md.

# Setting up the AMQ Connection

Open the `Fuse Console` and go to the Connections tab. Create a connection, select
`AMQ Connection`. In the Broker URL field, enter the URL of the AMQ TCP Service
in the following format: `tcp://<tcp-service>:61616`. In the User Name and the
Password field enter the username and password from the AMQ Stateful Set. Click
`Validate`, when it passes then select `Next`.

In the Connection Name field enter `Walkthrough One AMQ`. Click `Create`.

# Setting up the HTTP Connection

In the `Fuse Console` go to the Customizations tab. Click `Create API Connector`.
Select `Use a URL`, in the text field enter the URL of the CRUD application in
the following format: `<crud-app-url>/v2/api-docs`. Click `Next`. On the Specify
Security screen, click `Next`. On the Review Connector Details screen, update the
Connector Name to `Walkthrough One CRUD App`, click `Create API Connector`.

In the `Fuse Console` go to the Connections tab. Click `Create Connection`, select
`Walkthrough One CRUD App`. On the Configuration screen click `Next`. On the Add
Connection Details screen, enter `Walkthrough One CRUD` as the Connection Name.
Click `Create` 

# Setting up the Integration

In the `Fuse Console` go to the Integrations tab. Click `Create Integration`. For
the Start Connection, select `Walkthrough One AMQ`. On the "Choose an action"
screen, select `Subscribe for messages`. In the Destination Name field, enter
`work-queue/requests`, choose Queue for Destination Type. Click `Next`.

On the "Specify Output Data Type" screen, select "JSON Schema" for Select Type.
Enter the following in the Definition field:

```json
{
	"$schema": "http://json-schema.org/draft-04/schema#",
	"type": "object",
	"properties": {
		"type": {
			"type": "string"
		},
		"stock": {
			"type": "string"
		}
	}
}
```

For a Finish Connection, select `Walkthrough One CRUD`. On the "Choose an Action"
screen select "Create a fruit".

On the "Add to Integration" screen, select "Add a Step", chose "Data Mapper". From
the "Source" section, drag "type" to "name" in the body of the "Target" section.
Select `Done`.

Click `Publish`. For the Integration Name, enter `Walkthrough One Integration" and
click `Publish`. Wait for the publishing to complete.

# Using the Integration

Go to the `Messaging App` and create an item.
Go to the `CRUD App` and refresh the item.
