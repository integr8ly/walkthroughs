# Prerequisites

- The installation has already been performed on an OpenShift cluster as per the installation document https://github.com/integr8ly/installation/blob/master/README.md.

# Setting up the Enmasse Address
Open the `Enmasse Console`, click OpenShift and login using your OpenShift credentials. Go to the Addresses tab. Create a new address by clicking the `Create` button at the top of the page. Name the address `work-queue/requests`, select `queue` as the type and click `Next`. 
In the plan selection, choose the `Pooled Queue` plan and click `Next`. Confirm that
all details are correct and click `Create`. Wait until a green check mark appears 
which indicates that the Address has finished creating.

# Setting up the API Connector
In the `Fuse Console`, go to the Customizations tab. Click `Create API Connector`
and select `Use a URL`. Enter the URL of the Spring Boot app in the following
format: `<spring-boot-app>/v2/api-docs` and click `Next`. When prompted with `Review Actions`, click `Next`. When prompted with `Specify Security`, click `Next`. In the `Review/Edit Connector Details` section, change the `Connector Name` to `Walkthrough One A CRUD Connector` and click `Create API Connector`.

# Setting up the Enmasse Connection
In the `Fuse Console`, go to the Connections tab. Click `Create Connection` and select `AMQP Message Broker`. Enter the following details in the `AMQP Message Broker Configuration` section:
  - Connection URI: `amqp://<messaging-service-host>:<messaging-service-port>?amqp.saslMechanisms=PLAIN`
  - User Name: `<messaging-service-username>`
  - Password: `<messaging-service-password>`
  - Check Certificates: `Disable`

These values can all be found in the `messaging-service` config map within the walkthrough 1A namespace.

Click `Validate`. Once successfully validated, click `Next`. Enter `Walkthrough One A Messaging App` in the `Connection Name` field and click `Create`.

Create another connection and select `Walkthrough One A CRUD Connector`. When prompted with `Configure Connection`, click `Next`. Enter `Walkthrough One A CRUD App` in the `Connection Name` and click `Create`.

# Setting up the Integration
In the `Fuse Console`, go to the Integrations tab. Click `Create Integration` and select the `Walkthrough One A Messaging App` connection. When prompted to `Choose an Action`, select `Subscribe for Messages` and enter the following configuration and click `Next`:
  - Destination Name: `work-queue/requests`
  - Destination Type: `Queue`

When prompted with `Specify Output Data Type`, select the type `JSON Schema` and enter the following in the `Definition` field and click `Done`:

```json
{
	"$schema": "http://json-schema.org/draft-04/schema#",
	"type": "object",
	"properties": {
		"type": {
			"type": "string"
		}
	}
}
```

Choose `Walkthrough One A CRUD App` as the `Finish Connection` and select `Create a fruit` when prompted with `Choose an Action`. 

On the `Add to Integration`, click `Add a Step` and select `Data Mapper`. From the `Source`, drag `type` to `name` in the `body` of the `Target` section and select `Done`.

Click `Publish`. Enter `Walkthrough One A Integration` in the `Integration Name` field and click `Publish`. Wait for the publishing to complete.


# Using the Integration

Go to the `Messaging App` and create an item.

Go to the `CRUD App` and refresh the item.