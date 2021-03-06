# Deploying a Flask API

This is the project starter repo for the fourth course in the [Udacity Full Stack Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004): Server Deployment, Containerization, and Testing.

In this project you will containerize and deploy a Flask API to a Kubernetes cluster using Docker, AWS EKS, CodePipeline, and CodeBuild.

The Flask app that will be used for this project consists of a simple API with three endpoints:

- `GET '/'`: This is a simple health check, which returns the response 'Healthy'. 
- `POST '/auth'`: This takes a email and password as json arguments and returns a JWT based on a custom secret.
- `GET '/contents'`: This requires a valid JWT, and returns the un-encrpyted contents of that token. 

The app relies on a secret set as the environment variable `JWT_SECRET` to produce a JWT. The built-in Flask server is adequate for local development, but not production, so you will be using the production-ready [Gunicorn](https://gunicorn.org/) server when deploying the app.

## Initial setup
1. Fork this project to your Github account.
2. Locally clone your forked version to begin working on the project.

## Dependencies

- Docker Engine
    - Installation instructions for all OSes can be found [here](https://docs.docker.com/install/).
    - For Mac users, if you have no previous Docker Toolbox installation, you can install Docker Desktop for Mac. If you already have a Docker Toolbox installation, please read [this](https://docs.docker.com/docker-for-mac/docker-toolbox/) before installing.
 - AWS Account
     - You can create an AWS account by signing up [here](https://aws.amazon.com/#).
     
## Project Steps

Completing the project involves several steps:

1. Write a Dockerfile for a simple Flask API

2. Build and test the container locally

   ```
   docker build --tag jwt-api-test .
   docker run --env-file=env_file -p 80:8080 jwt-api-test
   
   // To try the /auth endpoint, use the following command:
   export TOKEN=`curl -d '{"email":"sl2kd@virginia.edu","password":"Lsm,5036"}' -H "Content-Type: application/json" -X POST localhost:80/auth  | jq -r '.token'`
   
   echo ${TOKEN}
   
   // To try the /contents endpoint which decrypts the token and returns its content, run:
   curl --request GET 'http://127.0.0.1:80/contents' -H "Authorization: Bearer ${TOKEN}" | jq .
   
   ```

3. Create an EKS cluster

   ```
   eksctl create cluster --name simple-jwt-api
   ```

4. Store a secret using AWS Parameter Store

5. Create a CodePipeline pipeline triggered by GitHub checkins

6. Create a CodeBuild stage which will build, test, and deploy your code

For more detail about each of these steps, see the project lesson [here](https://classroom.udacity.com/nanodegrees/nd004/parts/1d842ebf-5b10-4749-9e5e-ef28fe98f173/modules/ac13842f-c841-4c1a-b284-b47899f4613d/lessons/becb2dac-c108-4143-8f6c-11b30413e28d/concepts/092cdb35-28f7-4145-b6e6-6278b8dd7527).

### Result IP

```
$ kubectl get services simple-jwt-api -o wide
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP                                                               PORT(S)        AGE   SELECTOR
simple-jwt-api   LoadBalancer   10.100.112.97   ac23d40b46da411ea9d850a7c07a9b1e-1675096220.us-west-2.elb.amazonaws.com   80:30849/TCP   77m   app=simple-jwt-api

$ export TOKEN=`curl -d '{"email":"<EMAIL>","password":"<PASSWORD>"}' -H "Content-Type: application/json" -X POST ac23d40b46da411ea9d850a7c07a9b1e-1675096220.us-west-2.elb.amazonaws.com/auth  | jq -r '.token'`

$ curl --request GET 'ac23d40b46da411ea9d850a7c07a9b1e-1675096220.us-west-2.elb.amazonaws.com/contents' -H "Authorization: Bearer ${TOKEN}" | jq 

{
  "email": "<EMAIL>",
  "exp": 1586250754,
  "nbf": 1585041154
}

```

