# QA Bug Tracker Project

**QA Bug Tracker System**

# Tech Stack

* Python
* Flask
* SQLite
* HTML
* CSS
* Bootstrap

---

# Features

* Add Bugs
* View Bug List
* Update Bug Status
* Delete Bugs
* SQLite Database Integration
* Responsive UI
* Professional Dashboard

---

# Folder Structure

```bash
qa-bug-tracker/
│
├── app.py
├── requirements.txt
├── bugtracker.db
│
├── templates/
│   ├── index.html
│   ├── add_bug.html
│   └── edit_bug.html
│
├── static/
│   └── style.css
│
└── README.md
```

---

# Step 1: Install Dependencies

## requirements.txt

```txt
flask
```

Install:

```bash
pip install -r requirements.txt
```

---

# Step 2: Main Application

## app.py

```python
from flask import Flask, render_template, request, redirect
import sqlite3

app = Flask(__name__)

# Database connection

def connect_db():
    conn = sqlite3.connect('bugtracker.db')
    conn.row_factory = sqlite3.Row
    return conn

# Create table
conn = connect_db()
conn.execute('''
CREATE TABLE IF NOT EXISTS bugs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    description TEXT NOT NULL,
    priority TEXT NOT NULL,
    status TEXT NOT NULL
)
''')
conn.commit()
conn.close()

# Home Page
@app.route('/')
def index():
    conn = connect_db()
    bugs = conn.execute('SELECT * FROM bugs').fetchall()
    conn.close()
    return render_template('index.html', bugs=bugs)

# Add Bug
@app.route('/add', methods=['GET', 'POST'])
def add_bug():
    if request.method == 'POST':
        title = request.form['title']
        description = request.form['description']
        priority = request.form['priority']
        status = request.form['status']

        conn = connect_db()
        conn.execute(
            'INSERT INTO bugs (title, description, priority, status) VALUES (?, ?, ?, ?)',
            (title, description, priority, status)
        )
        conn.commit()
        conn.close()

        return redirect('/')

    return render_template('add_bug.html')

# Edit Bug
@app.route('/edit/<int:id>', methods=['GET', 'POST'])
def edit_bug(id):
    conn = connect_db()
    bug = conn.execute('SELECT * FROM bugs WHERE id = ?', (id,)).fetchone()

    if request.method == 'POST':
        title = request.form['title']
        description = request.form['description']
        priority = request.form['priority']
        status = request.form['status']

        conn.execute(
            'UPDATE bugs SET title=?, description=?, priority=?, status=? WHERE id=?',
            (title, description, priority, status, id)
        )
        conn.commit()
        conn.close()

        return redirect('/')

    conn.close()
    return render_template('edit_bug.html', bug=bug)

# Delete Bug
@app.route('/delete/<int:id>')
def delete_bug(id):
    conn = connect_db()
    conn.execute('DELETE FROM bugs WHERE id=?', (id,))
    conn.commit()
    conn.close()

    return redirect('/')

if __name__ == '__main__':
    app.run(debug=True)
```

---

# Step 3: Frontend Templates

## templates/index.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>QA Bug Tracker</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>

<div class="container mt-5">
    <h1 class="text-center mb-4">QA Bug Tracker Dashboard</h1>

    <a href="/add" class="btn btn-primary mb-3">Add New Bug</a>

    <table class="table table-bordered table-striped">
        <thead>
            <tr>
                <th>ID</th>
                <th>Title</th>
                <th>Description</th>
                <th>Priority</th>
                <th>Status</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            {% for bug in bugs %}
            <tr>
                <td>{{ bug.id }}</td>
                <td>{{ bug.title }}</td>
                <td>{{ bug.description }}</td>
                <td>{{ bug.priority }}</td>
                <td>{{ bug.status }}</td>
                <td>
                    <a href="/edit/{{ bug.id }}" class="btn btn-warning btn-sm">Edit</a>
                    <a href="/delete/{{ bug.id }}" class="btn btn-danger btn-sm">Delete</a>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</div>

</body>
</html>
```

---

## templates/add_bug.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>Add Bug</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body>

<div class="container mt-5">
    <h2>Add New Bug</h2>

    <form method="POST">
        <div class="mb-3">
            <label>Bug Title</label>
            <input type="text" name="title" class="form-control" required>
        </div>

        <div class="mb-3">
            <label>Description</label>
            <textarea name="description" class="form-control" required></textarea>
        </div>

        <div class="mb-3">
            <label>Priority</label>
            <select name="priority" class="form-control">
                <option>Low</option>
                <option>Medium</option>
                <option>High</option>
            </select>
        </div>

        <div class="mb-3">
            <label>Status</label>
            <select name="status" class="form-control">
                <option>Open</option>
                <option>In Progress</option>
                <option>Closed</option>
            </select>
        </div>

        <button type="submit" class="btn btn-success">Save Bug</button>
    </form>
</div>

</body>
</html>
```

---

## templates/edit_bug.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>Edit Bug</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body>

<div class="container mt-5">
    <h2>Edit Bug</h2>

    <form method="POST">
        <div class="mb-3">
            <label>Bug Title</label>
            <input type="text" name="title" value="{{ bug.title }}" class="form-control" required>
        </div>

        <div class="mb-3">
            <label>Description</label>
            <textarea name="description" class="form-control" required>{{ bug.description }}</textarea>
        </div>

        <div class="mb-3">
            <label>Priority</label>
            <select name="priority" class="form-control">
                <option {% if bug.priority == 'Low' %}selected{% endif %}>Low</option>
                <option {% if bug.priority == 'Medium' %}selected{% endif %}>Medium</option>
                <option {% if bug.priority == 'High' %}selected{% endif %}>High</option>
            </select>
        </div>

        <div class="mb-3">
            <label>Status</label>
            <select name="status" class="form-control">
                <option {% if bug.status == 'Open' %}selected{% endif %}>Open</option>
                <option {% if bug.status == 'In Progress' %}selected{% endif %}>In Progress</option>
                <option {% if bug.status == 'Closed' %}selected{% endif %}>Closed</option>
            </select>
        </div>

        <button type="submit" class="btn btn-success">Update Bug</button>
    </form>
</div>

</body>
</html>
```

---

# Step 4: CSS Styling

## static/style.css

```css
body {
    background-color: #f4f6f9;
}

h1 {
    font-weight: bold;
    color: #333;
}

.table {
    background: white;
}
```

---

# Step 5: README.md

````md
# QA Bug Tracker System

A beginner-friendly Flask-based bug tracking system.

## Features
- Add bugs
- Edit bugs
- Delete bugs
- Manage priorities
- Update bug status
- SQLite database

## Tech Stack
- Python
- Flask
- SQLite
- Bootstrap

## Installation

```bash
pip install -r requirements.txt
python app.py
````

## Author

Mohamed Sulaiman Athif

````

---

# Step 6: Run the Project

```bash
python app.py
````

Open browser:

```bash
http://127.0.0.1:5000
```

---

a-bug-tracker-system
```

---

## Git Commands

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin YOUR_GITHUB_REPOSITORY_LINK
git push -u origin main
```

---

