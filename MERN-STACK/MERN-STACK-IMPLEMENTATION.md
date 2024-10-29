

---

```markdown
# Setting Up a MERN Stack on Ubuntu: My Journey

Let me walk you through how I set up the backend environment for a MERN (MongoDB, Express.js, React, Node.js) stack on an Ubuntu server. I’ll explain each step in a way that’s easy to follow and give you insights into why each command matters.

## Step 1: Updating and Upgrading Ubuntu

The first thing I did was make sure that my system was fully updated. This is a crucial step because running the latest packages and security patches can prevent a lot of potential issues down the line.

I started with the following command:

```bash
sudo apt update
```

This command updated the list of available packages from the Ubuntu repository. It doesn’t actually install or change anything yet; it’s like refreshing the software catalog so the system knows what's available.

Next, I ran:

```bash
sudo apt upgrade
```

This command went ahead and upgraded all the installed packages to their latest versions. Doing this made me feel confident that I was working with the most up-to-date and secure software, which is always a good foundation.

## Step 2: Installing Node.js

With my system updated, it was time to install Node.js, which is essential for running JavaScript on the server. Since Node.js doesn’t always come in the latest version in Ubuntu’s default repository, I had to pull it from a special NodeSource repository.

### Adding the NodeSource Repository

To set this up, I used `curl` to fetch a setup script directly from NodeSource:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```

This command downloaded and executed a setup script that added the NodeSource repository to my system. Adding this repository ensured that I’d get the latest stable version of Node.js instead of an outdated version.

### Installing Node.js and npm

Once the repository was added, installing Node.js was straightforward. Here’s the command I used:

