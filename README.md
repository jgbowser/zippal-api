# Zip Pal (V 1.0)
### A pen pal app (api)
#### [Hosted Live on Vercel](https://zippal.vercel.app/)

In the times of Covid 19 and quarantine, we’ve all experienced isolation. Rush University’s research has shown that _chronic social isolation_ increases the risk of mental health issues like depression, anxiety and substance abuse, as well as raises the risk of dementia in older adults. It can also lead to chronic conditions like high blood pressure, heart disease and diabetes.

_Zip Pal_ was created with the idea of promoting conversation to help alleviate social isolation - and hopefully promote productive discourse in our increasingly polarized world. _Zip Pal_ , using our _find-a-pal_ algorithm, provides 2 random Users (Pals) the opportunity to connect and talk about shared interests or anything they'd like. 

When signing up, Users can add their location and a brief bio, this information is useful when _Pals_ are paired with them in a new conversation. When using the _find-a-pal_ feature, Users can start a new conversation with that _Pal_ or find another _Pal_. Currently, Users can have up to 5 conversations going at one time and they can leave a conversation at any time. 

---

#### Tech Stack & Dependencies
 ##### Node.js 
  * [Express](http://expressjs.com/)
   * [Cors](https://github.com/expressjs/cors#readme)
  * [Helmet](https://helmetjs.github.io/)
 ##### PostgreSQL 
  * [pg](https://github.com/brianc/node-postgres) 
  * [Knex](https://knexjs.org/)
  * [Postgrator](https://github.com/rickbergfalk/postgrator#readme)
 ##### _Testing_
  * [Chai](http://chaijs.com/)
  * [Mocha](https://mochajs.org/)
  * [Supertest](https://github.com/visionmedia/supertest#readme)
 ##### _Logging_
  * [Morgan](https://github.com/expressjs/morgan#readme)
  * [Winston](https://github.com/winstonjs/winston#readme)
 ##### _Authorization & Authentication_
  * [Bcrypt.js](https://github.com/dcodeIO/bcrypt.js#readme)
  * [JWT](https://github.com/auth0/node-jsonwebtoken#readme)
  * [xss](https://github.com/leizongmin/js-xss) 

---
## API Documentation
---

We use this codebase to access our datatables with user, conversation, and message data on the database _server hosted by heroku_.

We also use this codebase to confirm user credentials with the help of _JWT_ for authentication and _bcrypt_ to hash user passwords. 

With the exception of _registration and login_ all endpoints require _JWT_ .

### API Overview

```text
/api
.
├── /auth/token
│   └── POST
│       └── /
│   └── PUT
│       └── /
├── /users
│   └── GET
│       ├── /
│       └── /profile
│   └── POST
│       └── /
│   └── PATCH
│       └── /
├── /conversation
│   └── GET
│       ├── /
│       └── /find/:currentConversationIds
│   └── POST
│       └── /
│   └── PATCH
│       └── /:conversation_id/deactivate
├── /message
│   └── POST
│       └── /
│   └── PATCH
│       ├── /:message_id/save
│       ├── /:message_id/send
│       └── /:message_id/read
```
--- 

### [/api/auth] Auth Endpoints 

#### [/token] POST 
  * Generating intial token upon successful login

```js
// req.body
{
  username: String,
  password: String
}

// res.body
{
  authToken: String
}
```
#### [/token] PUT 
  * Refreshing token
```js
// req.header
Authorization: Bearer ${token}

// res.body
{
  authToken: ${token}
}
```
---

### [/api/user] User Endpoints 

#### [/] GET 
  * Request User's data

```js
// req.header
Authorization: Bearer ${token}

// res.body
{
  username: String,
  display_name: String,
  active_conversations: String,
  bio: String,
  active_conversations: Number,
  location: String,
  fa_icon: String
}
```

#### [/profile] GET
  * Request User's profile data, serialized with xss (for react-context)

```js
// req.header
Authorization: Bearer ${token}

// res.body
{
  id: String,
  display_name: xss(String),
  username: xss(String),
  location: xss(String),
  bio: xss(String),
  active_conversations: Number,
  fa_icon: String
}
```

#### [/] POST
  * Creates a new User account

```js
// req.body (no auth needed, password will be hashed prior to storing)
{
    password: String 
    // Password must be 8-72 characters long
    // and contain at least one upper case, lower case, special character, and number
    username: String
    display_name: String
}

// res.body (returns serialized user, does *not* return password)
{
  id: String,
  display_name: xss(String),
  username: xss(String),
  location: xss(String),
  bio: xss(String),
  active_conversations: Number,
  fa_icon: String
}
```

#### [/] PATCH
  * Updates User's information displayed on their profiles

```js

// req.header
Authorization: Bearer ${token}

// req.body (must contain at least one of the fields)
{ 
    bio: String
    location: String 
    fa_icon:String
}

// res.body (returns serialized user, does *not* return password)
{
  id: String,
  display_name: xss(String),
  username: xss(String),
  location: xss(String),
  bio: xss(String),
  active_conversations: Number,
  fa_icon: String
}
```

---
### [/api/conversation] Conversation Endpoints

#### [/] GET 
  * Requests all existing conversations and messages

```js

// req.header
Authorization: Bearer ${token}

// req.body - req.user.id

// res.body - 
// object with conversations messages objects {c:[{},{}],m:[[{},{}],[{},{}]]}
// conversations object contains an array of conversation objects
// messages oject contains and array of arrays of conversation message objects (by conversation id)
{
  conversations:
    [  
        {
            date_created: "2020-10-27T19:59:01.405Z"
            fa_icon: "user-circle"
            id: 7
            is_active: true
            pal_name: "Phillip"
            user_1: 1
            user_1_turn: false
            user_2: 9
            user_2_turn: true
        }, 
        {...},
    ...],       
  messages: 
    [
        [
            {
                content: "Message iwofhwpvw"
                conversation_id: 7
                date_sent: "2020-10-27T19:59:04.628Z"
                id: 10
                is_read: false
                receiver_id: 9
                receiver_status: "Received"
                sender_id: 1
                sender_status: "Sent"
            },
            {...},
        ],
        [{...}],
    ]
} 
```

#### [/find/:currentConversationIds] GET
  * Find-a-pal - params used to filter for any userIDs, to avoid any repeats

```js
// example /find/12%2014%20 (users 1,2 and 1,4 are already in a conversation)

// req.header
Authorization: Bearer ${token}

// res.body - returns RandomUser object

{
    id: Number,
    display_name: String,
    username: String,
    bio: String,
    location: String,
    fa_icon: String
}
```
#### [/] POST
  * Create a new conversation (with new pal!)

```js
// req.header
Authorization: Bearer ${token}

// req.body - UserId of new pal, User_1: req.user.id
{
    user_2: Number
}

// res.body - object with conversation data, newPal_icon, newPal_displayName

{ 
    id: Number,
    user_1: Number,
    user_2: Number,
    date_created: Date,
    is_active: Boolean,
    user_1_turn: Boolean,
    user_2_turn: Boolean,
// ^ above is returned upon new entry to conversation table, added to response object
// below is added to response object with seperate query for user_2's data
    display_name: String,
    fa_icon: String
}
```
#### [/:conversation_id/deactivate] PATCH
```js
// req.header
Authorization: Bearer ${token}

// uses conversation's id from req.params
// returns 204, no res.body
```
---

### [/api/message] Message Endpoints

#### [/] POST

#### [/:message_id/save] PATCH

#### [/::message_id/send] PATCH

#### [/::message_id/read] PATCH

---

## Zip Pal is brought to you by 

* [John Bowser](https://github.com/jgbowser)
* [Phillip 'Lip' Cowan](https://github.com/lipcowan)
* [Mathew Murray](https://github.com/MathewMurray)
* [Ryan Whitmore](https://github.com/warptrail)

--- 