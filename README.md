Creating a website to store documents involves several components, including a front-end interface for uploading and viewing documents, a back-end server to handle file storage and retrieval, and a database to store metadata about the documents. Below is a basic example using **Python** with **Flask** for the back-end, **HTML/CSS** for the front-end, and **SQLite** for the database.

---

### **1. Back-End (Flask)**
Install Flask and required dependencies:
```bash
pip install flask
```

Create a file named `app.py`:
```python
from flask import Flask, render_template, request, redirect, url_for, send_from_directory
import os
from werkzeug.utils import secure_filename
import sqlite3

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = 'uploads'
app.config['ALLOWED_EXTENSIONS'] = {'txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif', 'doc', 'docx'}

# Ensure the upload folder exists
if not os.path.exists(app.config['UPLOAD_FOLDER']):
    os.makedirs(app.config['UPLOAD_FOLDER'])

# Database setup
def init_db():
    conn = sqlite3.connect('documents.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS documents
                 (id INTEGER PRIMARY KEY AUTOINCREMENT, filename TEXT, filepath TEXT)''')
    conn.commit()
    conn.close()

init_db()

# Helper function to check allowed file extensions
def allowed_file(filename):
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in app.config['ALLOWED_EXTENSIONS']

# Home page
@app.route('/')
def index():
    conn = sqlite3.connect('documents.db')
    c = conn.cursor()
    c.execute('SELECT * FROM documents')
    documents = c.fetchall()
    conn.close()
    return render_template('index.html', documents=documents)

# Upload file
@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return redirect(request.url)
    file = request.files['file']
    if file.filename == '':
        return redirect(request.url)
    if file and allowed_file(file.filename):
        filename = secure_filename(file.filename)
        filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
        file.save(filepath)

        # Save file metadata to database
        conn = sqlite3.connect('documents.db')
        c = conn.cursor()
        c.execute('INSERT INTO documents (filename, filepath) VALUES (?, ?)', (filename, filepath))
        conn.commit()
        conn.close()

        return redirect(url_for('index'))
    return redirect(request.url)

# Download file
@app.route('/download/<int:doc_id>')
def download_file(doc_id):
    conn = sqlite3.connect('documents.db')
    c = conn.cursor()
    c.execute('SELECT filepath FROM documents WHERE id = ?', (doc_id,))
    filepath = c.fetchone()[0]
    conn.close()
    return send_from_directory(os.path.dirname(filepath), os.path.basename(filepath), as_attachment=True)

if __name__ == '__main__':
    app.run(debug=True)
```

---

### **2. Front-End (HTML/CSS)**
Create a folder named `templates` and inside it, create a file named `index.html`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document Storage</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        .upload-form {
            margin-bottom: 20px;
        }
        .document-list {
            list-style-type: none;
            padding: 0;
        }
        .document-list li {
            margin: 10px 0;
        }
    </style>
</head>
<body>
    <h1>Document Storage</h1>
    <form class="upload-form" action="/upload" method="post" enctype="multipart/form-data">
        <input type="file" name="file" required>
        <button type="submit">Upload</button>
    </form>

    <h2>Uploaded Documents</h2>
    <ul class="document-list">
        {% for doc in documents %}
            <li>
                <a href="{{ url_for('download_file', doc_id=doc[0]) }}">{{ doc[1] }}</a>
            </li>
        {% endfor %}
    </ul>
</body>
</html>
```

---

### **3. Run the Application**
1. Save the `app.py` file and the `index.html` file in the `templates` folder.
2. Run the Flask application:
   ```bash
   python app.py
   ```
3. Open your browser and go to `http://127.0.0.1:5000/`.

---

### **4. Features**
- Upload documents (txt, pdf, images, etc.).
- Store document metadata in SQLite.
- Download uploaded documents.

---

### **5. Enhancements**
- Add user authentication (e.g., Flask-Login).
- Improve file storage (e.g., use cloud storage like AWS S3).
- Add file preview functionality.
- Implement search and categorization for documents.

Let me know if you need help with any of these enhancements!