```bash
sudo apt-get install -y nodejs
```
![nodejs installed](https://github.com/user-attachments/assets/659c06bd-a6d2-46ed-aeb9-92a2fde47419)

This command installed both `node` and `npm` (Node Package Manager) on my server. The `-y` flag is a little trick that automatically confirms any prompts during installation, so it saved me from having to manually confirm.

At this point, I had both Node.js and `npm` installed. `npm` is especially handy because it allows me to install any additional packages and libraries I’ll need for my project, like Express.js.

### Verifying the Installation

After installation, I wanted to make sure everything was set up correctly. So, I checked the Node.js version with:

```bash
node -v
```
I also checked if the Node Packet Manager (npm) version that was instaled with the command 

```bash
npm -v
```
![nodejs version and installation verified ](https://github.com/user-attachments/assets/4399dbd7-b3bf-4b48-8ee1-5c9402c8db96)

Seeing a version numbers pop up confirmed that Node.js and npm were successfully installed! 

---

```markdown
## Step 3: Application Code Setup

With Node.js installed, I was ready to start setting up my project files. Here’s how I created the necessary directory and initialized my project for the MERN stack.

### Creating a Project Directory

First, I created a new directory for my project. Since I’m building a To-Do application, I named the folder `Todo`. Here’s the command I used:

```bash
mkdir Todo
```

This command created a folder called `Todo` where I’ll store all my project files. Having a dedicated directory keeps everything organized, especially as the project grows.

### Verifying the Directory Creation

To make sure the directory was created, I listed all the files and folders in my current directory using:

```bash
ls
```

This command showed me a list of files and directories, and I could see that `Todo` was indeed there. 

**TIP**: If you want to see more detailed information about files and directories, you can use `ls -lih`. This version of `ls` displays properties like file size and permissions in a human-readable format. You can explore other options with `ls --help`.

### Changing to the Project Directory

Next, I changed my working directory to the `Todo` folder:

```bash
cd Todo
```

This command moved me into the `Todo` directory, where I’ll be working for the rest of the setup.

### Initializing the Project with npm

Now that I was inside my project folder, I used `npm` to initialize my project. This step creates a `package.json` file, which will store important information about my application, like its dependencies and configurations.

Here’s the command I used:

```bash
npm init
```

Running `npm init` prompted me to enter information about my project. I could press `Enter` to accept the default values, which is a quick way to get started. At the end, it asked me to confirm by typing `yes`, which I did. Now, my project was set up with a `package.json` file, ready to have packages added to it as needed.
![creating the package json file](https://github.com/user-attachments/assets/c639c08c-14e8-4a06-a6a1-2ca00ded0b15)

---
After this, I checked to verify if the `package.json` was successfully created by listing the files in the `Todo` directory:
```ls
```
![confirming package json file was created](https://github.com/user-attachments/assets/14669cc9-03f2-4a5a-85c6-c5e47e4c8625)


At this point, I had a dedicated project folder and a `package.json` file initialized. The next steps would involve installing essential packages, setting up Express.js for the backend, and connecting to a MongoDB database. But for now, my project environment was ready to go!
```
---

Here’s a write-up in Markdown format to explain the steps involved in implementing the MERN (MongoDB, Express, React, Node.js) stack. I’ll write it in a first-person format and explain each command and code purpose.

---

# Steps to Implement the MERN Stack

## Introduction

In this guide, I’ll walk you through the process of setting up the backend for a MERN stack application using Node.js and Express. Express is a framework that simplifies the development of Node.js applications by abstracting low-level details, allowing us to define application routes based on HTTP methods and URLs easily.

We’ll start by installing Express and creating a basic `index.js` file to get our server running. Additionally, we’ll use `dotenv` to manage environment variables securely.

---

## Step 1: Installing Express

To begin, I need to install Express, which will serve as the foundation for building our backend routes. Express allows us to handle server-side operations, such as defining API endpoints and managing requests and responses.

```bash
npm install express
```

This command installs Express in our project. With Express installed, I’ll have the tools to define and organize my routes and middleware functions.

---

## Step 2: Creating the Entry File (`index.js`)

After installing Express, I need to create a main file, `index.js`, which will be the entry point for our backend application. This file will contain the code to initialize the Express server.

```bash
touch index.js
```

The `touch` command creates a new file named `index.js` in the current directory. To verify that the file was successfully created, I can use the `ls` command to list all files in the directory.

---

## Step 3: Installing dotenv

To keep our sensitive information secure (such as API keys or database credentials), I’ll install the `dotenv` package. This package allows me to manage environment variables in a `.env` file, which will be loaded into our project.

```bash
npm install dotenv
```

The `dotenv` package makes it easy to work with environment variables, providing a safe way to keep confidential information separate from the code. We’ll load this module later in our `index.js` file to ensure that our server can access the environment variables.

---

## Step 4: Opening the `index.js` File

Now that I have all necessary packages, I’ll open `index.js` and start writing the code to set up the Express server. 

```bash
vim index.js
```

I’m using `vim` to edit `index.js`, but you can use any code editor, such as Visual Studio Code or Sublime Text, to open the file. Once open, I’ll add the required code to configure and start the Express server.

---

Here's the updated Step 5 of the write-up, incorporating the new code from your image, along with the next steps for saving and starting the server.

---

## Step 5: Writing the Server Code with CORS Configuration

In `index.js`, I’ll set up an Express server that includes configurations for handling Cross-Origin Resource Sharing (CORS). This allows our application to manage requests from different origins, which is particularly useful if the frontend is hosted on a different domain or port.

Here’s the code I’ll add to `index.js`:

```javascript
const express = require('express');
require('dotenv').config();

const app = express();
const port = process.env.PORT || 5000;

// Set up CORS headers to allow cross-origin requests
app.use((req, res, next) => {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});

// Define a basic route to test the server
app.use((req, res, next) => {
  res.send('Welcome to Express!');
});

// Start the server on the specified port
app.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

### Code Breakdown
1. **Loading Modules**:
   - `const express = require('express')`: Imports the Express framework.
   - `require('dotenv').config()`: Loads environment variables from a `.env` file.
2. **Initializing Express and Setting the Port**:
   - `const app = express()` creates a new Express application.
   - `const port = process.env.PORT || 5000` specifies that the server should use the port defined in `.env` or default to `5000` if none is provided.
3. **Setting CORS Headers**:
   - `app.use((req, res, next) => {...})` sets up headers to allow cross-origin requests by allowing any origin (`*`) and specifying acceptable request headers. This configuration is essential for enabling frontend applications hosted elsewhere to interact with the backend.
4. **Creating a Basic Route**:
   - `app.use((req, res, next) => { res.send('Welcome to Express!'); })` defines a route that sends a welcome message to test if the server is working correctly.
5. **Starting the Server**:
   - `app.listen(port, () => {...})` starts the server and listens on the specified port, logging a message to confirm the server is running.

---

## Step 6: Saving and Exiting Vim

If you’re using Vim to edit the file, you can save your changes and exit as follows:
- Press `:w` to save your changes.
- Then, use `:qa` to quit Vim and return to the terminal.

---

## Step 7: Starting the Server

Now it’s time to test if the server works. In the terminal, navigate to the directory where `index.js` is located, and start the server using:

```bash
node index.js
```

If everything is set up correctly, you should see the following message in the terminal:

```bash
Server running on port 5000
```

---

Here’s the continuation of the write-up, including the steps for testing the server with the public IP address and setting up the route structure.

---

## Step 8: Testing the Server Using Public IP

With the server running, I can test if it’s accessible from a browser. I’ll need the server’s Public IP or DNS name, especially if I’m running this on an EC2 instance.

1. **Finding the Public IP or DNS Name**:
   - I can locate this in the AWS web console under the details of my EC2 instance.
   - Alternatively, I can use the following commands in the terminal:
     - For Public IP: `curl -s http://169.254.169.254/latest/meta-data/public-ipv4`
     - For Public DNS: `curl -s http://169.254.169.254/latest/meta-data/public-hostname`

2. **Accessing the Server in the Browser**:
   - In my browser, I’ll type in the server's Public IP or DNS name, followed by `:5000` to specify the port, like this:
     ```
     http://<PublicIP-or-PublicDNS>:5000
     ```
   - If everything is working correctly, I should see the message “Welcome to Express!” displayed in the browser, confirming that the server is accessible from outside.

---

## Step 9: Defining Routes for To-Do Application

Now that the server is up and accessible, I’ll begin structuring the backend for the To-Do application. There are three primary actions the application needs to support:

1. **Create a New Task** – This will allow users to add new tasks to the list.
2. **Display All Tasks** – This will display a list of all current tasks.
3. **Delete a Completed Task** – This will remove a task once it’s completed.

Each action will correspond to a specific HTTP request method:
   - **POST** for creating new tasks,
   - **GET** for retrieving the list of tasks, and
   - **DELETE** for removing tasks.

To organize these routes, I’ll create a `routes` folder to store route files.

### Creating the Routes Folder

In the terminal, I’ll run the following commands to create and navigate to the `routes` folder:

```bash
mkdir routes
cd routes
```

This folder will house different files for each route or endpoint I need for the To-Do application.

---

Here's a markdown file that includes both the code and the explanations in a structured format:

```markdown
# MERN Stack Implementation: Setting Up Express Routes

In this step of setting up my MERN stack, I am focusing on creating and organizing routes for the API endpoints. This setup will allow my app to handle different types of requests in a structured and modular way, making the codebase more maintainable and easier to extend.

## 1. Creating the Routes Folder

First, I created a folder named `routes`, which will contain all the route definitions. This folder serves as the central place for organizing endpoint logic, particularly for handling CRUD (Create, Read, Update, Delete) operations.

```bash
$ mkdir routes
$ cd routes
```

## 2. Creating the API File

Next, I created a file named `api.js` within the `routes` folder. This file will contain the main API logic for handling requests to my application.

```bash
$ touch api.js
```

Once the file was created, I opened it using a text editor to add the code for the routes.

## 3. Writing the API Code

In the `api.js` file, I used Express to define three primary routes for handling requests related to "to-do" items: retrieving all items, adding new items, and deleting specific items by ID. Here is the code I used:

```javascript
const express = require('express');
const router = express.Router();

// Define a route to get all "to-do" items
router.get('/todos', (req, res, next) => {
  // Logic for retrieving to-do items will go here
});

// Define a route to add a new "to-do" item
router.post('/todos', (req, res, next) => {
  // Logic for adding a to-do item will go here
});

// Define a route to delete a "to-do" item by ID
router.delete('/todos/:id', (req, res, next) => {
  // Logic for deleting a to-do item by ID will go here
});

module.exports = router;
```

### Explanation of the Code

- **Importing Express and Setting Up the Router**: The first two lines import Express and create a `router` instance using `express.Router()`. This router instance allows me to modularize route handling within the `api.js` file.

  ```javascript
  const express = require('express');
  const router = express.Router();
  ```

- **GET Route (`/todos`)**: The `router.get('/todos', ...)` route is configured to handle `GET` requests to the `/todos` endpoint. This route will be responsible for retrieving all "to-do" items from the database. In a RESTful API, `GET` requests are used to retrieve data, so it’s appropriate for fetching data here.

  ```javascript
  router.get('/todos', (req, res, next) => {
    // Logic for retrieving to-do items will go here
  });
  ```

- **POST Route (`/todos`)**: The `router.post('/todos', ...)` route handles `POST` requests to the `/todos` endpoint, allowing me to add new "to-do" items. In RESTful conventions, `POST` is used for creating new resources, which is why it’s ideal for adding items here.

  ```javascript
  router.post('/todos', (req, res, next) => {
    // Logic for adding a to-do item will go here
  });
  ```

- **DELETE Route (`/todos/:id`)**: The `router.delete('/todos/:id', ...)` route handles `DELETE` requests and uses `:id` as a route parameter to target specific "to-do" items by their unique identifier. This allows for the deletion of individual items based on the `id` provided in the request URL.

  ```javascript
  router.delete('/todos/:id', (req, res, next) => {
    // Logic for deleting a to-do item by ID will go here
  });
  ```

- **Exporting the Router**: Finally, I exported the router using `module.exports = router;`, making it available for import in the main server file. This modular approach keeps the project organized by separating the route definitions from other logic.

  ```javascript
  module.exports = router;
  ```

## 4. Next Steps

With the routes in place, the next step is to create a `Models` directory. This directory will define the structure of the data my app will manage, further organizing the application’s code and establishing clear separation between data handling and route handling.

---

By following this approach, I am setting up a clean, modular, and organized codebase that is easy to maintain and scalable as the application grows.
```

This markdown file format clearly documents each step, including the code segments and their explanations.

Here's a first-person write-up explaining each step of setting up models in my MERN stack application, as shown in the provided image:

---

Now that my app will use MongoDB as its database, the next step is to create a model. In JavaScript-based applications, models play a central role in making the app interactive by providing structure to the data. Here, I’m setting up models to define the database schema, which essentially serves as a blueprint for how data will be stored in MongoDB.

The schema is crucial because it defines what fields and properties each MongoDB document will contain. Even though MongoDB is schema-less, using a schema helps me manage and validate data more effectively, especially when using virtual properties (additional data that doesn’t get stored in the database but is accessible within the app).

To get started, I need to install **Mongoose**, a Node.js package that makes it easier to work with MongoDB by providing a straightforward way to create and manage schemas. Here’s how I went about it:

### Step 1: Install Mongoose

First, I navigated back to my main project folder (Todo) using the command `cd ..`. Then, I installed Mongoose with:

```bash
npm install mongoose
```

This command installs Mongoose, allowing me to set up schemas and models for MongoDB. With Mongoose, I can define schemas to structure my data, validate it, and apply different methods to interact with the database seamlessly.

### Step 2: Create the Models Folder

After installing Mongoose, I created a new folder called `models` using the following command:

```bash
mkdir models
```

This `models` folder is where I’ll keep all my models for the app. Organizing models in a separate directory keeps the project structure clean, which is especially helpful as the app grows and more models are added.

### Step 3: Navigate to the Models Folder and Create the Model File

Next, I navigated into the `models` folder with `cd models` and created a file named `todo.js`:

```bash
touch todo.js
```

The `todo.js` file is where I’ll define the schema for the "to-do" items. This schema will specify what fields each to-do item will have, such as the task description, completion status, timestamps, and more. Creating this file keeps each model's logic separate from the main application logic, making it easier to maintain and update.

### Tip: Combining Commands

I learned that all three steps above can be combined into one line for convenience using the `&&` operator. Here’s what that looks like:

```bash
mkdir models && cd models && touch todo.js
```

Using `&&` allows me to execute multiple commands in sequence. This line creates the `models` folder, navigates into it, and then creates the `todo.js` file within it, all in one go.

---

By organizing my code into models and defining a schema, I’m creating a solid foundation for managing and validating the data in my app. This setup will allow me to scale the app smoothly while keeping the codebase organized and manageable. Now, I’m ready to move on to defining the actual structure of the "to-do" schema within `todo.js`.

Continuing with my MERN stack setup, I moved on to define the structure of my "to-do" data in the `todo.js` file. Here’s how I set up the schema and model for the to-do items:

---

### Step 4: Define the Schema in `todo.js`

In the `todo.js` file, I first imported Mongoose and then defined the schema for my to-do items. The code below shows how I accomplished this:

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

// Create schema for todo
const TodoSchema = new Schema({
  action: {
    type: String,
    required: [true, 'The todo text field is required']
  }
});

// Create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```

### Explanation of the Code

- **Importing Mongoose and Schema**: First, I imported Mongoose and extracted `Schema` from it. The `Schema` object allows me to define the structure of each to-do item in a controlled way, setting requirements and data types.

  ```javascript
  const mongoose = require('mongoose');
  const Schema = mongoose.Schema;
  ```

- **Defining the TodoSchema**: Using `new Schema`, I created `TodoSchema`, which specifies the structure for each to-do item. In this schema, I defined an `action` field, which represents the actual to-do text. I set it as a string and made it a required field with a custom error message to ensure every to-do item has content.

  ```javascript
  const TodoSchema = new Schema({
    action: {
      type: String,
      required: [true, 'The todo text field is required']
    }
  });
  ```

  The `required` option ensures that any to-do item saved to the database must have text in the `action` field. This requirement helps maintain data integrity by preventing empty to-do items from being added.

- **Creating the Model**: With the schema in place, I used `mongoose.model()` to create a model named `Todo`, passing in the schema (`TodoSchema`). This model acts as a constructor for to-do items, allowing me to interact with to-do data in the database, like adding, updating, or deleting items.

  ```javascript
  const Todo = mongoose.model('todo', TodoSchema);
  ```

  By creating this model, I’m making it easy to perform database operations on the "to-do" collection without writing raw MongoDB commands. This is a key advantage of using Mongoose, as it streamlines database interactions and reduces the chances of errors.

- **Exporting the Model**: Finally, I exported the `Todo` model with `module.exports`. Exporting the model allows me to import and use it in other files, such as when defining routes in `api.js`.

  ```javascript
  module.exports = Todo;
  ```

---

Now that the model is set up, the next step is to update the `api.js` file in the `routes` directory to use this model. This will enable my routes to create, retrieve, and delete to-do items in the database, making the app fully functional for managing tasks.
