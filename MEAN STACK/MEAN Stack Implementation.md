# Implementing the MEAN Stack
# Table of Contents  
 
1. **Components of the MEAN Stack**  
2. **Setup and Installation**  
3. **Implementation Process**  
4. **Lessons Learned, Challenges Faced**  

## Initiate Ec2-Instance connect
I used Ec2-instance connect for this procedure.
![001 Conected to CLI](https://github.com/user-attachments/assets/87b9945d-db18-4ddf-a9b2-3bebc37b6be3)

## Step 1: Install Node.js

Node.js is a JavaScript runtime built on Chrome's V8 engine. It’s a crucial component of the MEAN stack because it lets me run JavaScript code on the server. I’ll also use it as the foundation for Express.js, which will help me create APIs and web applications.

### What I Did:

1. **Updated Ubuntu**  
   First, I made sure that the system package index was up to date. This step prevents issues caused by outdated package references. Here’s the command I ran:  
   ```bash
   sudo apt update
   ```

2. **Upgraded Ubuntu**  
   Next, I upgraded the system so that all the packages would be on their latest versions. This is important for security and compatibility.  
   ```bash
   sudo apt upgrade
   ```

3. **Added Certificates**  
   After that, I installed the necessary certificates and tools to access secure external sources. This ensured I could securely download the Node.js setup script.  
   ```bash
   sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
   ```
   Then, I fetched and executed the Node.js setup script for the latest version (in this case, Node.js 18.x). This script configured the package manager so I could install Node.js:  
   ```bash
   curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
   ```
![002 Cerificates added successfully](https://github.com/user-attachments/assets/607df338-df29-4290-8300-cda0b3d037a7)

4. **Installed Node.js**  
   Finally, I installed Node.js from the configured repository. This command also installed `npm`, which I’ll need for managing JavaScript dependencies in my project:  
   ```bash
   sudo apt install -y nodejs
   ```

### How I Verified:
After installing, I checked to make sure both Node.js and `npm` were installed correctly by verifying their versions:  
```bash
node -v
npm -v
```
![000 node and npm verified](https://github.com/user-attachments/assets/e4f344bf-b227-433b-a123-171516a83b66)

**Why This Step Is Important**:
Installing Node.js and `npm` is the foundation of my MEAN stack application. These tools allow me to build, test, and manage the backend efficiently while ensuring I can handle JavaScript dependencies for my project.

---

---

## Step 2: Install MongoDB

MongoDB is a NoSQL database that stores data in flexible, JSON-like documents. This structure is perfect for modern applications because the fields in a database can vary between documents and adapt over time. For my MEAN stack application, MongoDB will hold the backend data — in this case, I’m preparing it to store book records like book name, ISBN number, author, and the number of pages.

### What I Did:

1. **Installed Required Tools**  
   I started by installing the tools needed to add the MongoDB repository securely:  
   ```bash
   sudo apt-get install -y gnupg curl
   ```
   Then, I downloaded the MongoDB server's public GPG key and saved it securely. This ensures that the repository I’m using is authentic and untampered:  
   ```bash
   curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
   ```

2. **Added MongoDB Repository**  
   I created a MongoDB repository entry for my Ubuntu system to access MongoDB 7.0 packages. This configuration helps the system locate the right version of MongoDB for installation:  
   ```bash
   echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
   ```

3. **Updated the System Package Index**  
   After setting up the repository, I updated the package index to include MongoDB packages from the new repository:  
   ```bash
   sudo apt update
   ```

4. **Installed MongoDB**  
   Now that everything was configured, I installed MongoDB:  
   ```bash
   sudo apt-get install -y mongodb-org
   ```

5. **Started the MongoDB Server**  
   Once MongoDB was installed, I started the MongoDB service to ensure it was running and ready to accept connections:  
   ```bash
   sudo service mongodb start
   ```

6. **Verified MongoDB Status**  
   To confirm everything was working properly, I checked the status of the MongoDB service. This step reassured me that the service was running without any issues:  
   ```bash
   sudo systemctl status mongodb
   ```
![003 Mongo Db running successfully](https://github.com/user-attachments/assets/54a98d88-d5e3-4409-b9a2-90cf8772dbb0)

### Why This Step Is Important:
Setting up MongoDB is a critical part of the MEAN stack because it handles the data storage for the application. Ensuring MongoDB is installed correctly and running guarantees that I can move forward with building the backend and testing the connection between the database and my application.

---

6. **Install `npm` - Node Package Manager**

I installed `npm` to manage project dependencies:

```bash
sudo apt install -y npm
```

---

7. **Install `body-parser` Package**

I installed the `body-parser` package, which helps process JSON files passed in requests to the server:

```bash
sudo npm install body-parser
```
Then I ckecke dif body parser existed using 
```bash
ls node_modules | grep body-parser
```
![000body parser](https://github.com/user-attachments/assets/f6f28d62-14c2-4192-9b34-ffa97a54e059)

---

8. **Create a Project Directory**

I created a folder named `Books` and navigated into it:

```bash
mkdir Books && cd Books
```

---

9. **Initialize the Project**

In the `Books` directory, I initialized an `npm` project:

```bash
npm init
```

I followed the prompts to set up the project configuration.

---

10. **Add a Server File**

I created a file named `server.js`:

```bash
vi server.js
```

Next, I pasted the web server code into the `server.js` file.

Here’s the code in markdown format:

```markdown
# Server Code

Below is the code for setting up the server:

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3300;

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/test', {
    useNewUrlParser: true,
    useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParser.json());

require('./apps/routes')(app);

app.listen(PORT, () => {
    console.log(`Server up: http://localhost:${PORT}`);
});
---

![003 server js](https://github.com/user-attachments/assets/37ce1e54-1460-44d2-990c-4ec8869379bf)

---

# Step 3: Implementing Routes in the MEAN Stack Application

1. **Install Required Packages**

I installed the required packages, `express` and `mongoose`:

```bash
sudo npm install express mongoose
```

---

2. **Create the `apps` Folder**

Inside the `Books` folder, I created a folder named `apps` and navigated into it:

```bash
mkdir apps && cd apps
```

---

3. **Create the `routes.js` File**

I created a new file named `routes.js` inside the `apps` folder:

```bash
vi routes.js
```

---

4. **Add the Code to `routes.js`**

I copied and pasted the following code into the `routes.js` file:

```javascript
const Book = require('../models/book');
const path = require('path');

module.exports = function (app) {
    // Get all books
    app.get('/Book', async (req, res) => {
        try {
            const books = await Book.find();
            res.json(books);
        } catch (err) {
            res.status(500).json({ message: 'Error fetching books', error: err.message });
        }
    });

    // Add a new book
    app.post('/Book', async (req, res) => {
        try {
            const book = new Book({
                name: req.body.name,
                isbn: req.body.isbn,
                author: req.body.author,
                pages: req.body.pages,
            });

            const savedBook = await book.save();
            res.status(201).json({
                message: 'Successfully added book',
                book: savedBook,
            });
        } catch (err) {
            res.status(400).json({ message: 'Error adding book', error: err.message });
        }
    });

    // Delete a book by ISBN
    app.delete('/Book/:isbn', async (req, res) => {
        try {
            const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
            if (!result) {
                return res.status(404).json({ message: 'Book not found' });
            }
            res.json({
                message: 'Successfully deleted the book',
                book: result,
            });
        } catch (err) {
            res.status(500).json({ message: 'Error deleting book', error: err.message });
        }
    });

    // Serve the index file
    app.get('*', (req, res) => {
        res.sendFile(path.join(__dirname, '../public', 'index.html'));
    });
};
```

---

This code defines the CRUD operations and a fallback route for serving the front-end `index.html`. 

---

5. **Create a `models` Folder**
   Then, I created a folder named `models` within the `apps` folder. To do this, I used the following command:

```bash
mkdir models && cd models
```

7. **Create a File Named `book.js`**
Inside the `models` folder, I created a file named `book.js`:

```bash
vi book.js
```

8. **Define the `Book` Model**
I then added the following code to `book.js` to define the schema and model for books:

```javascript
const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
    name: { type: String, required: true },
    isbn: { type: String, required: true, unique: true, index: true },
    author: { type: String, required: true },
    pages: { type: Number, required: true, min: 1 }
}, {
    timestamps: true
});

module.exports = mongoose.model('Book', bookSchema);
```
![00 book js](https://github.com/user-attachments/assets/94cfea26-079f-4666-a807-baae19dc50ce)

---

### 4. Setting Up AngularJS and Connecting to Express Routes

In this step, I set up AngularJS to interact with the Express server and perform operations on the book register.

I navigated back to the root directory of the project:

```bash
cd ../..
```

1. **Create a `public` Folder**
Next, I created a folder named `public` to store the AngularJS files:

```bash
mkdir public && cd public
```

2. **Add a `script.js` File**
Inside the `public` folder, I created a file named `script.js`:

```bash
vi script.js
```

3. **Define the AngularJS Controller**
I added the following code to `script.js` to define the AngularJS controller and configure the operations:

```javascript
angular.module('myApp', [])
    .controller('myCtrl', function($scope, $http) {
        function fetchBooks() {
            $http.get('/Books')
                .then(response => {
                    $scope.books = response.data;
                })
                .catch(error => {
                    console.error('Error fetching books:', error);
                });
        }

        fetchBooks();

        $scope.del_book = function(book) {
            $http.delete(`/Books/${book.isbn}`)
                .then(() => {
                    fetchBooks();
                })
                .catch(error => {
                    console.error('Error deleting book:', error);
                });
        };

        $scope.add_book = function() {
            const newBook = {
                name: $scope.Name,
                isbn: $scope.Isbn,
                author: $scope.Author,
                pages: $scope.Pages
            };

            $http.post('/Book', newBook)
                .then(() => {
                    fetchBooks();
                    // Clear form fields
                    $scope.Name = $scope.Isbn = $scope.Author = $scope.Pages = '';
                })
                .catch(error => {
                    console.error('Error adding book:', error);
                });
        };
    });
```

explanation of the code:
1. **Fetch Books**:
   - Sends a `GET` request to the `/Books` endpoint to retrieve the list of books.
   - Stores the response data in `$scope.books`.

2. **Delete a Book**:
   - Sends a `DELETE` request to the `/Books/:isbn` endpoint to delete a book using its ISBN.
   - Refreshes the book list by calling `fetchBooks()`.

3. **Add a Book**:
   - Sends a `POST` request to the `/Book` endpoint with the book details from the form.
   - Clears the form fields and refreshes the book list.

---

---

4. **Creating the Frontend with AngularJS**

In this step, I learned to set up the HTML file for the frontend, which interacts with the AngularJS script to perform CRUD operations.

5. **Create an `index.html` File**
Inside the `public` folder, I created an `index.html` file:

```bash
vi index.html
```

6. **Add the Frontend Code**
I added the following code to `index.html`:

```html
<!DOCTYPE html>
<html ng-app="myApp" ng-controller="myCtrl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Book Management</title>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
    <script src="script.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
        input[type="text"], input[type="number"] { width: 100%; padding: 5px; }
        button { margin-top: 10px; padding: 5px 10px; }
    </style>
</head>
<body>
    <h1>Book Management</h1>

    <h2>Add New Book</h2>
    <form ng-submit="add_book()">
        <table>
            <tr>
                <td>Name:</td>
                <td><input type="text" ng-model="Name" required></td>
            </tr>
            <tr>
                <td>ISBN:</td>
                <td><input type="text" ng-model="Isbn" required></td>
            </tr>
            <tr>
                <td>Author:</td>
                <td><input type="text" ng-model="Author" required></td>
            </tr>
            <tr>
                <td>Pages:</td>
                <td><input type="number" ng-model="Pages" required></td>
            </tr>
        </table>
        <button type="submit">Add Book</button>
    </form>

    <h2>Book List</h2>
    <table>
        <thead>
            <tr>
                <th>Name</th>
                <th>ISBN</th>
                <th>Author</th>
                <th>Pages</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            <tr ng-repeat="book in books">
                <td>{{book.name}}</td>
                <td>{{book.isbn}}</td>
                <td>{{book.author}}</td>
                <td>{{book.pages}}</td>
                <td>
                    <button ng-click="del_book(book)">Delete</button>
                </td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

#### Explanation of the Code
1. **HTML Structure**:
   - The page is structured with two main sections:
     - **Add New Book**: A form to input book details and add a new book to the database.
     - **Book List**: A dynamically generated table that displays the list of books from the database.

2. **AngularJS Integration**:
   - The `ng-app="myApp"` attribute initializes the AngularJS application.
   - The `ng-controller="myCtrl"` attribute links the controller defined in `script.js` to manage the application logic.

3. **Form Binding**:
   - Inputs for `Name`, `ISBN`, `Author`, and `Pages` are bound to `$scope` variables using `ng-model`.
   - The form submits data to the `add_book()` function when the "Add Book" button is clicked.

4. **Dynamic Book List**:
   - The table dynamically populates rows using the `ng-repeat="book in books"` directive.
   - Each row includes a "Delete" button that triggers the `del_book(book)` function.

---
---

5. **Starting the Server**

At this stage, I started the server to make the application accessible. 
I navigated back to the main `Books` directory where the server file (`server.js`) is located.

```bash
cd ..
```

6. **Run the Server**
I started the server by executing the following command:

```bash
node server.js
```
![004 Server running on port 3300](https://github.com/user-attachments/assets/1240674b-33d5-47ce-a945-89cf1503cf3c)


7. **Make the Server Accessible Over the Internet**
To allow external access, I needed to configure the AWS security group to open TCP port **3300** for my EC2 instance.
![000 custom port 3300](https://github.com/user-attachments/assets/5162763e-7fef-4a12-b92a-f7787546c1ca)

and I accessed the server via the web browser using ```http://http://52.89.96.167:3300```

![005 book management app running](https://github.com/user-attachments/assets/1b8f1f0b-e4bb-4657-94a3-d8c7728d7a96)

---
### Lessons Learned, Challenges Faced, and Overcoming Challenges During MEAN Stack Implementation  

Implementing the MEAN stack provided valuable insights into modern web application development. I learned the importance of systematically setting up each component—Node.js, MongoDB, AngularJS, and Express.js—while ensuring seamless integration. Installing Node.js and MongoDB highlighted the need for updated system packages and secure repositories, while AngularJS's model-view-controller framework demonstrated how to bind data dynamically between the front and back ends. Moreover, understanding the structure of NoSQL databases like MongoDB emphasized the flexibility of JSON-like documents in accommodating evolving application requirements.  

However, challenges arose during the process, such as troubleshooting dependency conflicts and resolving errors while connecting MongoDB to the Express server. For instance, an outdated system package led to a version mismatch in MongoDB, causing server connection issues. Similarly, debugging AngularJS routes and ensuring the front end correctly communicated with the backend APIs required careful error tracking. Another obstacle involved correctly setting up the folder structure and ensuring all dependencies were installed without missing any steps.  

To overcome these hurdles, I employed strategies such as updating system packages regularly, referencing official documentation, and utilizing online developer forums for guidance. Using error logs and debugging tools, I pinpointed and resolved specific issues like server misconfigurations and database connectivity problems. Additionally, creating a detailed step-by-step implementation checklist ensured that no critical configuration was overlooked, leading to a functional, full-stack web application. These experiences not only deepened my understanding of the MEAN stack but also sharpened my problem-solving and debugging skills.


