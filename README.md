# node-hello-world


Let’s step by step implement continuous deployment in Google Cloud Platform using Cloud Build & Cloud Run.



### Pre-requisites

- *Google Cloud Platform console account with an active billing account.*
- *Basic knowledge of Docker & Node.js.*
- *Familiar with Google Cloud Platform console.*
- *A GitHub account*

### 1. Create a hello-world node.js application

To make the guidelines simple, we will create a hello-world node.js  application with source codes as below using the Express framework  (refer: [Express "Hello World" example](https://expressjs.com/en/starter/hello-world.html)).

```javascript
const express = require('express')
const app = express()
const port = 8080

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})
```

Next, let’s test our application locally using the below commands:

```ruby
$ npm init
$ npm install express
$ node app.js
```

![image](https://kodekloud.com/community/uploads/db1265/original/3X/5/4/54a6849f3f7e0268cd73e356bd49fe5b89807851.png)

After starting successfully, verify the app at [http://localhost:8080/](http://localhost:8080/) (make sure port 8080 is free before running our application). You are  supposed to see the “Hello World” message on the application response.

### 2. Dockerize our application

To dockerize our node.js application, first, create a Dockerfile at  the same level as app.js. Then modify the content of the Dockerfile as  below:

```graphql
FROM node:16-alpine

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source, copy everything in current folder to /usr/src/app (workdir)
COPY . .

EXPOSE 8080
CMD [ "node", "app.js" ]
```

Next, let’s build our docker image and run it container.

```python
Build docker image with tag node-hello-world:latest
$ docker build -t node-hello-world:latest .

Run docker image and map port 8080 from host network to port 8080 in container
$ docker container run -dt --name node-hello-world -p 8080:8080 node-hello-world
```

Congrats, you’re successfully dockerized your node.js application and run it, verify the result at [http://localhost:8080/](http://localhost:8080/), you should also see the “Hello World” response message.

Add a .gitignore with the content below to ignore the *node_modules* folder before pushing codes to the repository.

```undefined
node_modules/*
```

To host our source codes on GitHub, I created a private repository and push all the codes so far to the *master* branch using the below commands:

```ruby
$ git init
$ git remote add origin git@github.com:{your_github_name}/node-hello-world.git
$ git add .
$ git commit -m 'init project'
$ git push --set-upstream origin master
```

### 3. Push Docker image to GCP Container Artifact

In this demo, we will use the GCP Artifact Registry to store our  application docker image, more about the Artifact Registry service can  be found here: [Artifact Registry  | Google Cloud 1](https://cloud.google.com/artifact-registry)

Navigate to Artifact Registry at [https://console.cloud.google.com/artifacts 1](https://console.cloud.google.com/artifacts) and create a new repository (click the “CREATE REPOSITORY” button and fill in as below image).

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/9/f/9f2db0cea0b95be30bf2d5a85d37cf2e102318d4_2_351x499.png)

After creating successfully, navigate to the repository detail page  and click “SETUP INSTRUCTIONS” to get the auth commands, copy it, and  run it on your terminal/console (make sure you have *gcloud* installed).

**This is reference: https://cloud.google.com/artifact-registry/docs/docker/store-docker-container-images?hl=zh-CN#tag**

```shell
gcloud auth configure-docker asia-southeast1-docker.pkg.dev

Tag our docker image follow this syntax: docker tag SOURCE-IMAGE LOCATION-docker.pkg.dev/PROJECT-ID/REPOSITORY/IMAGE

$ docker tag node-hello-world:latest us-central1-docker.pkg.dev/<PROJECT-ID>/quickstart-docker-repo1/quickstart-image:tag1

Push our docker image to Artifact Registry follow this syntax: docker push LOCATION-docker.pkg.dev/PROJECT-ID/REPOSITORY/IMAGE

$ docker push us-central1-docker.pkg.dev/<PROJECT-ID>/quickstart-docker-repo1/quickstart-image:tag1
```



![image](https://kodekloud.com/community/uploads/db1265/original/3X/c/8/c8bf6417a8da5a2d5f349edc4d7490653f8841a1.png)

### 4. Provision Cloud Run service to run our docker container

There are many options to run containers on GCP such as App Engine  (flexible environment) or GKE. To keep the demo simple, we will use  Cloud Run (a fully managed service to run the containerized applications on GCP), more detail can be found here: [Cloud Run: Container to production in seconds  | Google Cloud](https://cloud.google.com/run).

Navigate to the Cloud Run dashboard at https://console.cloud.google.com/run and create a new service (click the “CREATE SERVICE” button).



![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/b/b/bb708ed9b514d823e59b5c1b17531c664693fbdd_2_342x500.png)

Wait for GCP a few minutes to create our service, after successfully  creating, the result will appear as below, we can also verify our  application via application URL.

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/6/a/6a60c8ff24ac5f7f49a3cb9446566865fd443f69_2_690x388.png)

Copy the application URL and try it on your browser, you should be able to see a well-known “Hello World” response message.

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/2/7/27ab5f6bc464b8b467811eeb776f5232b184b160_2_690x391.png)

### 5. Setup continuous deployment using GCP Cloud Build

In this final step, we will use Cloud Build (a serverless CI/CD  platform from GCP), to implement continuous deployment for our  application.

Our targets are: whenever new changes are pushed to the master branch on GitHub, Cloud Build will be triggered, it will build a new docker  image to reflect the changes, push to Artifact Registry and deploy this  new version to Cloud Run service.

Navigate to the Cloud Build trigger dashboard and create a new trigger (https://console.cloud.google.com/cloud-build/triggers).

First, we need to connect Cloud Build with our repository on GitHub,  from the trigger dashboard click “Connect Repository” to start the  processes.

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/5/3/53ab3df41990f1c59a6b6641e9dd9b82e4a76db6_2_417x500.png)

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/b/5/b5b6598a3ede8b287150ce297b7b19056da79dbd_2_647x500.png)

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/5/c/5ca808c5a6d9a97c4ea4087dc9ec07980c37c7cc_2_647x500.png)

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/7/6/7635845b9c3d0c898437db4bc4c81facac921cbb_2_423x500.png)

Next, we will create a trigger to implement our continuous deployment strategy.

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/f/c/fce50642aadb8da72bdab5dc6f18ac976c04b62c_2_334x500.png)

We will use an inline YAML file to define how our trigger runs, you  don’t need to remember this complex syntax, GCP has an official document which we can refer from: [Deploying to Cloud Run using Cloud Build  | Cloud Build Documentation  | Google Cloud](https://cloud.google.com/build/docs/deploying-builds/deploy-cloud-run).

> You also can select Location-Repository ,And use local **cloudbuild.yaml**

```bash
steps:
  - name: gcr.io/cloud-builders/docker
    args:
      - build
      - '-t'
      - '${_LOCATION}-docker.pkg.dev/$PROJECT_ID/${_REPOSITORY}/${_IMAGE}:$COMMIT_SHA'
      - .
  - name: gcr.io/cloud-builders/docker
    args:
      - push
      - '${_LOCATION}-docker.pkg.dev/$PROJECT_ID/${_REPOSITORY}/${_IMAGE}:$COMMIT_SHA'
  - name: gcr.io/google.com/cloudsdktool/cloud-sdk
    args:
      - run
      - deploy
      - $_SERVICE_NAME
      - '--image'
      - '${_LOCATION}-docker.pkg.dev/$PROJECT_ID/${_REPOSITORY}/${_IMAGE}:$COMMIT_SHA'
      - '--region'
      - $_REGION
    entrypoint: gcloud
images:
  - '${_LOCATION}-docker.pkg.dev/$PROJECT_ID/${_REPOSITORY}/${_IMAGE}:$COMMIT_SHA'
options:
  logging: NONE
```

In the YAML file, we used 5 variables which are *_SERVICE_NAME* (name of cloud run service) and *_REGION* (where your cloud run service runs), _IMAGE (name of docker image),  _LOCATION (where’s your artifact registry), _REPOSITORY (name of  artifact registry repository). _LOCATION & _REGION may look  duplicated but they are target different objects (artifact repo &  cloud run service).

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/4/5/45d042f400848aff4836240f46115f166896325e_2_405x500.png)

```
_IMAGE         node-hello-world
_LOCATION      us-central1
_REGION        us-central1
_REPOSITORY    node-hello-world
_SERVICE_NAME  node-hello-world
```

**We need to do 2 things:**

- Click Open Editor and paste the content of the above YAML to the editor, then save.
- Add 5 variables like the above image (_IMAGE, _LOCATION, _REPOSITORY, _SERVICE_NAME & _REGION).

Then click Create button to create the trigger and observe the  result. After the trigger is created successfully, go to your code and  change the message from “Hello World” to “Hello World Version 02” and  push the change to the master branch ( *it is OK with the demo, but in the real scenario, we should go through the pull request and code review process for any changes* ).

If your cloud build agent faced the permission issue while deploying  the new version to the Cloud Run service, go to the IAM dashboard, look  for @cloudbuild.gserviceaccount.com (auto-generated by GCP), and grant appropriate permission.

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/d/1/d14ce3651f25dbd222867c7e38a70819ce780868_2_690x388.png)

![image](https://kodekloud.com/community/uploads/db1265/optimized/3X/b/d/bd068fddeea68897c7b666fe961fcff0e41a3ffc_2_690x140.png)

Go to the Cloud Run service URL again and verify the changes, it should now return a new response as “Hello World Version 02”.

### Conclusions

In this post, we become familiar with the following services in GCP: Cloud Build, Cloud Run, and Artifact Registry.

We went through 5 steps to create a node.js application, dockerize it and push it to the GitHub repository, then push it to Artifact  Registry, host it with Cloud Run, and finally set up continuous  deployment with Cloud Build.

> reference:
>
> https://kodekloud.com/community/t/continuous-deployment-on-google-cloud-platform-with-cloud-build-and-cloud-run/232513



