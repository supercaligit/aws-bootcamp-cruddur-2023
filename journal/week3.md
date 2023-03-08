# Week 3 — Decentralized Authentication


## Required Homework

### Set up Cognito User Pool
In AWS Console create the Coginot User Pool
User Pool - when you want to manage users in your app
Federated Identity Pool - when you use third party Identity

### AWS Amplify
We need to install Amplify to use the javascript library for AWS Cognito.
https://docs.amplify.aws/lib/auth/getting-started/q/platform/js/

1. **Install AWS Amplify**
    install in `frontend-react-js` directory
    --save adds it to the package.json because we need it in production
    ```py
    cd frontend-react-js/
    npm i aws-amplify --save
```
2. **Configure Amplify**

We need to hook up our cognito pool to our code in the `App.js`
The env vars with REACT_APP_ are named because React expects that and will autoload these. 
```js
import { Amplify } from 'aws-amplify';

Amplify.configure({
  "AWS_PROJECT_REGION": process.env.REACT_APP_AWS_PROJECT_REGION,
  "aws_cognito_identity_pool_id": process.env.REACT_APP_AWS_COGNITO_IDENTITY_POOL_ID,
  "aws_cognito_region": process.env.REACT_APP_AWS_COGNITO_REGION,
  "aws_user_pools_id": process.env.REACT_APP_AWS_USER_POOLS_ID,
  "aws_user_pools_web_client_id": process.env.REACT_APP_CLIENT_ID,
  "oauth": {},
  Auth: {
    // We are not using an Identity Pool
    // identityPoolId: process.env.REACT_APP_IDENTITY_POOL_ID, // REQUIRED - Amazon Cognito Identity Pool ID
    region: process.env.REACT_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
    userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
    userPoolWebClientId: process.env.REACT_APP_AWS_USER_POOLS_WEB_CLIENT_ID,   // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
  }
});

```

### Implement Custom Sign-In Page
### Implement Custom Sign-Up Page
### Implement Custom Confirmation Page
### Implement Custom Recovery Page






## Homework Challenges
