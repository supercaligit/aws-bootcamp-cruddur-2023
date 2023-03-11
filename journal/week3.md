# Week 3 — Decentralized Authentication


## Required Homework

### Set up Cognito User Pool
In AWS Console create the Coginot User Pool
User Pool - when you want to manage users in your app
Federated Identity Pool - when you use third party Identity
- Authentication providers - 
  - Provider types- select `Cognito User Pools`
  - Cognito user pool sign-in options - select `Email` only
- Password policy
  - Password policy mode - select `Cognito defaults`
  - Multi Factor Authentication - select `No MFA`
  - User account recovery - leave default
- Configure sign-up experience
  - Self-service sign-up - leave default
  - Attribute verification and user account confirmation - leave default
  - Required attributes - select `name` and `preferred_username`
- Email
  - Email provider - select `Send email with Cognito`
- Integrate your app
  - User pool name - enter `cruddur-user-pool`
  - App client name - enter `cruddur`


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
        region: process.env.REACT_APP_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
        userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
        userPoolWebClientId: process.env.REACT_APP_CLIENT_ID,   // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
    }
    });

    ```
3. **Conditionally show components based on logged in or logged out**
    Inside our `HomeFeedPage.js`

```
import { Auth } from 'aws-amplify';

// set a state
const [user, setUser] = React.useState(null);

// check if we are authenicated
const checkAuth = async () => {
  Auth.currentAuthenticatedUser({
    // Optional, By default is false. 
    // If set to true, this call will send a 
    // request to Cognito to get the latest user data
    bypassCache: false 
  })
  .then((user) => {
    console.log('user',user);
    return Auth.currentAuthenticatedUser()
  }).then((cognito_user) => {
      setUser({
        display_name: cognito_user.attributes.name,
        handle: cognito_user.attributes.preferred_username
      })
  })
  .catch((err) => console.log(err));
};

// check when the page loads if we are authenicated
React.useEffect(()=>{
  loadData();
  checkAuth();
}, [])
```

We'll update ProfileInfo.js
```
import { Auth } from 'aws-amplify';

const signOut = async () => {
  try {
      await Auth.signOut({ global: true });
      window.location.href = "/"
  } catch (error) {
      console.log('error signing out: ', error);
  }
}
```
5. **Implement Custom Signin Page**
    ```
    import { Auth } from 'aws-amplify';

    const onsubmit = async (event) => {
        setCognitoErrors('')
        event.preventDefault();
        try {
        Auth.signIn(email, password)
            .then(user => {
            localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
            window.location.href = "/"
            })
            .catch(err => { console.log('Error!', err) });
        } catch (error) {
        if (error.code == 'UserNotConfirmedException') {
            window.location.href = "/confirm"
        }
        setErrors(error.message)
        }
        return false
    }
    ```
6.  **Manually add user to the Cognito user pool and setup a temp password.When you login you will get "Cannot read properties pf null(reading'accessToken')".The user created is in "password change pending" state so permanently rest password uding from CLI command.**
    ```sh
    aws cognito-idp admin-set-user-password \
      --user-pool-id <your-user-pool-id> \
      --username <username> \
      --permanent
      --password <password> \
    ```
7. **Confirm Custom Sign-In Page Works**

8. **Implement Custom Sign-Up Page**
    ```
    import { Auth } from 'aws-amplify'; //replace import Cookies from 'js-cookie'

    const onsubmit = async (event) => {
      event.preventDefault();
      setErrors('')
      try {
          const { user } = await Auth.signUp({
            username: email,
            password: password,
            attributes: {
                name: name,
                email: email,
                preferred_username: username,
            },
            autoSignIn: { // optional - enables auto sign in after user is confirmed
                enabled: true,
            }
          });
          console.log(user);
          window.location.href = `/confirm?email=${email}`
      } catch (error) {
          console.log(error);
          setErrors(error.message)
      }
      return false
    }

    ```
9. #Implement Custom Confirmation Page#
    ```
    import { Auth } from 'aws-amplify'; //replace import Cookies from 'js-cookie'

    const resend_code = async (event) => {
      setCognitoErrors('')
      try {
        await Auth.resendSignUp(email);
        console.log('code resent successfully');
        setCodeSent(true)
      } catch (err) {
        // does not return a code
        // does cognito always return english
        // for this to be an okay match?
        console.log(err)
        if (err.message == 'Username cannot be empty'){
          setErrors("You need to provide an email in order to send Resend Activiation Code")   
        } else if (err.message == "Username/client id combination not found."){
          setErrors("Email is invalid or cannot be found.")   
        }
      }
    }

    const onsubmit = async (event) => {
      event.preventDefault();
      setErrors('')
      try {
        await Auth.confirmSignUp(email, code);
        window.location.href = "/"
      } catch (error) {
        setErrors(error.message)
      }
      return false
    }
    ```
10. **Implement Custom Recovery Page**
    ```
    import { Auth } from 'aws-amplify';

    const onsubmit_send_code = async (event) => {
      event.preventDefault();
      setErrors('')
      Auth.forgotPassword(username)
      .then((data) => setFormState('confirm_code') )
      .catch((err) => setErrors(err.message) );
      return false
    }

    const onsubmit_confirm_code = async (event) => {
      event.preventDefault();
      setErrors('')
      if (password == passwordAgain){
        Auth.forgotPasswordSubmit(username, code, password)
        .then((data) => setFormState('success'))
        .catch((err) => setErrors(err.message) );
      } else {
        setErrors('Passwords do not match')
      }
      return false
    }
    ```





## Homework Challenges
