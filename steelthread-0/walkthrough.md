# Setting up a Enmasse queue in your namespace 

The following doc walks you through setting up a new queue via the catalog

https://github.com/lulf/openshift-amqp-clients/blob/master/service-catalog-tutorial.md

# Ignite Setup

Installation steps: https://github.com/integr8ly/installation-notes/tree/master/ipaas

NOTE: This setup requires an already configured keycloak user and client id in the enmasse namespace.

## Ignite Enmasse Setup

Create a new enmasse connection:

* Connections -> Create Connection
* Create a new AMQ connection:
    * Broker URL: your enmasse messaging service url (tcp://messaging-something.svc:$port)
    * User Name: enmasse keycloak username
    * Password: enmasse keycloak user password
    * Client ID: enmasse keycloak client id
* Click "validate" to check the amq connection
* Click "save"
