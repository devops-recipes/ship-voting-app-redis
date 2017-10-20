# Shippable Voting App

This sample creates a multi-container application in a GKE Kubernetes cluster. The application interface has been built using Python / Flask. The data component is using Redis.

## Shippable workflow

The workflow in this sample application is multi-stage deployments into Test and Prod environments.

![Shippable Assembly Lines workflow for a multi-stage deployment to Kubernetes on GKE](https://user-images.githubusercontent.com/2983749/31842893-3043b8c6-b5a5-11e7-854b-14cb112659ab.png)

The challenge with this is that the configuration of the app changes in each environment. You therefore need a config file that is specific to application/service/microservice and the environment! This creates a **new DevOps matrix from hell**  which is explained in detail in [our blog](http://blog.shippable.com/the-new-devops-matrix-from-hell).

With Shippable, you just need one Kubernetes template spec and you have the ability to replace environment specific values at runtime during deployment. This avoids the matrix from hell and creates an Assembly Line for your application as it moves from code to test to prod.

## Pre-requisites

You need the following in order to run this sample:

- A Google Cloud account at https://cloud.google.com/
  * A Project that has Google Container Engine (GKE) and Google Container Registry (GCR) enabled
  * An [existing Kubernetes cluster](https://cloud.google.com/container-engine/docs/clusters/operations) on GKE where you will deploy this sample application. If you want to do this programmatically using Shippable, [check out our docs on this topic](http://docs.shippable.com/provision/kubernetes-with-gkecli/)
  * [APIs should be enabled](https://support.google.com/cloud/answer/6158841?hl=en)
- A GitHub account where you will fork and run this sample
- [Sign in with GitHub](https://app.shippable.com) to create a Shippable account

If you're not familiar with Shippable, it is also recommended that you read the [Platform overview docs](http://docs.shippable.com/platform/overview/) to understand the overall structure of Shippable's DevOps Assembly Lines platform.

## Instructions

### Step 1: Fork the application

Fork this repository to your GitHub account.

### Step 2: Create integrations

Since your deployment config will interact with GCR to push the front-end image and GKE to deploy the application, you will need to create integrations for these in the Shippable UI.

Follow steps in **Adding an account integration** section to create these integrations:
- [Create a GCR integration](http://docs.shippable.com/platform/integration/gcr/) called `drship_gcr`
- [Create a GKE integration](http://docs.shippable.com/platform/integration/gke/) called `drship_gke`

You will also need to [create a GitHub integration](http://docs.shippable.com/platform/integration/github/) to access contents of your repository. Make sure the friendly name of this integration is `dr_github`.

### Step 3: Edit the config

Edit the following files:

In `shippable.resources.yml`:
- In the `ship_voting_app_redis_gitRepo` resource definition, replace `devops-recipes/ship-voting-app-redis` with the name of your forked repo
- In the `front_img` resource definition, replace `gcr.io/fair-future-183201/vote` with `gcr.io/YOUR-GCLOUD-PROJECT-ID/vote`. To find the project ID, [follow instructions here](https://support.google.com/cloud/answer/6158840?hl=en)
- In `kube_cluster`, replace the value of `sourceName` and `region` with the name of your GKE cluster name and region.

In `shippable.jobs.yml`:
- In the `build_frontend_imag` job definition, replace `gcr.io/fair-future-183201/vote` with `gcr.io/YOUR-GCLOUD-PROJECT-ID/vote`. To find the project ID, [follow instructions here](https://support.google.com/cloud/answer/6158840?hl=en)

### Step 4: Import config into shippable

Follow these instructions to add your config to Shippable: [Adding a syncRepo](http://docs.shippable.com/platform/tutorial/workflow/crud-syncrepo/)

To see a graphical view of your workflow, [click on the **eye** icon on your Subscription Dashboard to view your Single Pane of Glass](http://docs.shippable.com/platform/visibility/subscription/dashboard/).

### Step 5: Run the build_image job

Right click on the `build_image` job in your SPOG view and click on `Build`. This will trigger your workflow and deploy application to with the test labels. Deploy to production is manual and you can right click on the `deploy_prod` job to run it and deploy to prod.

## Understanding the sample

Here are some links that will help you understand how the sample is structured:

* This sample extensively uses the [shipctl utility](http://docs.shippable.com/platform/tutorial/workflow/using-shipctl/) to perform common operations like:
  * To extract information from IN resources
  * Put information in OUT resources
  * Replacing environment values in the Kubernetes spec template for each environment.

* Reference for jobs and resources used in this sample:
  * `build_job` is a [runSh job](http://docs.shippable.com/platform/workflow/job/runsh/) that builds and pushes the front-end image to GCR
  * `deploy_test` is a [runSh job](http://docs.shippable.com/platform/workflow/job/runsh/) that takes the deployment template `ship_vote_all_in_one_redis.yml.template`, replaces the environment values needed for test deployment, and deploys both `frontend_img` and `redis` images. It also outputs `relstate` which is an updated deployment spec with immutable image tags to test.
  * `deploy_prod` is a [runSh job](http://docs.shippable.com/platform/workflow/job/runsh/) that takes the deployment spec from `relstate`, replaces the labels needed for prod deployment, and deploys both `frontend_img` and `redis` images to production.
  * `test_params` and `prod_params` are [params resources](http://docs.shippable.com/platform/workflow/resource/params/) that contain values for labels for test and prod respectively
  * `kube_cluster` is a [cluster resource](http://docs.shippable.com/platform/workflow/resource/cluster/) that points to your Kubernetes cluster on GKE
  * `relstate` is a [state resource](http://docs.shippable.com/platform/workflow/resource/state/) that hold the immutable release spec
  * `gke_cliConfig` is a [cliConfig resource](http://docs.shippable.com/platform/workflow/resource/cliconfig/) that points to your GKE integration so we can initialize gcloud cli on your behalf
  * `front_img` is an [image resource](http://docs.shippable.com/platform/workflow/resource/image/) represents the Docker image of your front-end service stored in GCR
  * `ship_voting_app_redis_gitRepo` is a [gitRepo resource](http://docs.shippable.com/platform/workflow/resource/gitrepo/) a pointer to your github repository. This is an input to any job that needs content from this repository.
