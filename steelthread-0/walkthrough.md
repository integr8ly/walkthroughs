# Overview

This document will walk you through the following integration flow

- Creating a Enmasse message queue 
- Creating a connection and integration in fuse ignite linked to the Enmasse message queue
- Creating a http endpoint that can publish a message to the Enmasse queue 
- Creating a http endpoint that exposes a REST API using Launcher
- Creating a connection to REST API from fuse
- Creating an integration between all of these via Fuse Ignite



# Prerequisites 

The installation has already been performed on an OpenShift cluster as per the installation document https://github.com/integr8ly/installation/blob/master/README.md.


# Login and choose the evals namespace

At the login screen you will see two options:

- htpassword
- rh_sso

You should pick the rh_sso option. You should then login as the ```evals@example.com``` user and on the right hand side, you should see ```My Projects``` select the ```evals``` project.
You should also take the opportunity to login your oc CLI as this user. You can do this by clicking the user at the top right of the OpenShift console UI and selecting ```copy login command``` from the drop down. Open a terminal and paste the command in.

# Setting up a Enmasse queue in your namespace 

- From your namespace click the catalog button on the left hand nav
- Provision a Enmasse service from the catalog (choose EnMasse standard) name it eval and choose not to bind at this time.
- Once complete, in the Eval namespace you will find a EnMasse provisioned service.
- Now create a binding to this provisioned service checking each available option.
- Collapse the provisioned service and click on the dashboard link which will show on the right hand side of the EnMasse provisioned service. This link will bring you to the EnMasse dashboard. 
- When asked to login click the OpenShift button and login with the same user that provisioned the service. In an eval environment this will be ```evals@example.com```
- Create a new address in the EnMasse console named ```work-queue/requests``` choose the ```queue``` option and the pooled plan.
- Create a second address in the EnMasse console named ```work-queue/worker-updates``` choose the ```queue``` option and the standard plan

# Deploy the CRUD REST HTTP booster

This is the HTTP endpoint that will revieve the requests from the Fuse Ignite 
integreation we create and add new fruits to the inventory.


Load up the launcher application. This is at ```http://launcher-launcher.<BASE_MASTER_DOMAIN> ``` 

- Click on launch your project
- Login when prompted using the ```evals@example.com``` user
- The create application form will show. Choose ```eval``` as the name (this equates to your namespace)
- When prompted to select target environment choose : ```Code Locally, Build and Deploy```
- Click the blue down arrows
- Under mission select ```Crud```
- Under runtime select ```Node.js```
- Click the down arrows
- Under ``` Authorize Git Provider``` you will need to authorize launcher to have access to your github account.
- Once authorised, under the repository input change the value to ```crud-app```
- Once the down arrows go blue click them
- Click setup application.
- This will create a repo called crud-app in your github account and deploy the CRUD booster to the eval namespace.

You can try out this application by going to the eval namepsace and clicking on the route created for this new application.


# Deploy the messaging booster
**TODO at the moment we do this manually in the future it will be done via launcher**

Fork the following booster https://github.com/ssorj/nodejs-messaging-work-queue
into your own repo. We need to fork it as we will be adding some code changes later.

Next clone the repo to your local machine

```
git clone git@github.com:<YOUR_GITHUB_HANDLE>/nodejs-messaging-work-queue.gi
```

set up the templates in the cluster

```
cd nodejs-messaging-work-queue
oc apply -f templates
```
Deploy the frontend application

```
oc new-app --template=nodejs-messaging-work-queue-frontend -p SOURCE_REPOSITORY_URL=https://github.com/<YOUR_GITHUB_ACCOUNT>/nodejs-messaging-work-queue.git
```

Edit the messaging-service configmap

```
oc edit cm messaging-service -n eval
```

Add in the values from the secret from the Enmasse binding secret

Set the following key value pairs

```
MESSAGING_SERVICE_HOST = <value from messagingHost>
MESSAGING_SERVICE_PASSWORD = <value from password>
MESSAGING_SERVICE_USER= <value from username>
```

Restart the booster services

```
oc rollout latest nodejs-messaging-work-queue-frontend
```

In the log of the application you should see something like:

```
frontend-nodejs-aad7: Connected to AMQP messaging service at messaging.enmasse-eval.svc:5672
```

This is how we know it has connected to the AMQP service succesfully. 

# Setup Enmasse connection in Fuse Ignite

Back in your ```evals``` project, You will see a binding and a binding secret as part of the Enmaase provisioned service. There is an option to view this secret. Click on view and click on reveal. Take note of the following values

- messagingHost
- username
- password

- Open the Fuse Ignite route 
- Login in to the Fuse Ignite console using the evals@example.com user 
- Select connections and create connection
- Find the AMQP connection and select it. 
- Fill in the Connection URI Field with the a URI that is in the following format  ```amqp://<messagingHost>:5672?amqp.saslMechanisms=PLAIN```
- Fill in the username and password fields
- Disable Check Certificates
- Click Validate 
- Click the blue ```next``` button at the top right
- Add a connection name ```enmaasse```
- Click Next / Save



# Setup HTTP connection to the CRUD REST booster in Fuse Ignite
- Click on connections
- Click create connection
- Find the HTTP connection
- Set the base URL as the url exposed from the CRUD REST booster application that we created earlier
- Click validate to ensure Fuse Ignite can reach the endpoint
- Click next 
- Give the connection a name (crud booster)
- Click create

