**Title**: MERN Stack Implementation Guide

---

### Table of Contents

1. **Prerequisites**: Launching an EC2 Server and SSH Connection
2. **Backend Configuration**: Update and Upgrade Ubuntu, Node.js Installation
3. **Application Code Setup**: Creating Project Directory, Initializing npm Project
4. **Installing Express.js**: Setting Up Server, Configuring CORS, Starting Server
5. **Adding Routes**: API Routes for CRUD Operations in Todo App
6. **MongoDB Setup**: Cluster and Collection Setup, Environment Variables
7. **Testing Backend**: Using Postman for API Testing
8. **Frontend Creation**: Creating React App, Configuring `concurrently` and `nodemon`
9. **Proxy and Application Run**: Proxy Configuration, Running Both Servers
10. **Creating React Components**: Input, ListTodo, and Todo Components
11. **Styling and Integration**: Custom Styles, Integrating Components in `App.js`

---
### EC2 and SSH Connection

1. **Launching an EC2 Instance**:
   - Log in to the AWS Management Console.
   - Go to **EC2 Dashboard** and click **Launch Instance**.
   - Choose an Amazon Machine Image (AMI), typically an Ubuntu Server for MERN applications.
   - Select an instance type (e.g., t2.micro for free-tier).
   - Configure instance settings, storage, and create or select a key pair (for SSH).
   - Review settings and launch the instance.

2. **Connecting via SSH on Windows**:
   - Open the **Command Prompt** (or use **PowerShell**).
   - Use the SSH command to connect to your instance:
     ```bash
     ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
     ```
   - Replace `"your-key.pem"` with your private key file path and `your-ec2-public-ip` with your instance’s public IP address.
   - Accept the security prompt, and you’re connected to your EC2 instance.

# Backend Configuration

### Step 1: Update Ubuntu
   - **Command**: `sudo apt update`
   - **Explanation**: First, I updated my system’s package list. This step is important because it lets my system know about the latest versions of available packages and dependencies. By doing this, I avoid compatibility issues and ensure that I have access to the latest security patches and software improvements.

### Step 2: Upgrade Ubuntu
   - **Command**: `sudo apt upgrade`
   - **Explanation**: Next, I upgraded all the packages to their latest versions. This is a crucial step to keep my system secure, stable, and optimized for performance. Having an up-to-date system is particularly important for a server environment, as it ensures I’m not working with outdated or vulnerable software.

### Step 3: Add the Node.js Repository
   - **Command**: `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`
   - **Explanation**: I used this command to download and run a setup script from NodeSource, which adds the repository for Node.js version 18.x to my system. This step is necessary because the default Ubuntu repositories might not have the latest version of Node.js required for a MERN stack application. By adding this repository, I ensure I can install the specific version I need.

### Step 4: Install Node.js
   - **Command**: `sudo apt-get install -y nodejs`
   - **Explanation**: After adding the repository, I installed Node.js along with npm (Node Package Manager). Node.js is a JavaScript runtime that allows me to run JavaScript on the server side, which is essential for the backend of a MERN stack application. npm, which comes with Node.js, is necessary for managing dependencies and packages, making it a critical part of my setup.

### Step 5: Verify Node.js Installation
   - **Command**: `node -v`
   - **Explanation**: To confirm that Node.js installed successfully, I checked the version by running this command. Verifying the installation is important to ensure that I have the correct version of Node.js installed and that there were no issues during the installation process.

### Step 6: Verify npm Installation
   - **Command**: `npm -v`
   - **Explanation**: Finally, I verified the npm installation by checking its version. This step is important because npm will be used throughout the project to install and manage various Node modules and dependencies. Confirming the version assures me that npm is ready for use in managing the backend environment effectively.

