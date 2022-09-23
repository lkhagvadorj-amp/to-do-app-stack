# Introduction
In this tutorial, you will build a To-Do application using Django and React.

React is a JavaScript framework for developing SPAs (single-page applications). It has [solid documentation](https://reactjs.org/docs/getting-started.html) and a vibrant ecosystem around it.

Django is a Python web framework that simplifies common practices in web development. Django is reliable and also has a vibrant ecosystem of stable libraries supporting common development needs.

For this application, React serves as the frontend, or client-side framework, handling the user interface and getting and setting data via requests to the Django backend, which is an API built using the Django REST framework (DRF).

This application will allow users to create tasks and mark them as complete or incomplete.

# Prerequisites

To follow along with this tutorial, you will need to:

- [Install and set up a local programming environment for Python 3](https://www.digitalocean.com/community/tutorial_series/how-to-install-and-set-up-a-local-programming-environment-for-python-3)
- [Install Node.js and Create a Local Development Environment](https://www.digitalocean.com/community/tutorial_series/how-to-install-node-js-and-create-a-local-development-environment)

This tutorial was verified with Python v3.10.6, pip v22.2.2, Django v3.1.7, djangorestframework v3.14.0, django-cors-headers v3.13.0, Node v18.9.0, npm v8.19.1, React v18.2.0, and axios v0.27.2.

# Step 1 - Setting Up the Backend
In this section, you will create a new project directory and install Django.

Open a new terminal window and run the following command to create a new project directory:

```bash
mkdir django-todo-react
```

Next, navigate into the directory:

```bash
cd django-todo-react
```

Now install Pipenv using pip:

```bash
pip install pipenv
```

Note: Depending on your installation, you may need to use pip3 instead of pip.

And activate a new virtual environment:

```bash
pipenv shell
```

Install Django using Pipenv:

```bash
pipenv install django
```

Then create a new project called backend:

```bash
django-admin startproject backend
```

Next, navigate into the newly created backend directory:

```bash
cd backend
```

Start a new application called todo:

```bash
python manage.py startapp todo
```

Run migrations:

```bash
python manage.py migrate
```

And start up the server:

```bash
python manage.py runserver
```

Navigate to [http://localhost:8000](http://localhost:8000) in your web browser.

At this point, you will see an instance of a Django application running successfully. Once you are finished, you can stop the server (`CONTROL+C` or `CTRL+C`).

## Registering the todo Application

Now that you have completed the setup for the backend, you can begin registering the `todo` application as an installed app so that Django can recognize it.

Open the `backend/settings.py` file in your code editor and add `todo` to the `INSTALLED_APPS`:

```python
# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'todo',
]
```

Then, save your changes.

## Defining the `Todo` Model
Let’s create a model to define how the `Todo` items should be stored in the database.

Open the `todo/models.py` file in your code editor and add the following lines of code:

```python
from django.db import models

# Create your models here.

class Todo(models.Model):
    title = models.CharField(max_length=120)
    description = models.TextField()
    completed = models.BooleanField(default=False)

    def _str_(self):
        return self.title
```

The code snippet above describes three properties on the Todo model:

 - `title`
 - `description`
 - `completed`

The `completed` property is the status of a task. A task will either be completed or not completed at any time. Because you have created a `Todo` model, you will need to create a migration file:

```bash
python manage.py makemigrations todo
```

And apply the changes to the database:

```bash
python manage.py migrate todo
```

You can test to see that CRUD operations work on the `Todo` model you created by using the admin interface that Django provides by default.

Open the `todo/admin.py` file with your code editor and add the following lines of code:

```python
from django.contrib import admin
from .models import Todo

class TodoAdmin(admin.ModelAdmin):
    list_display = ('title', 'description', 'completed')

# Register your models here.

admin.site.register(Todo, TodoAdmin)
```

Then, save your changes.

You will need to create a “superuser” account to access the admin interface. Run the following command in your terminal:

```bash
python manage.py createsuperuser
```

You will be prompted to enter a username, email, and password for the superuser. Be sure to enter details that you can remember because you will need them to log in to the admin dashboard.

Start the server once again:

```bash
python manage.py runserver
```

Navigate to [http://localhost:8000](http://localhost:8000) in your web browser. And log in with the username and password that was created earlier.

You can create, edit, and, delete `Todo` items using this interface.

After experimenting with this interface, you can stop the server (`CONTROL+C` or `CTRL+C`).

# Step 2 — Setting Up the APIs

In this section, you will create an API using the Django REST framework.

Install the `djangorestframework` and `django-cors-headers` using Pipenv:

```bash
pipenv install djangorestframework django-cors-headers
```

You need to add `rest_framework` and `corsheaders` to the list of installed applications. Open the `backend/settings.py` file in your code editor and update the `INSTALLED_APPS` and `MIDDLEWARE` sections:

```python
# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'corsheaders',
    'rest_framework',
    'todo',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'corsheaders.middleware.CorsMiddleware',
]
```

Then, add these lines of code to the bottom of the `backend/settings.py` file:

```python
CORS_ORIGIN_WHITELIST = [
     'http://localhost:3000'
]
```

`django-cors-headers` is a Python library that will prevent the errors that you would normally get due to CORS rules. In the `CORS_ORIGIN_WHITELIST` code, you whitelisted `localhost:3000` because you want the frontend (which will be served on that port) of the application to interact with the API.

## Creating `serializers`

You will need serializers to convert model instances to JSON so that the frontend can work with the received data.

Create a `todo/serializers.py` file with your code editor. Open the `serializers.py` file and update it with the following lines of code:

```python
from rest_framework import serializers
from .models import Todo

class TodoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Todo
        fields = ('id', 'title', 'description', 'completed')
```

This code specifies the model to work with and the fields to be converted to JSON.

## Creating the View
You will need to create a `TodoView` class in the `todo/views.py` file.

Open the `todo/views.py` file with your code editor and add the following lines of code:

```python
from django.shortcuts import render
from rest_framework import viewsets
from .serializers import TodoSerializer
from .models import Todo

# Create your views here.

class TodoView(viewsets.ModelViewSet):
    serializer_class = TodoSerializer
    queryset = Todo.objects.all()
```

The `viewsets` base class provides the implementation for CRUD operations by default. This code specifies the `serializer_class` and the `queryset`.

Open the `backend/urls.py` file with your code editor and replace the contents with the following lines of code:

```python
from django.contrib import admin
from django.urls import path, include
from rest_framework import routers
from todo import views

router = routers.DefaultRouter()
router.register(r'todos', views.TodoView, 'todo')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),
]
```

This code specifies the URL path for the API. This was the final step that completes the building of the API.

You can now perform CRUD operations on the `Todo` model. The router class allows you to make the following queries:

 - `/todos/` - returns a list of all the `Todo` items. `CREATE` and `READ` operations can be performed here.
 - `/todos/id` - returns a single Todo item using the id primary key. `UPDATE` and `DELETE` operations can be performed here.
Let’s restart the server:

```bash
python manage.py runserver
```

Navigate to  [http://localhost:8000/api/todos](http://localhost:8000/api/todos) in your web browser.

You can `CREATE` a new Todo item using the interface:
If the `Todo` item is created successfully, you will be presented with a successful response:

You can also perform `DELETE` and `UPDATE` operations on specific `Todo` items using the `id` primary keys. Use the address structure `/api/todos/{id}` and provide an `id`.

Add `1` to the URL to examine the `Todo` item with the id of “1”. Navigate to [http://localhost:8000/api/todos/1](http://localhost:8000/api/todos/1) in your web browser.

This completes the building of the backend of the application.

# Step 3 — Setting Up the Frontend
Now that you have the backend of the application complete, you can create the frontend and have it communicate with the backend over the interface that you created.

First, open a new terminal window and navigate to the `django-todo-react` project directory.

To set up the frontend, this tutorial will rely upon Create React App. There are several approaches to using `create-react-app`. One approach is to use `npx` to run the package and create the project:

```bash
npx create-react-app frontend
```

You can learn more about this approach by reading the [How To Set Up a React Project with Create React App](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-react-project-with-create-react-app).

After the project is created, you can change into the newly created frontend directory:

```bash
cd frontend
```

Then, start the application:

```bash
npm start
```

Your web browser will open [http://localhost:3000](http://localhost:3000) and you will be presented with the default Create React App screen.

Next, install bootstrap and reactstrap to provide user interface tools.

```bash
npm install bootstrap@latest reactstrap@latest --legacy-peer-deps
```

Open `index.js` in your code editor and add `bootstrap.min.css`:

```js
import React from 'react';
import ReactDOM from 'react-dom';
import 'bootstrap/dist/css/bootstrap.css';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

If you are having difficulty with this step, you can [consult the official documentation for adding `bootstrap`](https://www.digitalocean.com/community/tutorials/build-a-to-do-application-using-django-and-react#:~:text=consult%20the%20official%20documentation%20for%20adding%20bootstrap).

Open `App.js` in your code editor and add the following lines of code:

```js
import React, { Component } from "react";

const todoItems = [
  {
    id: 1,
    title: "Go to Market",
    description: "Buy ingredients to prepare dinner",
    completed: true,
  },
  {
    id: 2,
    title: "Study",
    description: "Read Algebra and History textbook for the upcoming test",
    completed: false,
  },
  {
    id: 3,
    title: "Sammy's books",
    description: "Go to library to return Sammy's books",
    completed: true,
  },
  {
    id: 4,
    title: "Article",
    description: "Write article on how to use Django with React",
    completed: false,
  },
];

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      viewCompleted: false,
      todoList: todoItems,
    };
  }

  displayCompleted = (status) => {
    if (status) {
      return this.setState({ viewCompleted: true });
    }

    return this.setState({ viewCompleted: false });
  };

  renderTabList = () => {
    return (
      <div className="nav nav-tabs">
        <span
          className={this.state.viewCompleted ? "nav-link active" : "nav-link"}
          onClick={() => this.displayCompleted(true)}
        >
          Complete
        </span>
        <span
          className={this.state.viewCompleted ? "nav-link" : "nav-link active"}
          onClick={() => this.displayCompleted(false)}
        >
          Incomplete
        </span>
      </div>
    );
  };

  renderItems = () => {
    const { viewCompleted } = this.state;
    const newItems = this.state.todoList.filter(
      (item) => item.completed == viewCompleted
    );

    return newItems.map((item) => (
      <li
        key={item.id}
        className="list-group-item d-flex justify-content-between align-items-center"
      >
        <span
          className={`todo-title mr-2 ${
            this.state.viewCompleted ? "completed-todo" : ""
          }`}
          title={item.description}
        >
          {item.title}
        </span>
        <span>
          <button
            className="btn btn-secondary mr-2"
          >
            Edit
          </button>
          <button
            className="btn btn-danger"
          >
            Delete
          </button>
        </span>
      </li>
    ));
  };

  render() {
    return (
      <main className="container">
        <h1 className="text-white text-uppercase text-center my-4">Todo app</h1>
        <div className="row">
          <div className="col-md-6 col-sm-10 mx-auto p-0">
            <div className="card p-3">
              <div className="mb-4">
                <button
                  className="btn btn-primary"
                >
                  Add task
                </button>
              </div>
              {this.renderTabList()}
              <ul className="list-group list-group-flush border-top-0">
                {this.renderItems()}
              </ul>
            </div>
          </div>
        </div>
      </main>
    );
  }
}

export default App;
```

This code includes some hardcoded values for four items. These will be temporary values until items are fetched from the backend.

The `renderTabList()` function renders two spans that help control which set of items are displayed. Clicking on the Completed tab will display the completed tasks. Clicking on the Incomplete tab will display the incomplete tasks.

Save your changes and observe the application in your web browser.

To handle actions such as adding and editing tasks, you will need to create a modal component.

First, create a `components` folder in the `src` directory:

```bash
mkdir src/components
```

Then, create a `Modal.js` file and open it with your code editor. Add the following lines of code:

```js
import React, { Component } from "react";
import {
  Button,
  Modal,
  ModalHeader,
  ModalBody,
  ModalFooter,
  Form,
  FormGroup,
  Input,
  Label,
} from "reactstrap";

export default class CustomModal extends Component {
  constructor(props) {
    super(props);
    this.state = {
      activeItem: this.props.activeItem,
    };
  }

  handleChange = (e) => {
    let { name, value } = e.target;

    if (e.target.type === "checkbox") {
      value = e.target.checked;
    }

    const activeItem = { ...this.state.activeItem, [name]: value };

    this.setState({ activeItem });
  };

  render() {
    const { toggle, onSave } = this.props;

    return (
      <Modal isOpen={true} toggle={toggle}>
        <ModalHeader toggle={toggle}>Todo Item</ModalHeader>
        <ModalBody>
          <Form>
            <FormGroup>
              <Label for="todo-title">Title</Label>
              <Input
                type="text"
                id="todo-title"
                name="title"
                value={this.state.activeItem.title}
                onChange={this.handleChange}
                placeholder="Enter Todo Title"
              />
            </FormGroup>
            <FormGroup>
              <Label for="todo-description">Description</Label>
              <Input
                type="text"
                id="todo-description"
                name="description"
                value={this.state.activeItem.description}
                onChange={this.handleChange}
                placeholder="Enter Todo description"
              />
            </FormGroup>
            <FormGroup check>
              <Label check>
                <Input
                  type="checkbox"
                  name="completed"
                  checked={this.state.activeItem.completed}
                  onChange={this.handleChange}
                />
                Completed
              </Label>
            </FormGroup>
          </Form>
        </ModalBody>
        <ModalFooter>
          <Button
            color="success"
            onClick={() => onSave(this.state.activeItem)}
          >
            Save
          </Button>
        </ModalFooter>
      </Modal>
    );
  }
}
```

This code creates a `CustomModal` class and it nests the `Modal` component that is derived from the `reactstrap` library.

This code also defined three fields in the form:

 - `title`
 - `description`
 - `completed`

These are the same fields that we defined as properties on the Todo model in the backend.

The CustomModal receives activeItem, toggle, and onSave as props:

 - `activeItem` represents the Todo item to be edited.
 - `toggle` is a function used to control the Modal’s state (i.e., open or close the modal).
 - `onSave` is a function that is called to save the edited values of the Todo item.

Next, you will import the `CustomModal` component into the `App.js` file.
Revisit the `src/App.js` file with your code editor and replace the entire contents with the following lines of code:

```js
import React, { Component } from "react";
import Modal from "./components/Modal";

const todoItems = [
  {
    id: 1,
    title: "Go to Market",
    description: "Buy ingredients to prepare dinner",
    completed: true,
  },
  {
    id: 2,
    title: "Study",
    description: "Read Algebra and History textbook for the upcoming test",
    completed: false,
  },
  {
    id: 3,
    title: "Sammy's books",
    description: "Go to library to return Sammy's books",
    completed: true,
  },
  {
    id: 4,
    title: "Article",
    description: "Write article on how to use Django with React",
    completed: false,
  },
];

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      viewCompleted: false,
      todoList: todoItems,
      modal: false,
      activeItem: {
        title: "",
        description: "",
        completed: false,
      },
    };
  }

  toggle = () => {
    this.setState({ modal: !this.state.modal });
  };

  handleSubmit = (item) => {
    this.toggle();

    alert("save" + JSON.stringify(item));
  };

  handleDelete = (item) => {
    alert("delete" + JSON.stringify(item));
  };

  createItem = () => {
    const item = { title: "", description: "", completed: false };

    this.setState({ activeItem: item, modal: !this.state.modal });
  };

  editItem = (item) => {
    this.setState({ activeItem: item, modal: !this.state.modal });
  };

  displayCompleted = (status) => {
    if (status) {
      return this.setState({ viewCompleted: true });
    }

    return this.setState({ viewCompleted: false });
  };

  renderTabList = () => {
    return (
      <div className="nav nav-tabs">
        <span
          className={this.state.viewCompleted ? "nav-link active" : "nav-link"}
          onClick={() => this.displayCompleted(true)}
        >
          Complete
        </span>
        <span
          className={this.state.viewCompleted ? "nav-link" : "nav-link active"}
          onClick={() => this.displayCompleted(false)}
        >
          Incomplete
        </span>
      </div>
    );
  };

  renderItems = () => {
    const { viewCompleted } = this.state;
    const newItems = this.state.todoList.filter(
      (item) => item.completed === viewCompleted
    );

    return newItems.map((item) => (
      <li
        key={item.id}
        className="list-group-item d-flex justify-content-between align-items-center"
      >
        <span
          className={`todo-title mr-2 ${
            this.state.viewCompleted ? "completed-todo" : ""
          }`}
          title={item.description}
        >
          {item.title}
        </span>
        <span>
          <button
            className="btn btn-secondary mr-2"
            onClick={() => this.editItem(item)}
          >
            Edit
          </button>
          <button
            className="btn btn-danger"
            onClick={() => this.handleDelete(item)}
          >
            Delete
          </button>
        </span>
      </li>
    ));
  };

  render() {
    return (
      <main className="container">
        <h1 className="text-white text-uppercase text-center my-4">Todo app</h1>
        <div className="row">
          <div className="col-md-6 col-sm-10 mx-auto p-0">
            <div className="card p-3">
              <div className="mb-4">
                <button
                  className="btn btn-primary"
                  onClick={this.createItem}
                >
                  Add task
                </button>
              </div>
              {this.renderTabList()}
              <ul className="list-group list-group-flush border-top-0">
                {this.renderItems()}
              </ul>
            </div>
          </div>
        </div>
        {this.state.modal ? (
          <Modal
            activeItem={this.state.activeItem}
            toggle={this.toggle}
            onSave={this.handleSubmit}
          />
        ) : null}
      </main>
    );
  }
}

export default App;
```

Save your changes and observe the application in your web browser.

If you attempt to edit and save a `Todo` item, you will get an alert displaying the `Todo` item’s object. Clicking on `Save` or `Delete` will perform the respective actions on the `Todo` item.

Now, you will modify the application so that it interacts with the Django API you built in the previous section. Revisit the first terminal window and ensure the server is running. If it is not running, use the following command:

```bash
python manage.py runserver
```

To make requests to the API endpoints on the backend server, you will install a JavaScript library called `axios`.

In the second terminal window, ensure that you are in the `frontend` directory and install `axios`:

```bash
npm install axios
```

Then open the `frontend/package.json` file in your code editor and add a `proxy`:

```json
[...]
  "name": "frontend",
  "version": "0.1.0",
  "private": true,
  "proxy": "http://localhost:8000",
  "dependencies": {
    "axios": "^0.18.0",
    "bootstrap": "^4.1.3",
    "react": "^16.5.2",
    "react-dom": "^16.5.2",
    "react-scripts": "2.0.5",
    "reactstrap": "^6.5.0"
  },
[...]
```

The proxy will help in tunneling API requests to [http://localhost:8000](http://localhost:8000) where the Django application will handle them. Without this proxy, you would need to specify full paths:

```bash
axios.get("http://localhost:8000/api/todos/")
```

With proxy, you can provide relative paths:

```bash
axios.get("/api/todos/")
```

Revisit the `frontend/src/App.js` file and open it with your code editor. In this step, you will remove the hardcoded `todoItems` and use data from requests to the backend server. `handleSubmit` and `handleDelete`

Open the `App.js` file and replace it with this final version:

```js
import React, { Component } from "react";
import Modal from "./components/Modal";
import axios from "axios";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      viewCompleted: false,
      todoList: [],
      modal: false,
      activeItem: {
        title: "",
        description: "",
        completed: false,
      },
    };
  }

  componentDidMount() {
    this.refreshList();
  }

  refreshList = () => {
    axios
      .get("/api/todos/")
      .then((res) => this.setState({ todoList: res.data }))
      .catch((err) => console.log(err));
  };

  toggle = () => {
    this.setState({ modal: !this.state.modal });
  };

  handleSubmit = (item) => {
    this.toggle();

    if (item.id) {
      axios
        .put(`/api/todos/${item.id}/`, item)
        .then((res) => this.refreshList());
      return;
    }
    axios
      .post("/api/todos/", item)
      .then((res) => this.refreshList());
  };

  handleDelete = (item) => {
    axios
      .delete(`/api/todos/${item.id}/`)
      .then((res) => this.refreshList());
  };

  createItem = () => {
    const item = { title: "", description: "", completed: false };

    this.setState({ activeItem: item, modal: !this.state.modal });
  };

  editItem = (item) => {
    this.setState({ activeItem: item, modal: !this.state.modal });
  };

  displayCompleted = (status) => {
    if (status) {
      return this.setState({ viewCompleted: true });
    }

    return this.setState({ viewCompleted: false });
  };

  renderTabList = () => {
    return (
      <div className="nav nav-tabs">
        <span
          onClick={() => this.displayCompleted(true)}
          className={this.state.viewCompleted ? "nav-link active" : "nav-link"}
        >
          Complete
        </span>
        <span
          onClick={() => this.displayCompleted(false)}
          className={this.state.viewCompleted ? "nav-link" : "nav-link active"}
        >
          Incomplete
        </span>
      </div>
    );
  };

  renderItems = () => {
    const { viewCompleted } = this.state;
    const newItems = this.state.todoList.filter(
      (item) => item.completed === viewCompleted
    );

    return newItems.map((item) => (
      <li
        key={item.id}
        className="list-group-item d-flex justify-content-between align-items-center"
      >
        <span
          className={`todo-title mr-2 ${
            this.state.viewCompleted ? "completed-todo" : ""
          }`}
          title={item.description}
        >
          {item.title}
        </span>
        <span>
          <button
            className="btn btn-secondary mr-2"
            onClick={() => this.editItem(item)}
          >
            Edit
          </button>
          <button
            className="btn btn-danger"
            onClick={() => this.handleDelete(item)}
          >
            Delete
          </button>
        </span>
      </li>
    ));
  };

  render() {
    return (
      <main className="container">
        <h1 className="text-white text-uppercase text-center my-4">Todo app</h1>
        <div className="row">
          <div className="col-md-6 col-sm-10 mx-auto p-0">
            <div className="card p-3">
              <div className="mb-4">
                <button
                  className="btn btn-primary"
                  onClick={this.createItem}
                >
                  Add task
                </button>
              </div>
              {this.renderTabList()}
              <ul className="list-group list-group-flush border-top-0">
                {this.renderItems()}
              </ul>
            </div>
          </div>
        </div>
        {this.state.modal ? (
          <Modal
            activeItem={this.state.activeItem}
            toggle={this.toggle}
            onSave={this.handleSubmit}
          />
        ) : null}
      </main>
    );
  }
}

export default App;
```

The `refreshList()` function is reusable that is called each time an API request is completed. It updates the Todo list to display the most recent list of added items.

The `handleSubmit()` function takes care of both the create and update operations. If the item passed as the parameter doesn’t have an `id`, then it has probably not been created, so the function creates it.

At this point, verify that your backend server is running in your first terminal window:

```bash
python manage.py runserver
```

And in your second terminal window, ensure that you are in the frontend directory and start your frontend application:

```bash
npm start
```

Now when you visit [http://localhost:3000](http://localhost:3000) with your web browser, your application will allow you to `READ`, `CREATE`, `UPDATE`, and `DELETE` tasks.

This completes the frontend and backend of the Todo application.

# Conclusion
In this article, you built a To-Do application using Django and React. You achieved this with the `djangorestframework`, `django-cors-headers`, `axios`, `bootstrap`, and `reactstrap` libraries.
