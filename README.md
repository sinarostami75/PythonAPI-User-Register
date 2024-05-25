# PythonAPI-User-Register
An API for creating a user - creating a token for a specified time - creating a user account - with the ability to edit and delete in different parts




The provided code sets up a Flask application with user authentication, registration, and wallet management features, storing data in an SQLite database. Below is a summary and explanation of each part:

### 1- Importing Libraries:

---
```
 import jwt
 import datetime
 import secrets
 from functools import wraps
 from flask import Flask, request, jsonify
 import sqlite3
 import requests
```

### 2- Application Setup:
```
app = Flask(__name__)
SECRET_KEY = secrets.token_hex(16)
```

### 3- Database Connection and Table Creation:
```
get_db_connection(): Establishes a connection to the SQLite database.
create_users_table(), create_tokens_table(), create_passwords_table(), create_wallets_table(): Functions to create necessary tables if they do not exist.
create_database_tables(): Calls the above functions to ensure all tables are created.
```
### 4- Routes:
```
/protected: Protected route example that returns a message for authenticated users.
/login: Authenticates users and generates tokens.
/register: Registers new users.
/add_user: Adds a new user.
/delete_account: Deletes a user account.
/edit_user: Edits user's username and password.
/edit_user_info: Edits user's personal information.
/user_tokens: Returns all user tokens.
/add_wallet: Adds a new wallet for the user.
/delete_wallet: Deletes a user's wallet.
/edit_wallet: Edits an existing wallet.
```
### 5- Helper Functions:
```
authenticate_user(username, password): Verifies the user's password.
get_user_tokens(): Retrieves all user tokens from the database.
```

##### Detailed Code Explanation:
### 1- Generating a Secret Key:
```
SECRET_KEY = secrets.token_hex(16)
```
Generates a random 32-character hexadecimal string used as the secret key for encoding JWT tokens.

### 2- Database Connection:

```
def get_db_connection():
    conn = sqlite3.connect('users.db')
    conn.row_factory = sqlite3.Row
    return conn

```
Establishes a connection to the users.db SQLite database and configures the connection to return rows as dictionaries.

### 3- Table Creation Functions:

Table Creation Functions:
```
def create_users_table():
    with get_db_connection() as conn:
        c = conn.cursor()
        c.execute('''CREATE TABLE IF NOT EXISTS users
                     (username TEXT PRIMARY KEY, first_name TEXT, last_name TEXT, age INTEGER, phone_number TEXT)''')

```
Similar functions are created for tokens, passwords, and wallets tables.

### 4- JWT Token Functions:

```
def generate_token(username):
    payload = {
        'username': username,
        'exp': datetime.datetime.utcnow() + datetime.timedelta(minutes=30)
    }
    token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
    return token

```
Generates a JWT token with an expiration time of 30 minutes.

```
def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get('Authorization')
        if not token:
            return jsonify({'message': 'Token is missing!'}), 401
        token_parts = token.split(' ')
        if len(token_parts) != 2 or token_parts[0].lower() != 'bearer':
            return jsonify({'message': 'Invalid token format!'}), 401
        try:
            data = jwt.decode(token_parts[1], SECRET_KEY, algorithms=['HS256'])
            username = data['username']
        except jwt.ExpiredSignatureError:
            return jsonify({'message': 'Token has expired!'}), 401
        except jwt.InvalidTokenError:
            return jsonify({'message': 'Token is invalid!'}), 401
        return f(username, *args, **kwargs)
    return decorated

```
Decorator function to ensure the request has a valid JWT token.

