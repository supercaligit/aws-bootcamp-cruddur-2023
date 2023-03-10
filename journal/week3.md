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
    `--save` adds it to the package.json because we need it in production

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
### UI Changes
1. **Conditionally show components based on logged in or logged out**
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
2. **Implement Custom Signin Page**
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
3.  **Manually add user to the Cognito user pool and setup a temp password.When you login you will get "Cannot read properties pf null(reading'accessToken')".The user created is in "password change pending" state so permanently rest password uding from CLI command.**
    ```sh
    aws cognito-idp admin-set-user-password \
      --user-pool-id <your-user-pool-id> \
      --username <username> \
      --permanent
      --password <password> \
    ```
4. **Confirm Custom Sign-In Page Works**

     ![User signin](/journal/images/Week3-CruddurLogin.png)

5. **Implement Custom Sign-Up Page**
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
6. #Implement Custom Confirmation Page#
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
7. **Implement Custom Recovery Page**
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

### Authenticating Server Side

  1. Add in the `HomeFeedPage.js` a header eto pass along the access token

      ```
      headers: {
        Authorization: `Bearer ${localStorage.getItem("access_token")}`
      }
      ```

 2. In the app.py
    ```py
    cors = CORS(
      app, 
      resources={r"/api/*": {"origins": origins}},
      headers=['Content-Type', 'Authorization'], 
      expose_headers='Authorization',
      methods="OPTIONS,GET,HEAD,POST"
    )
    ...
    ..
    ..
    @app.route("/api/activities/home", methods=['GET'])
    def data_home():
      app.logger.debug('AUTH HEADER---')
      app.logger.debug(request.headers.get('Authorization'))
      data = HomeActivities.run()
      return data, 200


    ```
 3. Use Flask-AWSCognito
    Add to `requirements.txt`
    ```
      Flask-AWSCognito
    ```

  4. Create `cognito_jwt_token.py`
     ```py
      import time
      import requests
      from jose import jwk, jwt
      from jose.exceptions import JOSEError
      from jose.utils import base64url_decode

      class FlaskAWSCognitoError(Exception):
        pass

      class TokenVerifyError(Exception):
        pass

      def extract_access_token(request_headers):
          access_token = None
          auth_header = request_headers.get("Authorization")
          if auth_header and " " in auth_header:
              _, access_token = auth_header.split()
          return access_token

      class CognitoJwtToken:
          def __init__(self, user_pool_id, user_pool_client_id, region, request_client=None):
              self.region = region
              if not self.region:
                  raise FlaskAWSCognitoError("No AWS region provided")
              self.user_pool_id = user_pool_id
              self.user_pool_client_id = user_pool_client_id
              self.claims = None
              if not request_client:
                  self.request_client = requests.get
              else:
                  self.request_client = request_client
              self._load_jwk_keys()


          def _load_jwk_keys(self):
              keys_url = f"https://cognito-idp.{self.region}.amazonaws.com/{self.user_pool_id}/.well-known/jwks.json"
              try:
                  response = self.request_client(keys_url)
                  self.jwk_keys = response.json()["keys"]
              except requests.exceptions.RequestException as e:
                  raise FlaskAWSCognitoError(str(e)) from e

          @staticmethod
          def _extract_headers(token):
              try:
                  headers = jwt.get_unverified_headers(token)
                  return headers
              except JOSEError as e:
                  raise TokenVerifyError(str(e)) from e

          def _find_pkey(self, headers):
              kid = headers["kid"]
              # search for the kid in the downloaded public keys
              key_index = -1
              for i in range(len(self.jwk_keys)):
                  if kid == self.jwk_keys[i]["kid"]:
                      key_index = i
                      break
              if key_index == -1:
                  raise TokenVerifyError("Public key not found in jwks.json")
              return self.jwk_keys[key_index]

          @staticmethod
          def _verify_signature(token, pkey_data):
              try:
                  # construct the public key
                  public_key = jwk.construct(pkey_data)
              except JOSEError as e:
                  raise TokenVerifyError(str(e)) from e
              # get the last two sections of the token,
              # message and signature (encoded in base64)
              message, encoded_signature = str(token).rsplit(".", 1)
              # decode the signature
              decoded_signature = base64url_decode(encoded_signature.encode("utf-8"))
              # verify the signature
              if not public_key.verify(message.encode("utf8"), decoded_signature):
                  raise TokenVerifyError("Signature verification failed")

          @staticmethod
          def _extract_claims(token):
              try:
                  claims = jwt.get_unverified_claims(token)
                  return claims
              except JOSEError as e:
                  raise TokenVerifyError(str(e)) from e

          @staticmethod
          def _check_expiration(claims, current_time):
              if not current_time:
                  current_time = time.time()
              if current_time > claims["exp"]:
                  raise TokenVerifyError("Token is expired")  # probably another exception

          def _check_audience(self, claims):
              # and the Audience  (use claims['client_id'] if verifying an access token)
              audience = claims["aud"] if "aud" in claims else claims["client_id"]
              if audience != self.user_pool_client_id:
                  raise TokenVerifyError("Token was not issued for this audience")

          def verify(self, token, current_time=None):
              """ https://github.com/awslabs/aws-support-tools/blob/master/Cognito/decode-verify-jwt/decode-verify-jwt.py """
              if not token:
                  raise TokenVerifyError("No token provided")

              headers = self._extract_headers(token)
              pkey_data = self._find_pkey(headers)
              self._verify_signature(token, pkey_data)

              claims = self._extract_claims(token)
              self._check_expiration(claims, current_time)
              self._check_audience(claims)

              self.claims = claims 
              return claims
     ```
  
  5. In `app.py`
     ```py
      from lib.cognito_jwt_token import CognitoJwtToken, extract_access_token, TokenVerifyError
      ..
      ..

      app = Flask(__name__)

      cognito_jwt_token = CognitoJwtToken(
        user_pool_id=os.getenv("AWS_COGNITO_USER_POOL_ID"), 
        user_pool_client_id=os.getenv("AWS_COGNITO_USER_POOL_CLIENT_ID"),
        region=os.getenv("AWS_DEFAULT_REGION")
      )

      ..
      ..
      ..

      @app.route("/api/activities/home", methods=['GET'])
      @xray_recorder.capture('activities_home')
      def data_home():
        data = HomeActivities.run()
        access_token = extract_access_token(request.headers)
        try:
          claims = cognito_jwt_token.verify(access_token)
          # authenicatied request
          app.logger.debug("authenicated")
          app.logger.debug(claims)
          app.logger.debug(claims['username'])
          data = HomeActivities.run(cognito_user_id=claims['username'])
        except TokenVerifyError as e:
          # unauthenicatied request
          app.logger.debug(e)
          app.logger.debug("unauthenicated")
          data = HomeActivities.run()
        return data, 200

     ```
  6. In `homeactivities.py`
      ```py
        ..
        ..
        def run(cognito_user_id=None):

        ..
        ..
        if cognito_user_id != None:
          extra_crud = {
            'uuid': '248959df-3079-4947-b847-9e0892d1bab4',
            'handle':  'Lore',
            'message': 'My dear brother, it the humans that are the problem',
            'created_at': (now - timedelta(hours=1)).isoformat(),
            'expires_at': (now + timedelta(hours=12)).isoformat(),
            'likes': 1042,
            'replies': []
          }
          results.insert(0,extra_crud)

        span.set_attribute("app.result_length", len(results))

      ```
  7. Add to `docker-compose.yml`
      ```
        AWS_COGNITO_USER_POOL_ID: "user pool id"
        AWS_COGNITO_USER_POOL_CLIENT_ID: "user pool client id"   
      ```
  8.  In `ProfileInfo.js`
      ```js
          try {
              await Auth.signOut({ global: true });
              window.location.href = "/"
              localStorage.removeItem("access_token")
          } catch (error) {
              console.log('error signing out: ', error);
          }

      ```

## Homework Challenges

1. **Extract Email from url on Confirmation page**

   https://mytechsolutions.hashnode.dev/how-to-extract-parameter-value-from-query-string-in-reactjs





## Security Module - Ashish
[video](https://www.youtube.com/watch?v=tEJIeII66pY)
  - Types
    - Traditional - username/password, physical
    - SAML/SingleSignOn- Apple FaceID
    - OpenId Connect - social credentials(Google,Facebook,Amazon) (Authentication)
    - OAuth - tag teams with OpenID (Authorization)
  - Cognito User Pool vs Cognito Identity Pool
  - AWS WAF - rate limiting,Allow/deny lisy, deny acces from region
  - AWS Cloud Trail - enable to monitor malicious Cognito behavior
  - AWS Cognito should only be in the AWS Region that you are lgecally allowed to be holding user data in 

