import mysql.connector

def connect_db():
    try:
        db = mysql.connector.connect(
            host="127.0.0.1",
            port=3306,
            user="root",
            password="",  # Leave empty if no password
            database="Mini project"  # Use the correct database name
        )
        print("Connected to MySQL successfully!")
        return db
    except mysql.connector.Error as err:
        print(f"Error: {err}")
        return None

# Book Operations
def add_book(title, author_id, isbn, publication_date):
    db = connect_db()
    if db:
        cursor = db.cursor()
        sql = "INSERT INTO books (title, author_id, isbn, publication_date) VALUES (%s, %s, %s, %s)"
        cursor.execute(sql, (title, author_id, isbn, publication_date))
        db.commit()
        print("Book added successfully!")
        cursor.close()
        db.close()

def search_book(title=None, author=None):
    db = connect_db()
    if db:
        cursor = db.cursor()
        query = "SELECT * FROM books WHERE title LIKE %s OR author_id IN (SELECT id FROM authors WHERE name LIKE %s)"
        cursor.execute(query, (f"%{title}%", f"%{author}%"))
        books = cursor.fetchall()
        if books:
            for book in books:
                print(book)
        else:
            print("No books found.")
        cursor.close()
        db.close()

def borrow_book(user_id, book_id):
    db = connect_db()
    if db:
        cursor = db.cursor()
        cursor.execute("SELECT availability FROM books WHERE id = %s", (book_id,))
        book = cursor.fetchone()
        if book and book[0]:
            cursor.execute("""
                INSERT INTO borrowed_books (user_id, book_id, borrow_date)
                VALUES (%s, %s, NOW())
            """, (user_id, book_id))
            cursor.execute("UPDATE books SET availability = 0 WHERE id = %s", (book_id,))
            db.commit()
            print("Book borrowed successfully!")
        else:
            print("Book is not available.")
        cursor.close()
        db.close()

def return_book(user_id, book_id):
    db = connect_db()
    if db:
        cursor = db.cursor()
        cursor.execute("UPDATE borrowed_books SET return_date = NOW() WHERE user_id = %s AND book_id = %s AND return_date IS NULL", (user_id, book_id))
        cursor.execute("UPDATE books SET availability = 1 WHERE id = %s", (book_id,))
        db.commit()
        print("Book returned successfully!")
        cursor.close()
        db.close()

# User Operations
def add_user(name, library_id):
    db = connect_db()
    if db:
        cursor = db.cursor()
        sql = "INSERT INTO users (name, library_id) VALUES (%s, %s)"
        cursor.execute(sql, (name, library_id))
        db.commit()
        print("User added successfully!")
        cursor.close()
        db.close()

def view_user(library_id):
    db = connect_db()
    if db:
        cursor = db.cursor()
        cursor.execute("SELECT * FROM users WHERE library_id = %s", (library_id,))
        user = cursor.fetchone()
        if user:
            print(user)
        else:
            print("User not found.")
        cursor.close()
        db.close()

# Author Operations
def add_author(name, biography):
    db = connect_db()
    if db:
        cursor = db.cursor()
        sql = "INSERT INTO authors (name, biography) VALUES (%s, %s)"
        cursor.execute(sql, (name, biography))
        db.commit()
        print("Author added successfully!")
        cursor.close()
        db.close()

def view_author(author_id):
    db = connect_db()
    if db:
        cursor = db.cursor()
        cursor.execute("SELECT * FROM authors WHERE id = %s", (author_id,))
        author = cursor.fetchone()
        if author:
            print(author)
        else:
            print("Author not found.")
        cursor.close()
        db.close()

# Menu for Operations
def book_operations():
    while True:
        print("\nBook Operations")
        print("1. Add a new book")
        print("2. Search for a book")
        print("3. Borrow a book")
        print("4. Return a book")
        print("5. Go back")
        
        choice = input("Enter your choice: ")
        if choice == "1":
            title = input("Enter book title: ")
            author_id = input("Enter author ID: ")
            isbn = input("Enter ISBN: ")
            pub_date = input("Enter publication date (YYYY-MM-DD): ")
            add_book(title, author_id, isbn, pub_date)
        elif choice == "2":
            title = input("Enter title to search (or leave blank): ")
            author = input("Enter author name to search (or leave blank): ")
            search_book(title, author)
        elif choice == "3":
            user_id = input("Enter user ID: ")
            book_id = input("Enter book ID to borrow: ")
            borrow_book(user_id, book_id)
        elif choice == "4":
            user_id = input("Enter user ID: ")
            book_id = input("Enter book ID to return: ")
            return_book(user_id, book_id)
        elif choice == "5":
            break
        else:
            print("Invalid choice. Try again.")

def user_operations():
    while True:
        print("\nUser Operations")
        print("1. Add a new user")
        print("2. View user details")
        print("3. Go back")
        
        choice = input("Enter your choice: ")
        if choice == "1":
            name = input("Enter user name: ")
            library_id = input("Enter library ID: ")
            add_user(name, library_id)
        elif choice == "2":
            library_id = input("Enter library ID to view details: ")
            view_user(library_id)
        elif choice == "3":
            break
        else:
            print("Invalid choice. Try again.")

def author_operations():
    while True:
        print("\nAuthor Operations")
        print("1. Add a new author")
        print("2. View author details")
        print("3. Go back")
        
        choice = input("Enter your choice: ")
        if choice == "1":
            name = input("Enter author name: ")
            biography = input("Enter author biography: ")
            add_author(name, biography)
        elif choice == "2":
            author_id = input("Enter author ID to view details: ")
            view_author(author_id)
        elif choice == "3":
            break
        else:
            print("Invalid choice. Try again.")

def main_menu():
    while True:
        print("\nLibrary Management System")
        print("1. Book Operations")
        print("2. User Operations")
        print("3. Author Operations")
        print("4. Quit")
        
        choice = input("Enter your choice: ")
        if choice == "1":
            book_operations()
        elif choice == "2":
            user_operations()
        elif choice == "3":
            author_operations()
        elif choice == "4":
            print("Exiting...")
            break
        else:
            print("Invalid choice. Please try again.")
