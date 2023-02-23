# Week 1 — App Containerization

## Homework Tasks
1. Create a docker file for front end and back end and ensure it can run individually

2. Create a docker-compose file, which will combine the two docker files and launch both front end and back end. 
![Frontend](/journal/images/Week1-FrontEndpoint.png)

![Backend](/journal/images/Week1-BackEndpoint.png)

3. OpenAPI (ref:readme.com)
![OpenAPI](/journal/images/Week1-OpenAPI.png)

4. Notifications Backend Endpoint
![NotificationsBack](/journal/images/Week1-NotificationsEndpoint.png)

5. Notifications Front end React page
![NotificationsPage](/journal/images/Week1-NotificationsPage.png)

6. Run DynamoDB Local Container and ensure it works

7. Run Postgres Container and ensure it works
![Postgres](/journal/images/Week1-PostgresInstall.png)

## Homework Challenges
1. Run the dockerfile CMD as an external script
2. Push and tag a image to DockerHub (they have a free tier)
3. Use multi-stage building for a Dockerfile build
4. Implement a healthcheck in the V3 Docker compose file
5. Research best practices of Dockerfiles and attempt to implement it in your Dockerfile
6. Learn how to install Docker on your localmachine and get the same containers running outside of Gitpod / Codespaces
7. Launch an EC2 instance that has docker installed, and pull a container to demonstrate you can run your own docker processes. 

## Misc Notes

### Backend build and run

Build backend
```sh
docker build -t  backend-flask ./backend-flask
```

Run backend
```sh
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask

export FRONTEND_URL="*"
export BACKEND_URL="*"
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
unset FRONTEND_URL="*"
unset BACKEND_URL="*"
```

Run in background
```sh
docker container run --rm -p 4567:4567 -d backend-flask
```

URL
```sh
/api/activities/home
```

### Front end build and run

We have to run NPM Install before building the container since it needs to copy the contents of node_modules

```
cd frontend-react-js
npm i
```

Build front end
```sh
docker build -t frontend-react-js ./frontend-react-js
```

Run front end
```sh
docker run -p 3000:3000 -d frontend-react-js
```

