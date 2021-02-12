# Get Started - React.js

The tutorial lets you implement LoginRadius user registration, login and view profile on your React.js based application. 

> [Create an account](https://accounts.loginradius.com/auth.aspx?return_url=https://dashboard.loginradius.com/login&action=register) to get started if you don't have one yet!

When you signed up for the LoginRadius account, it created an app for you. This app is linked to a ready to use web page - [Auth Page (IDX)](https://www.loginradius.com/docs/developer/concepts/idx-overview/).

Auth Page (IDX) reflects the configuration changes that you make in [LoginRadius Dashboard](https://dashboard.loginradius.com/getting-started). You can utilize this webpage for authentication requirements on your React.js application.


## Choose Theme

In your LoginRadius Dashboard, from the left navigatation, click the **Auth Page (IDX)** and click the **Theme Customization** to choose the theme of your choice or customize it's look and feel:

![alt_text](images/image6.png "image_tooltip")


To preview your login page's theme, click **Go to your Login Page** link as highlighted on the above screen. Features like Email and Password login, User registration, Forgot password, and Remember me are already implemented on your Auth Page(IDX).


## Get Credentials {#getcredential}

Before using any of the APIs or Methods that LoginRadius provides, you need to get your **App Name**, **API Key**, and **API Secret**.

In your LoginRadius Dashboard, navigate to **[Configuration > API Credentials](https://dashboard.loginradius.com/configuration)** and click the **API Key And Secret** subsection to retrieve your API Credentials.



![alt_text](images/image7.png "image_tooltip")

## React JS Implementation

For this example, we will be using a sample app based on the Create React App boilerplate. For directions on how to use Create React App, you can reference [here.](https://reactjs.org/docs/create-a-new-react-app.html)

Once the CRA boilerplate is set up, follow these steps:

- Navigate to the project root and install `react-router-dom`: 

  `npm install react-router-dom`

- Go to `App.js` and modify the App component as follows:

```JavaScript
import './App.css';
import {
  BrowserRouter as Router,
  Switch,
  Route
} from "react-router-dom";
import Login from './Login'

function App() {
  return (
    <Router>
      <div className="App">
        <Switch>
          <Route exact path="/">
            <div>{"Application home"}</div>
          </Route>
        </Switch>
      </div>
    </Router>
  );
}

export default App;
```

## Registration or Login

In this tutorial, we are using Auth Page(IDX) for authentication. Thus, use the following registration and login URLs at the front end of your application. For example, you can add these links to **Sign Up** and **Sign In** buttons of your application. 

> Registration and Login functionality is already implemented on your Auth Page (IDX). Thus, you don’t need to implement them separately.

**Registration Page URL:**

`https://<LoginRadius APP Name>.hub.loginradius.com/auth.aspx?action=register&return_url=<Return URL>`

**Login Page URL:**

`https://<LoginRadius APP Name>.hub.loginradius.com/auth.aspx?action=login&return_url=<Return URL>`

**Where:**
- LoginRadius App Name is the name of your app as mentioned in [Get Credential](#getcredential) step.
- return_url is where you want to redirect users upon successful registration or login.

> return_url can be your website, frontend app, or backend server url where you are handling the access token. 



##  Obtain Access Token

On successful authentication on the Auth Page (IDX), the default script of LoginRadius sends an access token in the query string as a token parameter with the return_url.

The following is an example of the access token in the query string with the Return URL:

`<Return URL>?token=745******-3e8e-****-b3**2-9c0******1e.`

Pointing return_url to a route in your React application, you can capture the access token and proceed to retrieve the customer profile data with it in the next step, as well as handle other user functionality.

> Similar to Registration and Login actions, the Auth Page (IDX) supports more actions. Refer to [this document](https://www.loginradius.com/docs/developer/concepts/idx-overview/) for more information.



## Retrieve User Data using Access Token

Once the authentication is done using Auth Page, the user will be redirected to the supplied `return_url`. From your React application, we will implement a route to capture the access token.

For example: To get the user profile, add the `"/login"` route to the `App` component:

```JavaScript
import {
  BrowserRouter as Router,
  Switch,
  Route
} from "react-router-dom";
import Login from './Login'

function App() {
  return (
    <Router>
      <div className="App">
        <Switch>
          <Route exact path="/">
            <div>{"Application home"}</div>
          </Route>
          <Route path="/login">
            <Login />
          </Route>
        </Switch>
      </div>
    </Router>
  );
}
```

From the source root src folder, create a folder for the Login component and populate the index.js file:

```JavaScript
import React from "react"
import { withRouter } from "react-router-dom";

const apiKey = '{{ Your API Key }}';

class Login extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      userProfileResponse: null
    }
  }

  componentDidMount() {
    const token = new URLSearchParams(this.props.location.search).get("token");
    fetch("https://api.loginradius.com/identity/v2/auth/account?apikey=" + apiKey, {
      method: 'GET',
      headers: {
        'Authorization': 'Bearer ' + token
      },
    })
      .then(res => res.json())
      .then(res => {
        this.setState({ userProfileResponse: res })
      })
      .catch(e => {
        console.log(e);
      })
  }

  render() {
    const { userProfileResponse } = this.state;
    
    return (
      <div>
        <span style={{ whiteSpace: "pre-wrap", textAlign: "left" }}>
          {JSON.stringify(userProfileResponse, null, 4)}
        </span>
      </div>
    );
  }
}

export default withRouter(Login);
```

Once the `Login` component is implemented, set the `return_url` to point to the `/login` subdomain of your application. For example, in the local React instance, it can point to `http://localhost:3000/login`. This way, after logging in through the IDX Login page, your user will be redirected to the Login component that we just implemented.

##  Domain Whitelisting

For security reasons, LoginRadius processes the API calls that are received from the whitelisted domains. Local domains (http://localhost and http://127.0.0.1) are whitelisted by default. 

To whilelist your domain, in your LoginRadius Dashboard, navigate to **[Configuration > Domain Whitelisting](https://dashboard.loginradius.com/configuration)** and add your domain name:

![alt_text](images/image5.png "image_tooltip")


> Now, run the React application and you should get the user profile in response (json format) displayed in the "/login" route. Similarly, you can implement more features using the LoginRadius API. 

##  Explore Node.js Demo

As an alternative to handling all API calls in the React frontend, you may also opt to access the LoginRadius API from a Node backend. If you wish to do so, you can check out our Node.js demo to learn how to implement various LoginRadius features using SDK and its functions.

**[GitHub Demo Link](https://github.com/LoginRadius/login-page-demos/blob/master/node-idx-demo)**   |   **[Download Demo](https://github.com/LoginRadius/login-page-demos/archive/master.zip)**   


# Recommended Next Steps

How to manage email templates for verification and forgot password

How to personalize interfaces and branding of login pages

How to configure SMTP settings for sending emails to consumers

How to implement Social Login options like Facebook, Google

How to implement Phone Login

How to implement Passwordless Login


<!-- # Node.js SDK Reference

< Link to Node.js SDK doc > -->


# API Reference

< Link to APIs doc >