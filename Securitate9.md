To create a site with Python for the backend and HTML, CSS, and Bootstrap for the frontend, including features like a shopping cart, product archive, customer logins, and a home page, follow these detailed steps:

### Step 1: Set Up the Project Structure

1. **Create a Project Directory:**
   ```bash
   mkdir my_ecommerce_site
   cd my_ecommerce_site
   ```

2. **Set Up a Virtual Environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows: venv\Scripts\activate
   ```

3. **Install Necessary Packages:**
   ```bash
   pip install Flask SQLAlchemy Flask-Login Flask-WTF
   ```

### Step 2: Initialize a Flask Application

1. **Create the Flask App:**
   - Inside your project directory, create the `app` directory:
     ```bash
     mkdir app
     cd app
     touch __init__.py
     ```

2. **Set Up the Flask App in `__init__.py`:**
   ```python
   from flask import Flask
   from flask_sqlalchemy import SQLAlchemy
   from flask_login import LoginManager

   app = Flask(__name__)
   app.config['SECRET_KEY'] = 'your_secret_key'
   app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///site.db'
   db = SQLAlchemy(app)
   login_manager = LoginManager(app)
   login_manager.login_view = 'login'

   from app import routes
   ```

3. **Create the Database Models in `models.py`:**
   ```python
   from datetime import datetime
   from app import db, login_manager
   from flask_login import UserMixin

   @login_manager.user_loader
   def load_user(user_id):
       return User.query.get(int(user_id))

   class User(db.Model, UserMixin):
       id = db.Column(db.Integer, primary_key=True)
       username = db.Column(db.String(20), unique=True, nullable=False)
       email = db.Column(db.String(120), unique=True, nullable=False)
       image_file = db.Column(db.String(20), nullable=False, default='default.jpg')
       password = db.Column(db.String(60), nullable=False)
       orders = db.relationship('Order', backref='customer', lazy=True)

   class Product(db.Model):
       id = db.Column(db.Integer, primary_key=True)
       name = db.Column(db.String(100), nullable=False)
       description = db.Column(db.Text, nullable=False)
       price = db.Column(db.Float, nullable=False)
       image_file = db.Column(db.String(20), nullable=False, default='default.jpg')

   class Order(db.Model):
       id = db.Column(db.Integer, primary_key=True)
       date_ordered = db.Column(db.DateTime, nullable=False, default=datetime.utcnow)
       user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
       products = db.relationship('OrderItem', backref='order', lazy=True)

   class OrderItem(db.Model):
       id = db.Column(db.Integer, primary_key=True)
       order_id = db.Column(db.Integer, db.ForeignKey('order.id'), nullable=False)
       product_id = db.Column(db.Integer, db.ForeignKey('product.id'), nullable=False)
       quantity = db.Column(db.Integer, nullable=False)
   ```

4. **Create Routes in `routes.py`:**
   ```python
   from flask import render_template, url_for, flash, redirect, request
   from app import app, db
   from app.models import User, Product, Order, OrderItem
   from flask_login import login_user, current_user, logout_user, login_required

   @app.route("/")
   @app.route("/home")
   def home():
       return render_template('home.html')

   @app.route("/login", methods=['GET', 'POST'])
   def login():
       # Handle login logic
       pass

   @app.route("/logout")
   def logout():
       # Handle logout logic
       pass

   @app.route("/register", methods=['GET', 'POST'])
   def register():
       # Handle registration logic
       pass

   @app.route("/product/<int:product_id>")
   def product(product_id):
       # Display product details
       pass

   @app.route("/cart")
   @login_required
   def cart():
       # Display shopping cart
       pass

   @app.route("/checkout", methods=['GET', 'POST'])
   @login_required
   def checkout():
       # Handle checkout logic
       pass
   ```

### Step 3: Create HTML Templates

1. **Set Up the `templates` Directory:**
   - Create a `templates` directory inside the `app` directory.
   - Create a `layout.html` file for the common layout:
     ```html
     <!DOCTYPE html>
     <html lang="en">
     <head>
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1.0">
         <title>{{ title }}</title>
         <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
     </head>
     <body>
         <nav class="navbar navbar-expand-lg navbar-light bg-light">
             <a class="navbar-brand" href="{{ url_for('home') }}">MyShop</a>
             <div class="collapse navbar-collapse" id="navbarNav">
                 <ul class="navbar-nav ml-auto">
                     {% if current_user.is_authenticated %}
                         <li class="nav-item">
                             <a class="nav-link" href="{{ url_for('logout') }}">Logout</a>
                         </li>
                         <li class="nav-item">
                             <a class="nav-link" href="{{ url_for('cart') }}">Cart</a>
                         </li>
                     {% else %}
                         <li class="nav-item">
                             <a class="nav-link" href="{{ url_for('login') }}">Login</a>
                         </li>
                         <li class="nav-item">
                             <a class="nav-link" href="{{ url_for('register') }}">Register</a>
                         </li>
                     {% endif %}
                 </ul>
             </div>
         </nav>
         <div class="container">
             {% block content %}{% endblock %}
         </div>
     </body>
     </html>
     ```

2. **Create `home.html` for the Home Page:**
   ```html
   {% extends "layout.html" %}
   {% block content %}
   <div class="jumbotron">
       <h1>Welcome to MyShop</h1>
       <p>Your one-stop shop for all your needs!</p>
   </div>
   <div class="row">
       <!-- Display some products here -->
   </div>
   {% endblock %}
   ```

3. **Create Additional Templates:**
   - `login.html`, `register.html`, `product.html`, `cart.html`, `checkout.html` for their respective routes.
   - Each should extend `layout.html` and fill in the `{% block content %}` section.

### Step 4: Implement Authentication

1. **Create Forms in `forms.py`:**
   ```python
   from flask_wtf import FlaskForm
   from wtforms import StringField, PasswordField, SubmitField, BooleanField
   from wtforms.validators import DataRequired, Length, Email, EqualTo

   class RegistrationForm(FlaskForm):
       username = StringField('Username', validators=[DataRequired(), Length(min=2, max=20)])
       email = StringField('Email', validators=[DataRequired(), Email()])
       password = PasswordField('Password', validators=[DataRequired()])
       confirm_password = PasswordField('Confirm Password', validators=[DataRequired(), EqualTo('password')])
       submit = SubmitField('Sign Up')

   class LoginForm(FlaskForm):
       email = StringField('Email', validators=[DataRequired(), Email()])
       password = PasswordField('Password', validators=[DataRequired()])
       remember = BooleanField('Remember Me')
       submit = SubmitField('Login')
   ```

2. **Update Routes for Authentication:**
   - Implement user registration, login, and logout logic in `routes.py`.

### Step 5: Handle Products and Cart

1. **Create Product Routes:**
   - Create a route to display product details.
   - Create a route to add products to the cart.

2. **Create Cart Logic:**
   - Use sessions to manage cart items for each user.
   - Create a route to display the cart and a route to handle checkout.

### Step 6: Deploy the Application

1. **Deploy Backend on a Service like Heroku:**
   - Create a `Procfile`:
     ```text
     web: gunicorn app:app
     ```

   - Add `gunicorn` to your `requirements.txt`:
     ```bash
     pip freeze > requirements.txt
     ```

   - Deploy to Heroku:
     ```bash
     heroku create
     git add .
     git commit -m "Initial commit"
     git push heroku master
     ```

2. **Host Frontend Using a Static Site Host:**
   - Extract your static files (HTML, CSS, JS) and deploy them to a static site host like Netlify or Vercel.

3. **Connect Frontend and Backend:**
   - Ensure your frontend calls the backend API for dynamic content and user interactions.

By following these steps, you can build and deploy a fully functional e-commerce site with separate frontend and backend.