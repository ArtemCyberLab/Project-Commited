The project titled "commited" is a demonstration of working with the Git version control system, with a focus on analyzing commit history and recovering leaked sensitive information. The main idea is to show how careless handling of a repository can lead to accidentally storing confidential data such as passwords or access tokens in the Git history. In this project, we examine a repository where credentials for connecting to a MySQL database were committed and later an attempt was made to hide them—without actually cleaning up the commit history.

The project is implemented in Python using the mysql.connector library. The script main.py includes a sequence of functions to create a database, create tables, and insert data. It connects to a local MySQL server using the root user, with a password that was unintentionally left in the source code in one of the earlier commits. An example of the code:

import mysql.connector

def create_db():
    mydb = mysql.connector.connect(
        host="localhost",
        user="root",
        password="flag{a489a9dbf8eb9d37c6e0cc1a92cda17b}"
    )
    mycursor = mydb.cursor()
    mycursor.execute("CREATE DATABASE IF NOT EXISTS commited")

def create_tables():
    mydb = mysql.connector.connect(
        host="localhost",
        user="root",
        password="flag{a489a9dbf8eb9d37c6e0cc1a92cda17b}",
        database="commited"
    )
    mycursor = mydb.cursor()
    mycursor.execute("""
        CREATE TABLE IF NOT EXISTS customers (
            id INT AUTO_INCREMENT PRIMARY KEY,
            name VARCHAR(255),
            address VARCHAR(255),
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    """)

def populate_tables():
    mydb = mysql.connector.connect(
        host="localhost",
        user="root",
        password="flag{a489a9dbf8eb9d37c6e0cc1a92cda17b}",
        database="commited"
    )
    mycursor = mydb.cursor()
    customers = [
        ("Alice", "Apple St. 12"),
        ("Bob", "Berry Ave. 34"),
        ("Charlie", "Coconut Blvd. 56")
    ]
    sql = "INSERT INTO customers (name, address) VALUES (%s, %s)"
    mycursor.executemany(sql, customers)
    mydb.commit()
    print(f"{mycursor.rowcount} records inserted.")

def show_customers():
    mydb = mysql.connector.connect(
        host="localhost",
        user="root",
        password="flag{a489a9dbf8eb9d37c6e0cc1a92cda17b}",
        database="commited"
    )
    mycursor = mydb.cursor()
    mycursor.execute("SELECT * FROM customers")
    for record in mycursor.fetchall():
        print(record)

if __name__ == "__main__":
    create_db()
    create_tables()
    populate_tables()
    show_customers()
    The repository contains several commits, one of which—3a8cc16—contains the full working code with the password included. Checking out this commit with git checkout 3a8cc16 allowed recovery of the leaked credentials. Before this commit, another one with the hash c56c470 includes a message saying Oops, suggesting that the developer may have tried to fix or remove something. This illustrates a real-world scenario where sensitive data is deleted from the latest code but remains in the Git history.

There are also text files such as Note and Readme.md in the project, where the developer likely left reminders. These files might have contained clues about which commit to investigate for leaks.

Thus, the project not only implements a basic database structure and interactions but also demonstrates an important security lesson: Git remembers everything. Accidentally committed secrets can be extracted even long after deletion. In production, it is highly recommended to use .gitignore, environment variables, git filter-repo for history rewriting, and to never hardcode secrets directly into the codebase.

This project is valuable as an educational example for cybersecurity labs, penetration testing training, and DevSecOps best practices.
