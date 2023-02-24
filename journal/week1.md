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
Create a new file script.sh copy following in it
```sh
python3 -m flask run --host=0.0.0.0 --port=4567
```

Modify the Docker file in backend-flask, replace CMD command with lines below
```sh
#CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]

COPY script.sh script.sh
RUN chmod +x script.sh
ENTRYPOINT ./script.sh
```

Build the docker file
```sh
dockebuild -t  backend-flask ./backend-flask
```

Run the docker file
```sh
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
```

2. Push and tag an image to DockerHub (they have a free tier)
```sh
docker images

docker login --username={username} --password={password}

REPOSITORY                                    TAG         IMAGE ID       CREATED         SIZE
aws-bootcamp-cruddur-2023-frontend-react-js   latest      88c505216b54   2 minutes ago   1.19GB
aws-bootcamp-cruddur-2023-backend-flask       latest      cee0217b9b42   3 minutes ago   129MB
postgres                                      13-alpine   55f14697b527   12 days ago     238MB
amazon/dynamodb-local                         latest      904626f640dc   3 weeks ago     499MB


docker tag cee0217b9b42 supercali/aws-bootcamp-cruddur-2023:backend
docker tag 88c505216b54 supercali/aws-bootcamp-cruddur-2023:frontend
docker tag 55f14697b527 supercali/aws-bootcamp-cruddur-2023:postgres
docker tag 904626f640dc supercali/aws-bootcamp-cruddur-2023:dynamodb


docker push supercali/aws-bootcamp-cruddur-2023:backend
docker push supercali/aws-bootcamp-cruddur-2023:frontend
docker push supercali/aws-bootcamp-cruddur-2023:postgres
docker push supercali/aws-bootcamp-cruddur-2023:dynamodb
```

Ref: https://jsta.github.io/r-docker-tutorial/04-Dockerhub.html
![PushImageDockerHub](/journal/images/Week1-PushImageDockerHub.png)

3. Learn how to install Docker on your localmachine and get the same containers running outside of Gitpod / Codespaces
This was definitely a reach attempt today. It took a lot of trial and error to figure out why the container was not running locally but I was finally able to get it working. 
![PushImageDockerHub](/journal/images/Week1-RunDockerImagesLocalMachine.png)
![PushImageDockerHub](/journal/images/Week1-DockerDesktop-Containers.png)
![PushImageDockerHub](/journal/images/Week1-RunningBackEndLocalMachine.png)

4. Use multi-stage building for a Dockerfile build
5. Implement a healthcheck in the V3 Docker compose file
6. Research best practices of Dockerfiles and attempt to implement it in your Dockerfile
7. Launch an EC2 instance that has docker installed, and pull a container to demonstrate you can run your own docker processes. 

## Misc Notes

### Backend build and run

### Run Python

Run standalone outside of Docker
```sh
cd backend-flask
pip3 install -r requirements.txt
python3 -m flask run --host=0.0.0.0 --port=4567
cd ..
```

- make sure to unlock the port on the port tab
- open the link for 4567 in your browser
- append to the url to `/api/activities/home`
- you should get back json


Build backend using Docker
```sh
docker build -t  backend-flask ./backend-flask
```

Run backend using Docker
```sh
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask

export FRONTEND_URL="*"
export BACKEND_URL="*"
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
unset FRONTEND_URL="*"
unset BACKEND_URL="*"

OR

docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
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

