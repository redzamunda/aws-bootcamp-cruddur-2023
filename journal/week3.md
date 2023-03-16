# Week 3 — Decentralized Authentication

## Setup your Cognito User Pool

1.sign in to your aws account. Aways avoid loginin from root user for security reasons

2.find and open cognito from your search box

3.open the cognito and make sure to check the region .Try to maintain a particular region for this bootcamp

4.Create user pool

Fill in the following:

Cognito user pool sign-in options: Email click next

Multi-factor authentication: No MFA .you can set this we only picked this to reduce cost click next .leave the defaults

Required attributes (Additional required attributes): name, preferred_username

Email : choose email with Cognito

uncheck the use  aws cognito UI

User pool name: name of choice

App client name: name of choice

5. review your configurations and with cognito user pools you cant edit once created you would have to create another if you make a mistake so remember to carefully pick what you need.click create 

![user pool create](https://user-images.githubusercontent.com/127143210/225618345-c32e14d8-f9be-46aa-a887-2d0d160ed508.png)


## Install AWS Amplify
cd into the frontend folder and run the command below

```sh
npm i aws-amplify --save
```
this adds  `"aws-amplify"` to the  package.json folder bacause of the --save added at the end



## Configure Amplify

We need to hook up our cognito pool to our code in the `App.js`

```js
import { Amplify } from 'aws-amplify';
```

make sure to add this below the imports
```js
Amplify.configure({
  "AWS_PROJECT_REGION": process.env.REACT_APP_AWS_PROJECT_REGION,
  "aws_cognito_region": process.env.REACT_APP_AWS_COGNITO_REGION,
  "aws_user_pools_id": process.env.REACT_APP_AWS_USER_POOLS_ID,
  "aws_user_pools_web_client_id": process.env.REACT_APP_CLIENT_ID,
  "oauth": {},
  Auth: {
    // We are not using an Identity Pool
    // identityPoolId: process.env.REACT_APP_IDENTITY_POOL_ID, // REQUIRED - Amazon Cognito Identity Pool ID
    region: process.env.REACT_APP_AWS_PROJECT_REGION,           // REQUIRED - Amazon Cognito Region
    userPoolId: process.env.REACT_APP_AWS_USER_POOLS_ID,         // OPTIONAL - Amazon Cognito User Pool ID
    userPoolWebClientId: process.env.REACT_APP_CLIENT_ID,   // OPTIONAL - Amazon Cognito web Client ID (26-char alphanumeric string)
  }
});
```

define the following in your docker file under the frontend-react-js.also go to your user pool in aws click on it find the app integrations to copy your app client id

```
REACT_APP_AWS_PROJECT_REGION:"${AWS_DEFAULT_REGION}"
REACT_APP_AWS_COGNITO_REGION:"${AWS_DEFAULT_REGION"
REACT_APP_AWS_USER_POOLS_ID:" insert your user pool id"
REACT_APP_CLIENT_ID:"insert your app client id"
```

## Conditionally show components based on logged in or logged out

Inside our `HomeFeedPage.js`

```js
import { Auth } from 'aws-amplify';
```

add this below 

replace the checkAuth function with
```
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

We'll want to pass user to the following components:

```js
<DesktopNavigation user={user} active={'home'} setPopped={setPopped} />
<DesktopSidebar user={user} />
```

We'll rewrite `DesktopNavigation.js` so that it it conditionally shows links in the left hand column
on whether you are logged in or not.

Notice we are passing the user to ProfileInfo

```js
import './DesktopNavigation.css';
import {ReactComponent as Logo} from './svg/logo.svg';
import DesktopNavigationLink from '../components/DesktopNavigationLink';
import CrudButton from '../components/CrudButton';
import ProfileInfo from '../components/ProfileInfo';

export default function DesktopNavigation(props) {

  let button;
  let profile;
  let notificationsLink;
  let messagesLink;
  let profileLink;
  if (props.user) {
    button = <CrudButton setPopped={props.setPopped} />;
    profile = <ProfileInfo user={props.user} />;
    notificationsLink = <DesktopNavigationLink 
      url="/notifications" 
      name="Notifications" 
      handle="notifications" 
      active={props.active} />;
    messagesLink = <DesktopNavigationLink 
      url="/messages"
      name="Messages"
      handle="messages" 
      active={props.active} />
    profileLink = <DesktopNavigationLink 
      url="/@andrewbrown" 
      name="Profile"
      handle="profile"
      active={props.active} />
  }

  return (
    <nav>
      <Logo className='logo' />
      <DesktopNavigationLink url="/" 
        name="Home"
        handle="home"
        active={props.active} />
      {notificationsLink}
      {messagesLink}
      {profileLink}
      <DesktopNavigationLink url="/#" 
        name="More" 
        handle="more"
        active={props.active} />
      {button}
      {profile}
    </nav>
  );
}
```

We'll update `ProfileInfo.js`

```js
import { Auth } from 'aws-amplify';
```
```
const signOut = async () => {
  try {
      await Auth.signOut({ global: true });
      window.location.href = "/"
  } catch (error) {
      console.log('error signing out: ', error);
  }
}
```


##  Signin Page
Replace the import under the // [TODO] Authenication 

```js
import { Auth } from 'aws-amplify';
```
Replace the onSubmit function with

```
const [cognitoErrors, setErrors] = React.useState('');

const onsubmit = async (event) => {
  setErrors('')
  event.preventDefault();
  try {
    Auth.signIn(username, password)
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

let errors;
if (cognitoErrors){
  errors = <div className='errors'>{cognitoErrors}</div>;
}

// just before submit component
{errors}
```

## Signup Page
Replace the import under the // [TODO] Authenication 
```js
import { Auth } from 'aws-amplify';
```
Replace the signOut method 

```

const [cognitoErrors, setErrors] = React.useState('');

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

let errors;
if (cognitoErrors){
  errors = <div className='errors'>{cognitoErrors}</div>;
}

//before submit component
{errors}
```

## Confirmation Page

```js
const resend_code = async (event) => {
  setErrors('')
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
      setCognitoErrors("You need to provide an email in order to send Resend Activiation Code")   
    } else if (err.message == "Username/client id combination not found."){
      setCognitoErrors("Email is invalid or cannot be found.")   
    }
  }
}

const onsubmit = async (event) => {
  event.preventDefault();
  setCognitoErrors('')
  try {
    await Auth.confirmSignUp(email, code);
    window.location.href = "/"
  } catch (error) {
    setErrors(error.message)
  }
  return false
}
```

## Recovery Page

```js
import { Auth } from 'aws-amplify';
```

```

const onsubmit_send_code = async (event) => {
  event.preventDefault();
  setErrors('')
  Auth.forgotPassword(username)
  .then((data) => setFormState('confirm_code') )
  .catch((err) => setCognitoErrors(err.message) );
  return false
}

const onsubmit_confirm_code = async (event) => {
  event.preventDefault();
  setCognitoErrors('')
  if (password == passwordAgain){
    Auth.forgotPasswordSubmit(username, code, password)
    .then((data) => setFormState('success'))
    .catch((err) => setCognitoErrors(err.message) );
  } else {
    setErrors('Passwords do not match')
  }
  return false
}

## Authenticating Server Side

Add in the `HomeFeedPage.js` a header to pass along the access token

```js
  headers: {
    Authorization: `Bearer ${localStorage.getItem("access_token")}`
  }
```


In the `app.py`
change the cors to this

```py
cors = CORS(
  app, 
  resources={r"/api/*": {"origins": origins}},
  headers=['Content-Type', 'Authorization'], 
  expose_headers='Authorization',
  methods="OPTIONS,GET,HEAD,POST"
)
```

Testing the recovery and sign in and sign up pages

1.![confirmation code](https://user-images.githubusercontent.com/127143210/225679062-cef949f7-5d0a-4ac2-aca2-4a68beb4d471.png)

2.![password reset](https://user-images.githubusercontent.com/127143210/225680650-42a677b9-05eb-4787-83de-8de35c4ddbfc.png)

![setting error detection](https://user-images.githubusercontent.com/127143210/225701321-d5bc506a-f476-4d2f-93a8-a2f2bb49bcf7.png)

![s![mary-ejike sign in](https://user-images.githubusercontent.com/127143210/225701498-ff21f05a-1374-4fba-a155-eb716d2b3939.png)
ign in successful](https://user-images.githubusercontent.com/127143210/225701088-d0c1519a-fb1b-4c3c-909a-10770c94910c.png)

![user succesfully added](https://user-images.githubusercontent.com/127143210/225701395-1300c263-bf34-43f2-a9fd-b395e1d6682a.png)

https://user-images.githubusercontent.com/127143210/225699113-b881fc54-3bff-4404-899b-c53661d3608d.mp4

![ACTIVATION CODE SIDE](https://user-images.githubusercontent.com/127143210/225701320-49220f69-ff7a-4846-a7c9-d7d5e68e7fd1.png)

add the `Flask-AWSCognito` to the requirement.txt of the backend file and run a 

`pip installl-r requirements.txt`

create a user either from the command line or from the aws cognito page 
ensure to put your a real gmail address and confirm the user 

### set up user for user pool on the cli from your terminal 
$ aws cognito-idp admin-set-user-password \
 --user-pool-id <your-user-pool-id> \
 --username <username> \
 --password <password> \
 --permanent


