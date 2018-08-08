# Overview

This document will walk you through the following integration flow

- Creating a Enmasse message queue 
- Creating a connection and integration in fuse ignite linked to the Enmasse message queue
- creating a connection to a http endpoint that receives messages via fuse integration
- Creating a http endpoint that can publish a message to the Enmasse queue 


# Prerequisites 

The installation has already been performed on an OpenShift cluster as per the installation document https://github.com/integr8ly/installation/blob/master/README.md.


# Login and choose the evals namespace

At the login screen you will see two options:

- htpassword
- rh_sso

You should pick the rh_sso option. You should then login as the ```evals@example.com``` user and on the right hand side, you should see ```My Projects``` select the ```evals``` project.
You should also take the opportunity to login your oc CLI as this user. You can do this by clicking the user at the top right of the OpenShift console UI and selecting ```copy login command``` from the drop down. Open a terminal and paste the command in.

# Setting up a Enmasse queue in your namespace 

- Provision a Enmasse service from the catalog (choose EnMasse standard) name it eval and choose not to bind at this time.
- Once complete, in the Eval namespace you will find a EnMasse provisioned service.
- Create a binding to this provisioned service checking each available option.
- Collapse the provisioned service and click on the dashboard link which will show on the right hand side of the EnMasse provisioned service. This link will bring you to the EnMasse dashboard. 
- When asked to login click the OpenShift button and login with the same user that provisioned the service. In an eval environment this will be ```evals@example.com```
- Create a new address in the EnMasse console named ```work-queue/requests``` choose the ```queue``` option and the pooled plan.
- Create a second address in the EnMasse console named ```work-queue/worker-updates``` choose the multicast option and the standard plan

# Setup Enmasse connection in Fuse Ignite

Back in your ```evals``` project, You will see a binding and a binding secret as part of the Enmaase provisioned service. There is an option to view this secret. Click on view and click on reveal. Take note of the following values

- messagingHost
- username
- password

open the Fuse Ignite route and login in to the Fuse Ignite console using the evals@example.com user 

- Select connections and create connection
- Find the AMQP connection and select it. 
- Fill in the Connection URI Field with the a URI that is in the following format  ```amqp://<messagingHost>:5672?amqp.saslMechanisms=PLAIN```
- Fill in the username and password fields
- Disable Check Certificates
- Click Validate 
- Click the blue ```next``` button at the top right
- Add a connection name ```enmaasse```
- Click Next / Save


# Deploy the messaging booster

clone the following booster git@github.com:ssorj/nodejs-messaging-work-queue.git

```
git clone git@github.com:ssorj/nodejs-messaging-work-queue.git

```

set up the templates in the cluster

```
cd nodejs-messaging-work-queue
oc apply -f templates

```
Deploy the applications

```
oc new-app --template=nodejs-messaging-work-queue-frontend
oc new-app --template=nodejs-messaging-work-queue-worker
```

Edit the messaging-service configmap

```
oc edit cm messaging-service -n eval:
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
oc rollout latest nodejs-messaging-work-queue-worker
```

# Setup HTTP connection to the messaging booster in Fuse Ignite
- Open the Fuse ignite console and open the connections again. 
- Click create connection
- Find the HTTP connection
- Set the base URL as the url exposed from the messaging front end application
- Click validate to ensure Fuse Ignite can reach the endpoint
- Click next 
- Give the connection a name (booster)
- Click create

# Creating the integration

- Open the Fuse Ignite console
- Click on integrations
- Click create integration
- Click on the Enmaase connection
- Click on the ``` Subscribe for messages ``` option
- Under Destination Name add ``` work-queue/worker-updates ```
- Choose Queue as the Destination Type
- Click Next
- Set the Specify Output Data Type as ``` Type specification not required ``` this is the default by clicking done
- On the ``` Choose a Finish Connection ``` screen select the messaging booster connection you created
- Choose ```Invoke URL ``` 
- In the URL Path set the path to ```/api/data```
- In the method select ```GET``` and click next
- Again under Specify Input Data Type leave it as ``` Type specification not required ``` and click done
- Click publish at the top right
- Choose the integration name (steel thread)


# Invoke the integration

- Open the front end booster webapp URL.
- Send a new message
- Back in the fuse console, open the integration and open the activity tab
