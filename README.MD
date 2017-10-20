# Shippable Voting App

This sample creates a multi-container application in a GKE Kubernetes cluster. The application interface has been built using Python / Flask. The data component is using Redis.

To walk through a quick deployment of this application, follow the steps below.

##Pre-requisites

You need the following in order to run this sample:

- A Google Cloud account at https://cloud.google.com/
  * A Project that has Google Container Engine (GKE) and Google Container Registry (GCR) enabled
  * An [existing Kubernetes cluster](https://cloud.google.com/container-engine/docs/clusters/operations) on GKE where you will deploy this sample application
- A GitHub account where you will fork and run this sample
- [Sign in with GitHub](https://app.shippable.com) to create a Shippable account

###Step 1: Fork the application

Fork this repository to your GitHub account.

###Step 2: Create integrations

Since your deployment config will interact with GCR to push the front-end image and GKE to deploy the application, you will need to create integrations for these in the Shippable UI.

Follow steps in **Adding an account integration** section to create these integrations:
- [Create a GCR integration called drship_gcr](http://docs.shippable.com/platform/integration/gcr/)
- [Create a GKE integration called drship_gke](http://docs.shippable.com/platform/integration/gke/)

You will also need to [create a GitHub integration](http://docs.shippable.com/platform/integration/github/) to access contents of your repository. Make sure the friendly name of this integration is `dr_github`.

###Step 3: Edit the config

Edit the following files:

In `shippable.resources.yml`:
- In the `twotier_app_gitRepo` resource definition, replace `devops-recipes/twotier-kubernetes-gke` with the name of your forked repo
- In `kube_cluster`, replace the value of `sourceName` and `region` with the name of your GKE cluster name and region.

**Please note that if you did not use the integration names specified in step 2, you will need to replace `dr_github`, `drship_gcr` and `drship_gke` with your integration names from `https://app.shippable.com/subs/github/[Subscription name]/integrations`**

###Step 4: Import config into shippable

Follow these instructions to add your config to Shippable: [Adding a syncRepo](http://rcdocs.shippable.com/platform/tutorial/workflow/crud-syncrepo/)

To see a graphical view of your workflow, [click on the **eye** icon on your Subscription Dashboard to view your Single Pane of Glass](http://rcdocs.shippable.com/platform/visibility/subscription/dashboard/).

### Step 5: Run the build_image job

Right click on the `build_image` job in your SPOG view and click on `Build`. This will trigger your workflow and deploy application to with the test labels. Deploy to production is manual and you can right click on the `deploy_prod` job to run it and deploy to prod.
