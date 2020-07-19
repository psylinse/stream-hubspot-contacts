# Generating Lead Records in HubSpot from your Landing Page Sales Chat Widget Using Stream Chat

In this tutorial, learn how to build a [Stream](https://getstream.io) chat widget that connects to the HubSpot CRM to automatically create a new contact when a customer initiates a chat. This widget, backed by the [Stream chat API](https://getstream.io/chat/docs/?language=js), can be easily embedded to your site as a chat widget for sales, support, or a landing page. You can take this knowledge to build powerful sales tools that seamlessly integrate with the HubSpot API.

The application utilizes a [React](https://reactjs.org/) `frontend` and an [Express](https://expressjs.com/) `backend`. The tutorial explains how to use some basic features of the powerful [Stream Library](https://getstream.io/), which handles most of creating a chat widget.

## Overview

The application utilizes a React `frontend` and an Express `backend`. The tutorial explains how to use some basic features of the powerful [Stream Library](https://getstream.io/), which handles most of the chat widget UX.

The code required for this tutorial is available in [GitHub](https://github.com/isaidspaghetti/stream-hubspot-contacts). If you'd like to build the app from scratch, use `npm express generator --no-view` for the backend, and `create-react-app` for the frontend. Be sure to use the `package.json` file from this repository to get the required dependencies loaded in your version. Otherwise, you can clone the repo from GitHub and follow along.

## Prerequisites

This tutorial is written to work with a wide range of skillsets. It requires basic knowledge of [React Hooks](https://reactjs.org/docs/hooks-intro.html), [Express](https://expressjs.com/), and [Node.js](https://nodejs.org/en/). The code is built and run with the [Node Package Manager](https://www.npmjs.com/get-npm) and is made to run locally. We also use [dotenv](https://www.npmjs.com/package/dotenv).

You'll need to set up a free [Stream Account](https://getstream.io/get_started/?signup=#flat_feed) and a free [HubSpot Account](https://app.hubspot.com/signup/crm/step/user-info?hubs_signup-cta=getstarted-crm&hubs_signup-url=www.hubspot.com%2Fproducts%2Fget-started).

### Not Covered

* We'll create a Stream Client and register a user with a chat channel, but we won't specifically describe how to set up support/sales user experience. We'll focus primarily on the customer's experience.
* We won't explore notifying a customer representative when a chat is initiated.
* Styling and CSS: this app uses the out-of-the-box styling of Stream. Check out Stream's [free UI Kit](https://getstream.io/chat/ui-kit/) to make your chat app shine ✨.
* Encryption or Authentication. To add some more security to your app, check out [this post](https://getstream.io/blog/hipaa-chat/), which shows how to authenticate users and encrypt messages.

## What We'll Do

* Set up a free HubSpot account and activate a key.
* Set up a free Stream account and activate a key.
* Create a React form to capture the customer's first name, last name, and email.
* Use an Express backend to:
    1. Send user form data to your HubSpot Dashboard
      * Bonus: how to create custom HubSpot Contact Fields!
    2. Create a one-on-one, private Stream Chat [Channel](https://github.com/isaidspaghetti/stream-hubspot-contacts)
    3. Respond to the frontend with required [credentials](https://github.com/isaidspaghetti/stream-hubspot-contacts) to join
* Join and load the specified Chat in the frontend using Stream's built-in UI Components.

## Let's Get Down To Business

First, we need to set up your unique API keys from HubSpot and Stream. These authenticate your app and are to be stored in a secure `.env` file. The Git Repo includes a `.env.example` file you can use as a template. Add your unique keys to this file, then remove '.example' from the file name.

<!-- https://gist.github.com/isaidspaghetti/cf14fabb996a5d16cde7608ec34c3b15 -->
```text
// backend/.env.example
NODE_ENV=development
PORT=8080

STREAM_API_KEY=your stream API key goes here
STREAM_API_SECRET=your stream API secret goes here
HUBSPOT_API_KEY=your HubSpot API key goes here
```

### Set-up your HubSpot

1. Create your account at [HubSpot](https://app.hubspot.com/signup/crm/step/user-info?hubs_signup-cta=getstarted-crm&hubs_signup-url=www.hubspot.com%2Fproducts%2Fget-started) and complete the registration form.

2. Once you are logged into the `HubSpot Dashboard,` go to Settings in the upper-right corner

![](images/hubspot-navbar.png)

3. Navigate to Integrations > API Key, and create a key. If you're a robot, stop here. You've gone too far...

![](images/hubspot-settings.png)

4. Copy the HubSpot API key and paste it in the `.env` file located in the `backend` folder. HubSpot's API is authenticated via this key.

### Set-up your Stream Account

1. Sign up for a [Stream Trial](https://getstream.io/get_started/?signup=#flat_feed).

1. To generate a Stream API Key and API Secret, navigate to your [Stream.io Dashboard](https://getstream.io/dashboard/). 

![](images/stream-dashboard-button.png)

2. Then click on "Create App", and complete the form like in the following screenshot.

![](images/stream-create-app-button.png)

3. Give your app a name, select "Development" and click "Submit". 
 
![](images/stream-create-new-app-button.png)

4. Stream will generate a Key and Secret for your app. You need to copy these into your `backend` `.env` file as well.
 
![](images/stream-key-secret-copy.png)

### Spin up the app

1. If you haven't already, run `npm install` on both the `frontend` and `backend` folders.

2. Once your packages are installed, run either `npm start` or `nodemon` on both the `frontend` and `backend` folders. 

## Registration Form

When opening this app in the browser, the user will see this login form:

![](images/stream-chat-login.png)

The following snippet shows how the registration form is created. We'll ignore the chat app code for now, as indicated with `// ...`. 

<!-- https://gist.github.com/isaidspaghetti/800dec9674e42bb829ebaac1494d0076 -->
```jsx
//frontend/src/App.js:7
function App() {
  const [email, setEmail] = useState('');
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
//...
    return (
      <div className="App container">
        <form className="card" onSubmit={register}>
          <label>First Name</label>
          <input
            type="text"
            value={firstName}
            onChange={(e) => setFirstName(e.target.value)}
            placeholder="first name"
          />
          <label>Last Name</label>
          <input
            type="text"
            value={lastName}
            onChange={(e) => setLastName(e.target.value)}
            placeholder="last name"
          />
          <label>Email</label>
          <input
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            placeholder="email"
          />
          <button className="btn btn-block" type="submit">
            Start chat
          </button>
        </form>
      </div>
    );
  }
}

export default App;
```

The simple form above sets up three `useStates` to update and store the user input fields. The form's `onSubmit` function, `register()`, will post the user credentials to the backend.

## Registering User with Backend

Let's take a look at the first half of the frontend's `register()` function. The second half of this function handles the response from the backend, which we will cover next. We use an asynchronous await function to give the backend time to do its work before we continue rendering in the frontend, and wrap the work in a try block for error handling. 

<!-- https://gist.github.com/isaidspaghetti/330e5a38b7e6bf518b19e8a57e914b47s -->
 ```jsx
 //frontend/src/App.js:15
 const register = async (e) => {
    try {
      e.preventDefault();
      var response = await fetch('http://localhost:8080/registrations', {
        method: 'POST',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          firstName,
          lastName,
          email,
        }),
      });
    // ...
    } catch (err) {
        console.error(err)
    }
```

## Configure the Backend


> **A brief explanation of backend routing**
> Let's take a peek at how we handle the backend routing. `npm express generator` sets up this routing for us, but let's run through it in case it's new to you. To start our server we run `npm start`. The command `npm start` is a script included in `backend/package.json`. This script tells node to run the file `backend/bin/www`, which in turn tells Express to start a server on port 8080 (the port number can be changed in the `.env` file).  Any communications from the frontend are directed to this port. The `www` file also configures the `api.js` file to handle those requests. Next, `api.js` tells Express to `.use` the file `backend/routes/index.js` which is mounted under the root path (`'/'`). In other words: our request from the front end will get routed to `index.js`. (To learn more about how this works, take a peek at [this tutorial](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/routes)).


Before we dive into handling our routes, let's configure `index.js`:

<!-- https://gist.github.com/isaidspaghetti/2e40841cf8dfdeb1f581912e88f374a8 -->
```javascript
//backend/routes/index.js:1
const express = require('express');
const router = express.Router();
const StreamChat = require('stream-chat');
const Hubspot = require('hubspot');
require('dotenv').config();

const apiKey = process.env.STREAM_API_KEY;
const apiSecret = process.env.STREAM_API_SECRET;
```

The [Stream Chat](https://github.com/GetStream/stream-chat-js) library is Stream's Chat App library that does all the heavy lifting of creating the chat app itself. HubSpot offers an excellent [library](https://www.npmjs.com/package/hubspot) we will utilize as well.
By requiring and configuring `dotenv`, we're able to access the private variables we set up in `.env`. Call these variables using `process.env`. The `hubspot` library will make connecting to their API a breeze.

## Backend Registration Endpoint Process Flow

When a user registers to start a chat, the handler function, configured via `router.post('/registrations')`, takes over. This handler is our primary backend function, and will call a few handy methods to set up our chat session. Let's review the router function, then step through it to understand it.

* Call `createHubspotContact()` to create a HubSpot contact
* Call `createUsers()` to create our `customer` and `supporter` chat members
* Register our app as a Stream `client`
* Register (or update) users with our Stream client using `upsertUsers()`
* Create a private chat `channel` in our `client`
* Create a `customerToken` for the frontend to join said channel
* Respond to the frontend with all required data to start the client in a browser

<!-- https://gist.github.com/isaidspaghetti/1eebf1a62534d5970a2c6b5ffaeb2c82 -->
```javascript
//backend/routes/index.js:46
router.post('/registrations', async (req, res, next) => {
  try {
    await createHubspotContact(firstName, lastName)

    const client = new StreamChat.StreamChat(apiKey, apiSecret);

    [customer, supporter] = createUsers(firstName, lastName)

    await client.upsertUsers([
      customer,
      supporter
    ]);

    const channel = client.channel('messaging', customer.id, {
      members: [customer.id, supporter.id],
    });

    const customerToken = client.createToken(customer.id);

    res.status(200).json({
      customerId: customer.id,
      customerToken,
      channelId: channel.id,
      apiKey,
    });

  } catch (err) {
    console.error(err);
    res.status(500).json({ error: err.message });
  }
});
```

## Creating Custom Contact Properties in HubSpot

This application will update a customized contact property in the HubSpot CRM.
To use a custom property follow these steps in your HubSpot Dashboard: 

1. Navigate to your contacts:

![](images/hubspot-navbar-contacts.png)

2. Click the 'Actions' drop down menu, then 'Edit Properties':

![](images/hubspot-contacts.png)

3. Click the 'Create Property' button and add whatever type of custom fields you'd like to use.

![](images/hubspot-properties.png)

## Connecting to the HubSpot API

The backend router first creates the HubSpot contact with the `createHubspotContact()` method: 

<!-- https://gist.github.com/isaidspaghetti/d074eaad8678d4c86c928983948ba38e -->
```javascript
//backend/routes/index.js:10
async function createHubspotContact(firstName, lastName) {
  const hubspot = new Hubspot({
    apiKey: process.env.HUBSPOT_API_KEY,
    checkLimit: false
  })

  const contactObj = {
    properties: [
      { property: 'firstname', value: firstName },
      { property: 'lastname', value: lastName },
      { property: 'email', value: email },
      {
        property: 'your_custom_property',
        value: 'anything you want, even a multi-line \n string'
      }
    ]
  }
  const hubspotContact = hubspot.contacts.create(contactObj)
  ```

 The `contactObj` is the argument to HubSpot's `.create()` method. Any HubSpot contact property can be used in `contactObj`. Check out their full list of properties [here](https://legacydocs.hubspot.com/docs/methods/contacts/contact-properties-overview). Note how we used `your_custom_property` as a key. The code will throw an error if you don't have a matching property in your HubSpot CRM. 

## Customer Stream Registration

To keep a chat secure, we can specify which users can use our client. Let's create a `customer` object for our frontend user, and a `supporter` object to represent a sales rep or support rep on the other end of the chat.

<!-- https://gist.github.com/isaidspaghetti/ec7f995b81421fb23e178c4afa52a056 -->
```javascript
//backend/routes/index.js:30
function createUsers(firstName, lastName) {
  const customer = {
    id: `${firstName}-${lastName}`.toLowerCase(),
    name: firstName,
    role: 'user',
  };

  const supporter = {
    id: 'adminId',
    name: 'unique-admin-name',
    role: 'admin'
  }
  return [customer, supporter]
}
```

Note the keys included for the users above. Stream supports a myriad of [properties](https://getstream.io/chat/docs/init_and_users/?language=js) you can add to your users, but for this example, we'll simply add an `id`, `name`, and `role`.  

Back in our primary backend function, the `upsertUsers()` method registers both our `customer` and our `supporter` admin so they can use our app. 

## Create a Stream channel

Back to the `router.post` function. Now that we have our client configured with the proper credentials, and our users registered with that client, we can open a channel for the two to chat. Stream's `channel()` method first accepts a [channel type](https://getstream.io/chat/docs/channel_features/?language=js); `'messaging'` is the best type for this app.

Each channel on your client should have a unique name. For simplicity, we use the customer's email address, so that if the user is disconnected from their chat, they can return to it by entering the same credentials into the registration form. In your production application you should create secure id's that can't be guessed.

The `members` argument specifies which users can join this channel. This is not required for the channel, but by specifying the members, we add a layer of security by preventing other users from joining the channel. If `members` is not included with the arguments, the channel will be public by default.

Stream provides a quick and easy token generator: `createToken()`. This will be used in the frontend to verify the user. The final response to the frontend includes all information required to load the client in the browser and join the channel specified.

## Customer Joins Chat in Frontend

Once the backend is finished, the frontend needs to:

* Decompose the response
* Join the Stream client using the `apiKey`
* Establish the browser's user using the `customerToken`
* Join the specific channel using the `channelId`
* Render the Stream Chat

The snippet below uses `//...` to indicate code we already covered in the first section.

<!-- https://gist.github.com/isaidspaghetti/f88359e13d0a226a4f69191125cff2e5 -->
```jsx
//frontend/App.js:7
function App() {
  //...
  const [chatClient, setChatClient] = useState(null);
  const [channel, setChannel] = useState(null);

  const register = async (e) => {
    try {
      e.preventDefault();
      var response = await fetch('http://localhost:8080/registrations', {
        method: 'POST',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          firstName,
          lastName,
          email,
        }),
      });
      
      const { customerId, customerToken, channelId, apiKey } = await response.json();
      const chatClient = new StreamChat(apiKey);
      await chatClient.setUser(
        {
          id: customerId,
          name: firstName,
        },
        customerToken,
      )
      const channel = chatClient.channel('messaging', channelId);
      setChatClient(chatClient);
      setChannel(channel)

    } catch (e) {
      console.error(e)
    }
  };

  if (chatClient && channel) {
    return (
      <div className="App">
        <Chat client={chatClient} theme={'messaging light'}>
          <Channel channel={channel}>
            <Window>
              <ChannelHeader />
              <MessageList />
              <MessageInput />
            </Window>
            <Thread />
          </Channel>
        </Chat>
      </div>
    );
  } else {
    return (
      //...
    );
  }
}
```

The responses from the backend are used in the same manner for the frontend: 
* Establish the client using the Stream `apiKey`
* Set the frontend user, supplying the `customerToken`
* Join the channel we already created

The `chatClient` and `channel` states determine what to render on the page. After a successful response from the backend, these objects are present, and the Stream Chat components will be rendered. 

To create a fully functional chat component from scratch would be a monumental task. Using the Stream's components get's you going quickly. Keep in mind that the Stream Chat Components used here are the most basic, but Stream offers deeper customization.

## That's a Wrap

So, there you have it: a customizable shat widget that easily pushes user inputs to your HubSpot CRM. Stay tuned for more posts on how to connect HubSpot with agile Stream apps!