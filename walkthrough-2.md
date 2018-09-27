# Prerequisites

* Github Account


# Deploy the Fuse Integration App booster

* Go to Launcher
* Login
* Set the Application Name to xxxx (TODO: what should this be?)
* Follow through the wizard, authorizing openshift & github
* Choose the Fuse booster (TODO: what is the name of it, and what version should be choosen in the dropdown)
* Create the booster
* Go to OpenShift Project & verify it has created the following:
  * Fuse Integration App
  * API Server 1 (TODO: Better name)
  * API Server 2 (TODO: Better name)

# Modify the Fuse Integration App to aggregate data from the API Servers

* Go to Che
* Create new Java workspace, importing the Fuse integration app repo
* Uncomment some code in the `CamelRouter.java` file (TODO: which code)
* Commit and Push the change, thereby triggering a build & deploy
* Verify build is successful & app redeployed

# Add the Fuse Integration App endpoint to 3Scale

* Go to 3Scale Admin Dashboard
* Choose RHSSO & login if needed (TODO: will SSO kick in here first time?)
* Create a new API, pointing at the Fuse integration App (TODO: steps for this)
* Publish the API to APICast
* (TODO: modify the developer portal to include the swagger API)
* (TODO: Configure rate limiting on the API to 5/minute)

# Call the Fuse Integration App endpoint from the 3Scale Developer Portal

* Go to the 3Scale Developer Portal (TODO how to get link?)
* Register as a developer & login
* Do the below steps using the Swagger Client UI in the Developer Portal

## Successful Call

* Ensure the User Key parameter is set
* Call the Fuse App API
* Verify a successful response

## No API Key

* Remove the User Key parameter
* Call the Fuse App API
* Verify the call fails with a permission denied error

## Rate Limiting

* Set the User Key parameter back
* Call the Fuse App API multiple times
* Verify after 5 calls, the call responds with an error about rate limits being reach

# Activity Monitoring

* Go back to the 3Scale Admin Dashboard
* Go to the Analytics section
* Verify the graph is showning successful & failed/errorer calls