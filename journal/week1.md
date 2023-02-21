# Week 1 — App Containerization


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

