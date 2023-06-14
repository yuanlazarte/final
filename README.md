1. download xampp and postman
2. create folder in desktop
3. open cmd
4. cd desktop, then foldername
5. git init .
6. git clone (url of you repository in github)
7. cd name of your github repository
8. notepad .gitignore then type Lib/
Scripts/
pyvenv.cfg
.gitignore
9. git statis
10. notepad rest_api.py, leave empty
11. git add (files you want to push)
12. git commit -m "your comment"
13. git push origin master
14. go to you github repository to double check the files that you pushed

from flask import Flask, make_response, jsonify, request
from flask_mysqldb import MySQL
import xml.etree.ElementTree as ET
import functools

app = Flask(__name__)
app.config["MYSQL_HOST"] = "127.0.0.1"
app.config["MYSQL_USER"] = "root"
app.config["MYSQL_PASSWORD"] = "083001"
app.config["MYSQL_DB"] = "school_db"
app.config["MYSQL_CURSORCLASS"] = "DictCursor"

def security(f):
    @functools.wraps(f)
    def decorated(*args, **kwargs):
        a = request.authorization
        if a and a.username == "school_db" and a.password == "083001":
            return f(*args, **kwargs)
        return make_response(
            "the username and password does not match!",
            401,
            {"WWW-Authenticate": 'Basic realm="Login"'},
        )

    return decorated


mysql = MySQL(app)

@app.route("/")
@security
def hello_world():
    return "<p>Hello, World!</p>"


def get_info(query):
    cur = mysql.connection.cursor()
    cur.execute(query)
    data = cur.fetchall()
    cur.close()
    return data

@app.route("/tables", methods=["GET"])
def show_tables():
    return make_response(jsonify(get_info("show tables")), 200)

@app.route("/tables/<string:table>", methods=["GET"])
def select_table(table):
    return make_response(jsonify(get_info(f"select * from {table}")))

@app.route("/tables/<string:table>/<int:id>", methods=["GET"])
def select_table_id(table, id):
    return make_response(jsonify(get_info(f"select * from {table} where student_id='{id}'")))


@app.route("/tables/<string:table>/<int:id>", methods=["POST"])
def add_entries(table, id):
    cur = mysql.connection.cursor()
    info = request.get_json()

    if table == "students":
        name = info["name"]
        age = info["age"]
        grade = info["grade"]
        query = "INSERT INTO students (student_id, name, age, grade) VALUES (%s, %s, %s, %s)"
        cur.execute(query, (id, name, age, grade))
        mysql.connection.commit()

    elif table == "teachers":
        name = info["name"]
        age = info["age"]
        subject = info["subject"]
        query = "INSERT INTO teachers (teacher_id, name, age, subject) VALUES (%s, %s, %s, %s)"
        cur.execute(query, (id, name, age, subject))
        mysql.connection.commit()

    cur.close()
    return make_response(jsonify({"Message": "Added successfully"}))


@app.route("/tables/<string:table>/<int:id>", methods=["PUT"])
def update_entry(table, id):
    cur = mysql.connection.cursor()
    info = request.get_json()

    if table == "students":
        name = info["name"]
        age = info["age"]
        grade = info["grade"]
        query = "UPDATE students SET name=%s, age=%s, grade=%s WHERE student_id=%s"
        cur.execute(query, (name, age, grade, id))
        mysql.connection.commit()

    elif table == "teachers":
        name = info["name"]
        age = info["age"]
        subject = info["subject"]
        query = "UPDATE teachers SET name=%s, age=%s, subject=%s, WHERE teacher_id=%s"
        cur.execute(query, (name, age, subject, id))
        mysql.connection.commit()


    cur.close()
    return make_response(jsonify({"Message": "Updated successfully"}))


@app.route("/tables/<string:table>/<int:id>", methods=["DELETE"])
def delete_by_id(table, id):
    cur = mysql.connection.cursor()

    if table == "students":
        query = "DELETE FROM students WHERE student_id = %s"
        cur.execute(query, (id,))
        mysql.connection.commit()

    elif table == "teachers":
        query = "DELETE FROM teachers WHERE teacher_id = %s"
        cur.execute(query, (id,))
        mysql.connection.commit()
        
    cur.close()
    return make_response(jsonify({"Message": "Deleted successfully"}))

if __name__ == "__main__":
    app.run(debug=True)
