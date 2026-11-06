# Social Network API

An API for a social network web application where users can share thoughts, react to friends' thoughts, and manage a friend list. It is built with Express.js for routing, a MongoDB database, and the Mongoose ODM (Object Data Modeling).

This is the NoSQL Challenge from a coding boot camp. The app is not deployed, so a walkthrough video demonstrates it.

## Walkthrough Video

[Watch the walkthrough video](https://drive.google.com/file/d/1JFfYQ777PrCddtdIs-wCtNOW2htyFC8F/view?usp=sharing)

## Table of Contents

- [Features](#features)
- [Built With](#built-with)
- [Installation](#installation)
- [Running the API](#running-the-api)
- [Usage](#usage)
- [API Reference](#api-reference)
- [Data Models](#data-models)
- [Project Structure](#project-structure)
- [Credits](#credits)
- [License](#license)

## Features

- **Users:** create, view, update and delete users. Deleting a user also deletes their thoughts.
- **Friend lists:** add and remove friends on a user.
- **Thoughts:** create a thought linked to a user, view all thoughts (newest first) or one thought, update it and delete it.
- **Reactions:** add and remove reactions on a thought. Reactions are stored inside the thought as embedded documents.
- **Virtual fields:** `friendCount` on users and `reactionCount` on thoughts are calculated automatically.
- **Validation:** usernames and emails must be unique, emails must be in a valid format, and thoughts and reactions are limited to 280 characters.
- **Formatted timestamps:** `createdAt` dates are formatted with Moment.js, for example `Feb 10, 2023 at 03:15 pm`.

## Built With

- [Node.js](https://nodejs.org/) and [Express](https://expressjs.com/) for the server and routing
- [MongoDB](https://www.mongodb.com/) for the database
- [Mongoose](https://mongoosejs.com/) (v6) for modeling and querying
- [Moment.js](https://momentjs.com/) for date formatting
- [nodemon](https://nodemon.io/) for automatic restarts during development
- [Insomnia](https://insomnia.rest/) for testing the routes

## Installation

You need:

- [Node.js](https://nodejs.org/) (the LTS version is fine), which includes npm
- [MongoDB Community Server](https://www.mongodb.com/docs/manual/installation/) running on your computer
- An API client such as [Insomnia](https://insomnia.rest/download) to send requests

**1. Get the code and install the packages:**

```bash
git clone https://github.com/jveyhl/social_network_api.git
cd social_network_api
npm install
```

**2. Install and start MongoDB.** On a Mac with [Homebrew](https://brew.sh/):

```bash
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

For Windows and Linux, follow the [MongoDB installation guide](https://www.mongodb.com/docs/manual/installation/). The database and its collections are created automatically the first time you add data, so there is nothing to set up or seed.

## Running the API

```bash
npm start
```

For development, `npm run watch` uses nodemon to restart the server whenever you save a file.

When you see `API server running on port 3001!`, the API is ready at **http://localhost:3001/api**.

### Configuration

The defaults work for a local setup. To change them, set these environment variables in your terminal before starting the server:

| Variable | Default | Purpose |
|---|---|---|
| `PORT` | `3001` | Port the server listens on |
| `MONGODB_URI` | `mongodb://localhost/socialmedia` | MongoDB connection string |

For example: `MONGODB_URI="mongodb://127.0.0.1/socialmedia" PORT=4000 npm start`

This project does not read a `.env` file, so the variables must be set in the terminal.

### Troubleshooting

- **The server never prints "API server running" and shows a connection error:** MongoDB is not running. Start it (for Homebrew, `brew services start mongodb-community`) and try again.
- **The error mentions `::1`:** use `127.0.0.1` instead of `localhost` by setting `MONGODB_URI="mongodb://127.0.0.1/socialmedia"`.

## Usage

Use Insomnia (or any API client) to send requests to `http://localhost:3001/api`. Send data as JSON, with the `Content-Type` set to JSON. This order creates a small network from scratch:

1. **Create two users** with `POST /api/users`:

   ```json
   {
     "username": "sam_taylor",
     "email": "sam_t@example.com"
   }
   ```

   Each response includes the user's `_id`. Copy both ids. Use lowercase letters in emails.

2. **Make them friends** with `POST /api/users/<sam's id>/friends/<other user's id>`. No request body is needed.

3. **Create a thought** with `POST /api/thoughts`. The `userId` is the `_id` of the user posting it:

   ```json
   {
     "thoughtText": "Here is my first thought!",
     "username": "sam_taylor",
     "userId": "<sam's id>"
   }
   ```

   The response is only a confirmation message. Use `GET /api/thoughts` to see the new thought and its `_id`.

4. **Add a reaction** with `POST /api/thoughts/<thought id>/reactions`:

   ```json
   {
     "reactionBody": "Great thought!",
     "username": "jen_walker"
   }
   ```

5. **View the results** with `GET /api/users/<sam's id>`, which returns the user with their friends and thoughts filled in, plus `friendCount`. `GET /api/thoughts/<thought id>` shows the thought with its reactions and `reactionCount`.

## API Reference

### Users

| Method | Route | Description | Request body |
|---|---|---|---|
| GET | `/api/users` | Get all users | |
| POST | `/api/users` | Create a user | `username`, `email` |
| GET | `/api/users/:userId` | Get one user, with friends and thoughts | |
| PUT | `/api/users/:userId` | Update a user | Fields to change |
| DELETE | `/api/users/:userId` | Delete a user and their thoughts | |
| POST | `/api/users/:userId/friends/:friendId` | Add a friend | |
| DELETE | `/api/users/:userId/friends/:friendId` | Remove a friend | |

### Thoughts and Reactions

| Method | Route | Description | Request body |
|---|---|---|---|
| GET | `/api/thoughts` | Get all thoughts, newest first | |
| POST | `/api/thoughts` | Create a thought and link it to a user | `thoughtText`, `username`, `userId` |
| GET | `/api/thoughts/:thoughtId` | Get one thought | |
| PUT | `/api/thoughts/:thoughtId` | Update a thought | Fields to change |
| DELETE | `/api/thoughts/:thoughtId` | Delete a thought | |
| POST | `/api/thoughts/:thoughtId/reactions` | Add a reaction | `reactionBody`, `username` |
| DELETE | `/api/thoughts/:thoughtId/reactions/:reactionId` | Remove a reaction | |

Any other route returns `This route does not exist!`.

## Data Models

**User**

| Field | Type | Rules |
|---|---|---|
| `username` | String | Required, unique, trimmed |
| `email` | String | Required, unique, must match an email format |
| `thoughts` | Array of Thought ids | References the user's thoughts |
| `friends` | Array of User ids | References other users |
| `friendCount` | Virtual | Number of friends |

**Thought**

| Field | Type | Rules |
|---|---|---|
| `thoughtText` | String | Required, 1 to 280 characters |
| `username` | String | Required |
| `createdAt` | Date | Set automatically and formatted with Moment.js |
| `reactions` | Array of Reactions | Embedded documents |
| `reactionCount` | Virtual | Number of reactions |

**Reaction** (embedded in a thought)

| Field | Type | Rules |
|---|---|---|
| `reactionId` | ObjectId | Generated automatically |
| `reactionBody` | String | Required, up to 280 characters |
| `username` | String | Required |
| `createdAt` | Date | Set automatically and formatted with Moment.js |

## Project Structure

```
social_network_api/
├── config/
│   └── connection.js        Connects Mongoose to MongoDB
├── controllers/
│   ├── user-controller.js   User and friend logic
│   └── thought-controller.js Thought and reaction logic
├── models/
│   ├── User.js
│   ├── Thought.js
│   ├── Reaction.js          Reaction schema embedded in Thought
│   └── index.js
├── routes/
│   ├── api/
│   │   ├── user-routes.js
│   │   ├── thought-routes.js
│   │   └── index.js
│   └── index.js
└── server.js                Starts Express once MongoDB is connected
```

## Credits

Built as a coding boot camp challenge. [Moment.js](https://momentjs.com/) is used for date formatting.

## License

Released under the MIT License. See the [LICENSE](LICENSE) file for details.