# Creating the integration
- In the Fuse console
- Click on integrations
- Click create integration
- Click on the Enmaase connection
- Click on the ``` Subscribe for messages ``` option
- Under Destination Name add ``` work-queue/requests ```
- Choose Queue as the Destination Type
- Click Next
- Set the Specify Output Data Type as ``` JSON Schema ``` 
- Add the following schema
```
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
- On the ``` Choose a Finish Connection ``` screen select the crud booster connection you created
- Choose ```Invoke URL ``` 
- In the URL Path set the path to ```/api/fruits```
- In the method select ```POST``` and click next
- Again under Specify Input Data Type set it to ``` JSON Schema ``` 
- Add the following schema
```
{
	"$schema": "http://json-schema.org/draft-04/schema#",
	"type": "object",
	"properties": {
		"name": {
			"type": "string"
		},
		"stock": {
			"type": "string"
		}
	}
}

```
Next click the small plus between the two connections on the left hand side then click ``` add step```

- Pick ```data mapper``` from the choices 
- Map the stock property to the stock property by dragging stock from the left model to the right model
- Map type to name by dragging type from the left model to name on the right model
- Click done at the top right

Add some logging steps (optional)

To add some more visibility we can add some logging steps to our integration.

- On the left hand side inbetween the connections you will see small blue boxes with plus symbols. For each of these do the following:

- click it
- click add step
- choose log
- select ```message body```

Once done we should now have 5 steps in our integration. Click publish in the top right.

Name the integration ```steel thread```


# Setup CHE to enable editing of the booster

#TODO DAVE MARTIN

# Customise the messaging booster
We want to add some additional information to the message. To do this we will need to change some of the code in the booster you cloned locally earlier. 

- In che, in your workspace, click the workspace menu item and select import project
- Select github as the target
- enter the github repo url for the messaging booster (something like git@github.com:<YOUR_GITHUB_ACCOUNT>/nodejs-messaging-work-queue.git)
- under project configuration select ```nodejs```
- open the project and change the context of index.html to be

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <title>Messaging Work Queue</title>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <link href="https://fonts.googleapis.com/css?family=Source+Sans+Pro:300,300italic,400,400italic,600,600italic|Ubuntu+Mono:400" rel="stylesheet" type="text/css"/>
    <link rel="stylesheet" href="/app.css" type="text/css"/>
    <link rel="icon" href="" type="image/png"/>
    <script type="text/javascript" src="/gesso.js"></script>
    <script type="text/javascript" src="/app.js"></script>
    <script>
      "use strict";
      const app = new Application();
    </script>
  </head>
  <body>
    <div id="-body">
      <div id="-body-content">
        <h1>Messaging Work Queue</h1>

        <h2>Add Fruit</h2>

        <form id="requests" method="post" action="javascript:void(0);">
          Fruit: <input id="request-text" type="text" name="text" autofocus="autofocus"/>
          Stock: <input id="request-stock" type="text" name="stock" autofocus="autofocus"/>

          <button>Send request</button>
        </form>
      </div>
    </div>
  </body>
</html>


```


Next open ```app.js`` and  replace it with:

```js
 "use strict";

const gesso = new Gesso();

class Application {
    constructor() {
        this.data = null;

        window.addEventListener("statechange", (event) => {
            this.renderResponses();
            this.renderWorkers();
        });

        window.addEventListener("load", (event) => {
            this.fetchDataPeriodically();

            $("#requests").addEventListener("submit", (event) => {
                this.sendRequest(event.target);
                this.fetchDataPeriodically();
            });
        });
    }

    fetchDataPeriodically() {
        gesso.fetchPeriodically("/api/data", (data) => {
            this.data = data;
            window.dispatchEvent(new Event("statechange"));
        });
    }

    sendRequest(form) {
        console.log("Sending request");

        let request = gesso.openRequest("POST", "/api/send-request", (event) => {
            if (event.target.status >= 200 && event.target.status < 300) {
                this.fetchDataPeriodically();
            }
        });

        let data = {
            text: form.text.value,
            stock: form.stock.value,
            uppercase: false,
            reverse: false,
        };

        let json = JSON.stringify(data);

        request.setRequestHeader("Content-Type", "application/json");
        request.send(json);

        form.text.value = "";
        form.stock.value = "";
    }

    renderResponses() {
      
    }

    renderWorkers() {
        console.log("Rendering workers");

    }
}


```

Finally open the ```server.js``` and replace the ```/api/send-request``` route with the following

```js

app.post("/api/send-request", (req, resp) => {
    let message = {
        message_id: `${id}/${request_sequence++}`,
        application_properties: {
            uppercase: req.body.uppercase,
            reverse: req.body.reverse
        },
        body: JSON.stringify({type:req.body.text, stock: req.body.stock})
    };

    request_messages.push(message);
    request_ids.push(message.message_id);

    send_requests();

    resp.status(202).send(message.message_id);
});


```

Save your changes.

- Open the git menu and click commit. Type in a commit message into the message window. 
- Select push commited changes to option and ensure it is pointing to ```origin/master``` then click commit

# Invoke the integration

- Open the front end messaging webapp URL.
- Send a new message
- Back in the fuse console, open the integration and open the activity tab
- You should see the different steps it went through while invoking the integration
- Next open the route to the CRUD rest app. You should see a new fruit type has been added with a stock quantity.
