---
sidebar: auto
---

# Screenlife Analytics - handover documentation

To the new person taking over this project, good luck with your work and sorry for the mess. I took a messy open source codebase, and added more mess to the pile. So that you won't have to waste a month figuring out what the hell is all this doing, like I did, this is a comprehensive guide for the Screenlife Analytics platform, including a clear explanation of the codebase.

I hope this guide will be useful for you, and once you're done with the project, add on to this for archive. The world needs more clear and usable documentations.

## Quick guide

The repo for the Analytics app is found [here](https://github.com/ScreenLife-Capture-Team/screenlife-analytics/tree/dev). My last update was on the dev branch, but as of now it probably can be merged back into main.

### Running steps

The steps for installing and using the app can be found in the readme on dev branch

## Code breakdown: Deployment
### Compose file
For building purposes, thus far I've only used `docker-compose.dev.yml`. You will see in the compose file, under `services`, there are 5 images to be deployed. The compose file will read the Dockerfiles at each location to build the images.
#### Front end
`annotator_client` is of course the client facing side. Code for all the frontend stuff is in the `client` directory. The client has another Dockerfile inside to be used for deployment at `./client/Dockerfile`. We'll get to that later below.
#### Backend
`annotator_webserver` is the Flask backend server. It also has a Dockerfile to be read at `./backend/webserver/Dockerfile`.
#### Workers
`annotator_workers` is a server to manage workers, to perform asynchronous tasks. These workers will be handling a majority of API calls, interacting with the database. Its Dockerfile is at `./backend/workers/Dockerfile`
#### Database
This image uses the stock Mongo image from Docker to build. It does not have a Dockerfile.

However, note that **this image is not where data is stored**. Data will be stored in a volume called `mongodb_data`.
#### Messageq
Built using the stock RabbitMQ image. This one is mostly stable, it should be left alone unless you need to upgrade/downgrade RabbitMQ.

### Individual Dockerfiles

#### Client
Our client image is built upon Node 10, using Vue-cli 3.4.0. The client access point for public access or dev access is at port 8080. 

But also, take a look at `vue.config.js`. I've set `publicPath` to screenlife-analytics. That means, access to client will be at `<ip_address>:8080/screenlife-analytics`. After deploying locally, you will access it at `localhost:8080/screenlife-analytics/`

#### Backend
This file has quite a lot to look at. Our starting point is the GPU enabled Tensorflow image `tensorflow/tensorflow:1.15.4-gpu-py3`. After that, it's a matter of installing a ton of dependencies before we can start. I highly suggest you should save this image and upload to Dockerhub for future use, otherwise it takes time to rebuild every time you start fresh.

Backend is exposed at port 8081. In the init file for our Flask server (`backend/webserver/api`), I've prefixed the URL for API endpoint to be `api-screenlife`. Again, look at the `proxy` section in `Vue.config.js`. I've set `:8081/api-screenlife` to `/api-screenlife`. That means, `:8080/api-screenlife` will also point at the server access point.

#### Workers
The workers image is also built upon `tensorflow/tensorflow:1.15.4-gpu-py3`. Workers image endpoint is at port 5555.

## Code breakdown: Client
Now, we're taking a dive into the fuckfest that is `./client`

### Entry point
As usual, our entry point is the `App.vue` file. You'll notice that it only has two components: the Navbar, and a RouterView. Every web content we see on the page is within this RouterView component.

### Router
The `router.js` is, as usual, where the index for the pages are. They are all located under `./client/src/views`

Note the `requiresAuth: true` for most routes. This is to ensure no unauthorized access to any component page, unless the user has already logged in.

The auth and rerouting process can be found in `main,js` in the `router.beforeEach` definition.

### Views

#### About
This page is meant to be an FAQ for the webapp, but for now it's blank. You can build this up after you're done fully developing the app.

#### AdminPanel
In the Navbar component, you will see that this page's access is restricted only to admin. This is where the admin can manage users, including creating and granting access.

#### Annotator
The page for image segmentation. Also the most goddamn annoying page of all.

#### Auth
The login page. Only shown upon first accessing the page. This is where, if authentication guard triggers, the user will be sent to.

#### Categories
This is a leftover from the original authors, and for the sake of avoiding confusion, I've purposely hidden it from normal users. It displays all the categories that has been created. 

Because of how the deletion works in this app, removing a category doesn't actually erase it from the database. I've developed the workflow for users to add/remove categories in the annotation pages itself. Long story short, this page is somewhat redundant now.

#### Dataset
Not to be confused with Datasets.

This is the most extensive one, which is the page that shows after you click on a dataset from the home page. In this page, navigation is done using the variable `tab`. Changing it will display the corresponding `div` in the HTML code.

- `batchtag`: The default tab. This is where batch annotation is performed.
- `images`: This is for selecting individual images for image segmentation task.
- `exports`: For user to download the exported data or tagsets.
- `analytics`: The dashboard where data visualization is shown.
- `searchbar`: Search function.

#### Datasets
Not to be confused with Dataset.

The de facto homepage of the webapp. This is where user sees the datasets that's been assigned to them. Clicking on one will go to `Dataset.vue`.

#### Help
My attempt at a FAQ page, which wasn't finished. 

#### Home
Redundant page. Pretty much can ignore

#### PageNotFound
The 404 page

#### Tasks
This one is also only visible to admin. This is where the workers will report their task progress when summoned.

It's no longer useful for average users, but a great way to help you debug issues with workers.

#### Undo
A legacy from the original authors. This is supposed to be where you can remove data from the database entirely. I admit I haven't really looked into it; but you can if it helps in future development.

#### User
For users to update their profile and password.

## Code breakdown: How an API call is handled, from client to backend

When a user does any action on the client, here is the long winded ass process of how methods are called

### Predefined Axios requests
Axios is there to help you with constructing HTTP requests.

Predefined HTTP requests can be found in `./client/src/models`.  Each of them is dedicated for an API, which can be found at `./backend/webserver/api`.

You can of course use Axios on its own to send custom HTTP requests within the Vue pages/components.

### The process

#### Sending the request

HTTP request is sent from client to webserver when you either use Axios raw to build a request and send, or trigger a function call in one of the `models` files listed above. 

#### Receiving the request
Receiving request is handled by the APIs in Flask (found at `./backend/webserver/api`). 

Some simple ones will get a return immediately in the API code. Other requests, particularly requests regarding reading/modifying database might need to trigger more layers of function calls.

#### Database code
These code are found in `./backend/database`. They are class objects that inherit MongoEngine's `DynamicDocument` class. Read [here](https://docs.mongoengine.org/guide/defining-documents.html) to understand more about MongoEngine's classes if you're not yet familiar.

Essentially these classes represent data tables in MongoDB, where the class attributes are fields in the database. Modification of the DB is carried out through class methods.

Tasks that take long to perform will be handed over to workers, to perform asynchronously in the background.

#### Workers and tasks
We're using Celery as the worker manager, with RabbitMQ being the task broker.

Workers can also call database code (DynamicDocument class objects) to perform database operations.

It is important to remember that workers are meant to be working in the background. Therefore, API calls that spawn workers should not await the worker's result. In those case, ensure that:
-  the database class methods that spawn workers should return the task's metadata (task ID, task name etc)
- the backend API should expect a return of that too
- API should respond to client similarly with task metadata, or a simple acknowledgement. 

To see a good example of how this process goes, take a look at `./backend/webserver/api/datasets.py`, in the `DatasetExport` class (route `/<int:dataset_id>/export`)

### Debugging database operations/API backend

Note that in a Docker environment, `print` statements in Flask and MongoEngine code won't display to CLI. For that, you need to use the Gunicorn `logger`, as defined for example in `./backend/webserver/__init__.py` or `./backend/database/__init__.py`.