![node and npm versions](https://github.com/user-attachments/assets/17274cb8-9a49-4d43-adac-ba6a4cf5f173)


# Application Code Setup

To start building my To-Do project, I set up the necessary project directory and initialized it as an npm project. Below are the steps I followed, along with explanations for each.

---

### Step 1: Create a New Directory
   - **Command**: `mkdir Todo`
   - **Explanation**: I created a new directory called `Todo` to store all the files and resources related to my project. Organizing my files within a dedicated folder makes it easier to manage and keep track of everything for this specific project.

### Step 2: Verify the Directory Creation
   - **Command**: `ls`
   - **Explanation**: To confirm that the `Todo` directory was successfully created, I used the `ls` command to list all items in the current directory. Verifying the creation of directories and files is a good habit as it ensures each step is executed correctly.

![image](https://github.com/user-attachments/assets/efbb00ce-088b-4e38-9fb0-0b77d59650f5)


### Step 4: Change Directory to `Todo`
   - **Command**: `cd Todo`
   - **Explanation**: I navigated into the `Todo` directory to work within this newly created folder. Changing to the correct directory ensures that any commands I run next, such as initializing npm, will apply to this specific project folder.

### Step 5: Initialize the Project with npm
   - **Command**: `npm init`
   - **Explanation**: I used `npm init` to initialize the project, which creates a new file named `package.json`. This file holds metadata about the project, such as its name, version, and dependencies. The prompts guided me through providing basic information, but I could accept the default values by pressing "Enter." Setting up `package.json` is essential because it enables me to manage project dependencies easily with npm.
![image](https://github.com/user-attachments/assets/e13635e6-4638-4efd-8e30-4266d46c81cb)

### Step 6: Complete `package.json` Setup
   - **Process**: After completing the prompts, npm generated a `package.json` file with details like the project name, version, and default scripts. I confirmed the setup by typing `yes`.
   - **Explanation**: The `package.json` file is vital for tracking project dependencies and configurations. As I add packages and dependencies, they will be recorded here, making it easier to manage the project and share it with others.

---
![image](https://github.com/user-attachments/assets/ebebcb5e-1a26-4270-bba1-1b130bcdc5f8)

With these steps completed, my project is now set up with an organized directory structure and a `package.json` file, which will allow me to add dependencies and manage the project efficiently.

# Installing Express.js

Express.js is a powerful framework for Node.js that simplifies the development of server-side applications. By handling many low-level details for us, it allows us to focus on building features instead of handling the boilerplate code that developers would normally write.

---

### Step 1: Install Express.js
   - **Command**: `npm install express`
   - **Explanation**: I started by installing Express.js using npm. Express is essential because it allows us to set up a server, handle HTTP requests, and define routes in a straightforward way. This installation makes it available as a dependency, which is tracked in the `package.json` file.
![image](https://github.com/user-attachments/assets/ba6d08d1-6209-43c8-ac91-e9a15f25c083)

### Step 2: Create the `index.js` File
   - **Command**: `touch index.js`
   - **Explanation**: I created a new file named `index.js` to serve as the main entry point for my server application. This file will contain the server code for our application, which Express will use to listen for incoming requests.

### Step 3: Confirm File Creation
   - **Command**: `ls`
   - **Explanation**: To confirm that the `index.js` file was successfully created, I used `ls` to list the files in the current directory. This ensures that the file is in place before I begin editing it.

### Step 4: Install the `dotenv` Module
   - **Command**: `npm install dotenv`
   - **Explanation**: I installed the `dotenv` module to allow my application to load environment variables from a `.env` file. This module is important for keeping sensitive information, like API keys or port numbers, outside the codebase. It helps enhance security and makes it easier to change configurations without modifying the code.
![image](https://github.com/user-attachments/assets/62fb47a5-39d8-4a35-aa14-3016d583d672)

### Step 5: Open `index.js` for Editing
   - **Command**: `vim index.js`
   - **Explanation**: I opened `index.js` in the Vim text editor to add the initial server code. Editing this file allows me to define how my server should respond to requests.

### Step 6: Add Server Code
   - **Code**:
     ```javascript
     const express = require('express');
     require('dotenv').config();

     const app = express();
     const port = process.env.PORT || 5000;

     app.use((req, res, next) => {
         res.header("Access-Control-Allow-Origin", "*");
         res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
         next();
     });

     app.use((req, res, next) => {
         res.send('Welcome to Express!');
     });

     app.listen(port, () => {
         console.log(`Server running on port ${port}`);
     });
     ```
     ![image](https://github.com/user-attachments/assets/4f5e082a-f0a9-43c7-9e6a-1deb49285704)

**Explanation**: In this code:
     - I imported Express and initialized it with `const app = express()`.
     - The `dotenv` module loads environment variables, allowing me to set the port dynamically using `process.env.PORT || 5000`.
     - I added middleware to handle CORS (Cross-Origin Resource Sharing), which allows the server to handle requests from different origins by setting headers.
     - I defined a basic route that responds with "Welcome to Express!" when accessed.
     - Finally, `app.listen(port)` starts the server and listens for connections on the specified port, logging a message to confirm the server is running.

### Step 7: Note on Port Configuration
   - **Explanation**: I set the server to listen on port 5000, which will be important when I test the server in the browser or with tools like Postman. Setting the port through `process.env.PORT` allows for flexibility, as I can specify a different port without changing the code.

### Step 8: Start the Server
   - **Command**: `node index.js`
   - **Explanation**: I start the server by running `node index.js` in the terminal. If everything is set up correctly, I should see the message `Server running on port 5000` in the terminal. This indicates that my Express application is up and listening for requests.

---
![image](https://github.com/user-attachments/assets/a04855d1-f4ec-4c66-91c1-fca2fd64ce77)

### Step 9: Configure Inbound Rules on AWS EC2
   - **Explanation**: To access the server externally, I need to allow inbound traffic on port 5000 in the EC2 Security Groups. This ensures that my server can be reached from a web browser or other clients.

   - **Steps**:
     1. In the **AWS Management Console**, I navigate to **EC2** and select **Security Groups** associated with my instance.
     2. I click **Edit Inbound Rules** to add a new rule for port 5000.
     3. I configure the rule as follows:
        - **Type**: Custom TCP
        - **Port Range**: 5000
        - **Source**: Anywhere (0.0.0.0/0, ::/0) *(Note: For security, in production, restrict access to specific IPs)*
        - **Description**: Allows inbound traffic on port 5000 for Express application testing.
     4. After adding the rule, I click **Save Rules** to apply the changes.

---
![image](https://github.com/user-attachments/assets/f33a6a21-fe7f-4372-b9ed-c02c1d95114b)
### Step 10: Access the Server in the Browser
   - **Explanation**: Now that port 5000 is open, I can test the server by accessing it through my browser. I use my server’s public IP or DNS name followed by `:5000` to specify the port.

   - **URL Format**:
     ```
     http://35.164.156.218:5000
     ```

   - **Expected Outcome**: When I navigate to this URL, I should see a response message, such as "Welcome to Express!" This confirms that the server is accessible and responding to requests over the internet.

---
![image](https://github.com/user-attachments/assets/40fcfd57-81a2-4fe0-822b-5ef7a73129c2)

### Next Steps
With the server running and accessible on port 5000, I’m ready to proceed with connecting Express to MongoDB and setting up additional routes and functionality. This foundational setup allows me to build out the backend components of my MERN stack application.

---

By following these steps, I’ve successfully set up an Express.js server that is accessible externally, laying the groundwork for further development in my MERN stack project.

# Adding Routes for the To-Do Application in Express.js
### Step 12: Create a `routes` Folder
   - **Command**: `mkdir routes`
   - **Explanation**: I start by creating a new directory called `routes`. This folder will contain files that define the various endpoints for the To-Do application.

### Step 13: Navigate to the `routes` Directory
   - **Command**: `cd routes`
   - **Explanation**: After creating the `routes` directory, I navigate into it. This is where I’ll add the route definitions for handling tasks.

### Step 14: Create the `api.js` File
   - **Command**: `touch api.js`
   - **Explanation**: I create a file named `api.js` within the `routes` directory. This file will define the API endpoints that allow users to interact with the To-Do application.
![image](https://github.com/user-attachments/assets/57ab4016-a155-4649-aebd-8d4e2471511d)

### Step 15: Open `api.js` for Editing
   - **Command**: `vim api.js`
   - **Explanation**: I open `api.js` in Vim to add the code that will define the routes for the application.
   - **Code**:
     ```javascript
     const express = require('express');
     const router = express.Router();

     // GET route to fetch all tasks
     router.get('/todos', (req, res, next) => {
         // Logic to retrieve all tasks
     });

     // POST route to create a new task
     router.post('/todos', (req, res, next) => {
         // Logic to create a new task
     });

     // DELETE route to delete a task by ID
     router.delete('/todos/:id', (req, res, next) => {
         // Logic to delete a task
     });

     module.exports = router;
     ```
   - **Explanation**:
     - I import Express and create a `router` instance using `express.Router()`.
     - I define three routes:
       - **GET** `/todos`: Retrieves all tasks.
       - **POST** `/todos`: Creates a new task.
       - **DELETE** `/todos/:id`: Deletes a specific task based on its ID.
     - Each route is currently a placeholder and will need actual logic to perform the operations.
     - Finally, I export the `router` module so it can be imported and used in the main server file.

### Next Steps
With the basic routes set up, I’m ready to move forward with adding functionality to these routes by defining models and connecting to a database. The next step involves creating a **Models** directory, where I’ll define the schema for a task to interact with the database effectively.

---

### Step 17: Install Mongoose
   - **Command**: `npm install mongoose`
   - **Explanation**: I installed Mongoose to simplify database interactions. Mongoose allows me to create schemas that outline how documents in MongoDB should be structured. It also provides a way to model data, ensuring consistency and simplifying queries.

### Step 18: Create the `models` Directory
   - **Command**: `mkdir models`
   - **Explanation**: I created a new directory named `models`. This folder will contain files that define the data structures, or schemas, for the To-Do application's database collections.

### Step 19: Navigate to the `models` Directory
   - **Command**: `cd models`
   - **Explanation**: After creating the `models` directory, I navigate into it to start creating the model for my application.

### Step 20: Create the `todo.js` Model File
   - **Command**: `touch todo.js`
   - **Explanation**: I created a file named `todo.js` inside the `models` directory. This file will contain the Mongoose schema and model for each task in the To-Do application.
![image](https://github.com/user-attachments/assets/79937b39-398d-4fe6-8798-92bc7748060e)

### Step 21: Define the Task Schema in `todo.js`
![image](https://github.com/user-attachments/assets/811b45d0-9cb9-4791-8a32-2a5a7d44f9e1)

     ```
   - **Explanation**:
     - I imported Mongoose and created a new schema using `mongoose.Schema`.
     - The schema `todoSchema` includes a single field, **action**, which stores the task's text. I set the `type` of this field to `String` and made it **required**, with a custom error message if left empty.
     - I then created a model named `Todo` based on `todoSchema`. This model provides an interface to interact with the `todos` collection in MongoDB.
     - Finally, I exported the `Todo` model so that it can be used in other parts of the application.

### Step 22: Update Routes to Use the New Model
   - **Explanation**: Now that the model is created, I need to update the routes in `api.js` to make use of this model. By integrating the `Todo` model, the routes can interact directly with the MongoDB database to create, retrieve, and delete tasks.
![image](https://github.com/user-attachments/assets/9011cf0b-d9d2-4976-87ae-54383359b140)

   - **Explanation of Updated Routes**:
     - **GET** `/todos`: Uses `Todo.find()` to fetch all tasks from the database, returning only the `action` field for each task.
     - **POST** `/todos`: Checks if the `action` field is provided in the request body. If present, it creates a new task in the database with `Todo.create()`. If the field is empty, it returns an error message.
     - **DELETE** `/todos/:id`: Finds and deletes a task by its ID using `Todo.findOneAndDelete()`, allowing specific tasks to be removed from the database.

---
# MongoDB Setup for MERN Stack

## Step 1: Set Up MongoDB Database
This was straight forward:

1. I Signed up on [mLab](https://www.mlab.com).
2. Chose **AWS** as the cloud provider and select a nearby region.

## Step 2: Create a Cluster

1. Went to the **Clusters** section.
2. Click **Create Cluster**.
3. Chose the free tier options and confirm the setup.

## Step 3: Allow Access from Anywhere (For Testing)

1. I went to **Network Access** in the dashboard.
2. Clicked **Add IP Address**.
3. And Chose **Allow access from anywhere (0.0.0.0/0)**.

> **Note**: This setting is only for testing and should not be used in production.

## Step 4: Create a Database and Collection

1. In the **Databases** section, click **Create Database**.
2. I inserted the name of the database (SampleTodoApp) and clicked **Create**.
3. Go to **Collections** and click **Add Collection** to create your first collection.
![image](https://github.com/user-attachments/assets/8aeb46d5-0d8a-4caf-843c-ce577c92b1df)
## Step 5: Set Up Environment Variables

1. In the project folder, I created a file called `.env`.
2. Then I added the MongoDB connection string in this format:

   ```plaintext
   mongodb+srv://macbeth:<Password.1>@sampletodoapp.wg4iw.mongodb.net/?retryWrites=true&w=majority&appName=SampleTodoApp


## Step 7: Update `index.js` File to Use `.env`

To connect Node.js to MongoDB using the `.env` file, follow these steps:

1. I Opened the `index.js` file in the root of my project.
2. Delete all existing content in `index.js`.
3. Add the following code:

   ```javascript
   const express = require("express");
   const mongoose = require("mongoose");
   const dotenv = require("dotenv");
   const app = express();

   // Load environment variables from .env file
   dotenv.config();

   // Connect to MongoDB using the connection string from .env
   mongoose.connect(process.env.MONGO_URI, {
       useNewUrlParser: true,
       useUnifiedTopology: true,
   })
   .then(() => console.log("Connected to MongoDB"))
   .catch((error) => console.error("Connection error", error));

   // Start the server
   app.listen(3000, () => {
       console.log("Server is running on port 3000");
   });

# Testing Backend with Postman in a MERN Stack Application

Since we’re testing the backend of our MERN app without a frontend, we use **Postman** to make HTTP requests directly to the backend. This allows us to verify that our API endpoints work as expected.

## Step 1: Set Up Postman

1. I Downloaded and installed [Postman](https://www.postman.com/downloads/).
2. Open Postman and get familiar with the interface.

```markdown
## Step 2: Test Adding a New Task with a POST Request

1. Created a new **POST** request in Postman.
2. In the **Headers** tab, added:
   - **Key**: `Content-Type`
   - **Value**: `application/json`
3. Went to the **Body** tab, selected **raw**, chose **JSON** format, and entered:
   ```json
   {
       "action": "Finish project report"
   }
   ```
4. Set the **URL** to the endpoint for creating tasks: `http://35.164.156.218:5000/api/todos`

![Postman Request Example](https://github.com/user-attachments/assets/aa2b1447-c0b1-4acb-aa1c-01b9baddf1ac)

---

# Frontend Creation and Running the React App

## Step 1: Frontend Creation

With the backend setup completed for the To-Do app, the next step is to build the user interface using React. This interface will enable users to interact with the application through a browser and send requests to the backend API.

### 1. Creating the React App

To scaffold a new React app, use the `create-react-app` tool:
```bash
npx create-react-app client
```

### 2. Installing `concurrently`

Install `concurrently` to run multiple npm commands in a single terminal:
```bash
npm install concurrently --save
```
**Explanation**:
- **concurrently** allows simultaneous execution of npm commands, useful for running both the backend server and the frontend React app at the same time. This improves development efficiency by managing both processes in a single terminal.

### 3. Installing `nodemon`

Install `nodemon` for automatic server restarts on file changes:
```bash
npm install nodemon --save-dev
```
**Explanation**:
- **nodemon** automatically restarts the server when source files change, eliminating the need for manual restarts.
- The `--save-dev` flag adds `nodemon` as a development dependency, as it's not needed in production.

## Step 2: Configuring Scripts in `package.json`

After installing the dependencies, update `package.json` to simplify the process of starting both servers:
![package.json Configuration](https://github.com/user-attachments/assets/5a3921f0-219f-416c-8ef4-de0e2e829349)

## Step 3: Configuring Proxy in `package.json`

To ensure the React app interacts seamlessly with the backend API, configure a proxy in the frontend. This enables API calls from the frontend to the backend server without specifying the full backend URL each time.

1. Navigate to the `client` directory:
   ```bash
   cd client
   ```
   **Explanation**: Changes the current directory to `client`, where the React application code is located.

2. Open the `package.json` file in the client directory and add a proxy configuration:
   ```json
   "proxy": "http://localhost:5000"
   ```
   ![Proxy Configuration](https://github.com/user-attachments/assets/6050c677-8f50-461a-8433-8079b8738731)

**Explanation**:
- Adding `"proxy": "http://localhost:5000"` forwards API requests from the React app running on `localhost:3000` to the backend server on `localhost:5000`.
- This allows the use of relative paths (e.g., `/api/todos`) in the frontend code, avoiding cross-origin issues (CORS) and simplifying code.

## Step 4: Running the Application

With everything configured, start the application (both backend and frontend) using the command:
```bash
npm run dev
```
![Running Application](https://github.com/user-attachments/assets/ae9acef2-ac6e-4c56-bbed-d6e28c7e844f)

**Explanation**:
- This command executes the `"dev"` script in `package.json`, launching both the backend server and the frontend React app concurrently.
- Using `concurrently`, logs from both services are displayed in a single terminal, making it easier to debug and monitor.

**Outcome**:
- The backend server starts at `http://localhost:5000`, and the React frontend is accessible at `http://localhost:3000`.
- Visiting `http://localhost:3000` in a browser allows interaction with the React app, which communicates with the backend API.
```

# Creating React Components for the Todo App

## Introduction

In ls
this final stage of my MERN stack application, I will create the React components needed for the Todo app. One of the strengths of React is its use of reusable and modular components, making the code cleaner and easier to maintain. For this Todo app, I will create three components:
1. **Input.js** - A stateful component for adding new todos.
2. **ListTodo.js** - A stateful component for displaying the list of todos.
3. **Todo.js** - A stateless component that represents a single todo item.

These components will be placed inside a new folder called `components` within the `src` directory of my React app.

## Step 1: Navigate to the `client` Folder

I start by navigating to the `client` directory, where the React application is located.

```bash
cd client
```

### Explanation:
- This command changes the current directory to the `client` folder, which contains all the frontend code of the application.

## Step 2: Navigate to the `src` Folder

Next, I move into the `src` directory, where the main source code for the React app is located.

```bash
cd src
```

### Explanation:
- The `src` directory is where the core components and logic of the React application are written. It contains files like `App.js`, `index.js`, and other components.
![image](https://github.com/user-attachments/assets/144e51e3-dd24-4578-b79f-add413090d8f)

## Step 3: Create the `components` Folder

To keep the code organized, I create a new folder named `components` inside the `src` directory.

```bash
mkdir components
```

### Explanation:
- The `mkdir` command creates a new folder called `components`.
- This folder will house all the React components, making the project structure clean and maintainable.

## Step 4: Navigate to the `components` Folder

I then move into the newly created `components` directory.

```bash
cd components
```

### Explanation:
- The `cd components` command switches the current working directory to `components`, where I will create the component files.

## Step 5: Create Component Files

Inside the `components` folder, I create three files: `Input.js`, `ListTodo.js`, and `Todo.js`.

```bash
touch Input.js ListTodo.js Todo.js
```
![image](https://github.com/user-attachments/assets/51209a03-384b-4650-b9d1-48b8438aae6b)

### Explanation:
- The `touch` command creates new files. In this case, I am creating three files at once:
  - `Input.js` for handling user input.
  - `ListTodo.js` for displaying the list of todos.
  - `Todo.js` for representing individual todo items.

## Step 6: Implementing the `Input.js` Component

Now, I open the `Input.js` file to define the component for adding new todos.

```bash
vi Input.js
```

### Explanation:
- This command opens the `Input.js` file in a text editor (`vi` in this case), allowing me to write the component's code.

![image](https://github.com/user-attachments/assets/ba2c5c59-0fb2-4e4a-b70e-c9c2cda32d2a)

### Explanation:
- **Imports:** 
  - `React` and `Component` are imported to define the component.
  - `axios` is imported for making HTTP requests to the backend API.
- **State:** 
  - The component has a state variable `action`, which stores the current input value from the user.
- **addTodo Function:**
  - This function is called when the "Add Todo" button is clicked.
  - It checks if the input field is not empty and then sends a POST request to the backend API (`/api/todos`) to create a new todo.
  - If the request is successful, it clears the input field and fetches the updated list of todos by calling `this.props.getTodos()`.
- **handleChange Function:**
  - This function updates the component's state as the user types in the input field.
- **Render Method:**
  - Renders an input field and a button. The input value is controlled by the component's state.

## Step 7: Implementing the `ListTodo.js` Component

Next, I define the `ListTodo.js` component, which will display the list of todos.

### Open `ListTodo.js`:

```bash
vi ListTodo.js
```

### Code for `ListTodo.js`:

![image](https://github.com/user-attachments/assets/c93b374a-6183-4e36-bba3-918daba6cac1)

## Step 8: Implementing the `Todo.js` Component

Lastly, I define the `Todo.js` component, which represents individual todo items.

### Open `Todo.js`:

```bash
vi Todo.js
```
### Code for `Todo.js`:
![image](https://github.com/user-attachments/assets/c64ddfdc-7248-4991-909d-c81404637165)

### Explanation:
- This is a simple stateless functional component that takes a `todo` prop and displays its `action` text.
- It serves as a reusable component for rendering individual todos.

```markdown
# Adjusting the `App.js` and `App.css` Files in the React Application

Now that I have created the necessary components, the next step is to integrate them into the main application file, `App.js`. Additionally, I will make some styling adjustments in `App.css` to enhance the look and feel of the Todo app.

## Step 1: Navigate to the `src` Folder

I begin by navigating back to the `src` folder where the main application files are located.

## Step 1: Open and Edit `App.js`

Next, I open the `App.js` file using a text editor to make the necessary adjustments.

```bash
vi App.js
```

## Step 3: Update the `App.js` File

I replace the existing code in `App.js` with the following:
![image](https://github.com/user-attachments/assets/b6957034-abf6-45ef-a91a-80043e9de717)


### Explanation:
- **Imports:**
  - `React` is imported to use React's features and hooks.
  - `Todo` component is imported from the `components` folder to be rendered as the main component.
  - `./App.css` is imported for applying styles defined in the CSS file.
- **Functional Component:**
  - I use an arrow function to define a simple functional component, `App`.
- **Render:**
  - The `App` component renders a `div` with a class name "App" and includes the `Todo` component within it.
- **Export:**
  - Finally, I export the `App` component as the default export.

### Why This Change?
- By including the `Todo` component directly in `App.js`, I ensure that it is the main visible component when the app loads.
- This structure sets up the application to use the Todo feature as the central functionality.
Then I dsaved and exited.

## Step 4: Open `App.css` for Styling

Next, I open the `App.css` file to add custom styles.

```bash
vi App.css
```
And go ahead to Update `App.css` with the Following Styles

I replace the content of `App.css` with the following CSS code:

```css
.App {
  text-align: center;
  font-size: calc(10px + 2vmin);
  width: 50%;
  margin-left: auto;
  margin-right: auto;
}

input {
  height: 40px;
  width: 50%;
  border: none;
  border-bottom: 2px #101113 solid;
  background: none;
  font-size: 1.5em;
  color: #787878;
}

input:focus {
  outline: none;
}

button {
  height: 45px;
  width: 50px;
  margin-left: 10px;
  background: #101113;
  color: #fafafa;
  border: none;
  font-size: 1em;
  cursor: pointer;
}

button:focus {
  outline: none;
}

button:hover {
  background: #3c3c3c;
}

p {
  font-size: 1.5em;
  color: #3c3c3c;
}
```
![image](https://github.com/user-attachments/assets/828c58b5-caaa-4b68-ab33-c94f4d16217c)
![image](https://github.com/user-attachments/assets/969ddfa7-d425-45f7-801e-9c97ef6ae1d4)

### Explanation:
- **.App Class:**
  - Centers the content, sets a responsive font size, and limits the width to 50% of the viewport width for a clean look.
- **input Styles:**
  - Styles the input field for adding todos with a height of 40px, no border, and a solid bottom border.
  - The `input:focus` style removes the default focus outline.
- **button Styles:**
  - Customizes the appearance of buttons with a dark background and white text, providing a clean and modern look.
  - `button:hover` changes the background on hover for a better user experience.
- **p Styles:**
  - Sets the font size and color for paragraphs, ensuring consistency in the app's appearance.

Now, I can start the React application to test the user interface and see if everything is displayed correctly. So I run

```bash
npm start
```

### Final Check: Application was finally deployed successfully
![Finally Deployed Successfully](https://github.com/user-attachments/assets/8c67d17b-6c3a-4133-a7dd-4f773dac6b8e)

##Lessons Learned

    Setting up a full MERN stack application provides hands-on experience with building and connecting both the frontend and backend using industry-standard tools.
    Leveraging tools like Express, Mongoose, and Postman enhances efficiency in backend configuration and API testing.

##Challenges Encountered and Solutions

    Challenge: Configuring secure access on EC2 with proper inbound rules.
        Solution: Carefully configuring EC2 Security Groups to allow access only on necessary ports improved security while enabling remote testing.

    Challenge: Managing dependencies and environment configurations.
        Solution: Using .env files to securely manage environment variables and tools like concurrently to streamline development processes across both servers.

    Challenge: Cross-Origin Resource Sharing (CORS) issues during frontend-backend communication.
        Solution: Adding CORS headers in the Express backend resolved errors, allowing the React frontend to communicate with the backend securely.
