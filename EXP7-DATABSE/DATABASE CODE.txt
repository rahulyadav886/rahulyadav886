import sqlite3
import tkinter as tk
from tkinter import ttk, messagebox

# Database setup
def connect_db():
    conn = sqlite3.connect("students.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS students (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            age INTEGER NOT NULL,
            grade TEXT NOT NULL
        )
    """)
    conn.commit()
    conn.close()

# Functions
def add_student():
    name = name_var.get()
    age = age_var.get()
    grade = grade_var.get()
    if name and age and grade:
        conn = sqlite3.connect("students.db")
        cursor = conn.cursor()
        cursor.execute("INSERT INTO students (name, age, grade) VALUES (?, ?, ?)", (name, age, grade))
        conn.commit()
        conn.close()
        messagebox.showinfo("Success", "Student added successfully!")
        load_students()
    else:
        messagebox.showerror("Error", "All fields are required!")

def load_students():
    for row in student_table.get_children():
        student_table.delete(row)
    conn = sqlite3.connect("students.db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM students")
    for row in cursor.fetchall():
        student_table.insert("", "end", values=row)
    conn.close()

def delete_student():
    selected_item = student_table.selection()
    if selected_item:
        student_id = student_table.item(selected_item)['values'][0]
        conn = sqlite3.connect("students.db")
        cursor = conn.cursor()
        cursor.execute("DELETE FROM students WHERE id=?", (student_id,))
        conn.commit()
        conn.close()
        messagebox.showinfo("Deleted", "Student deleted successfully!")
        load_students()
    else:
        messagebox.showerror("Error", "Please select a student to delete!")

def update_student():
    selected_item = student_table.selection()
    if selected_item:
        student_id = student_table.item(selected_item)['values'][0]
        name = name_var.get()
        age = age_var.get()
        grade = grade_var.get()
        if name and age and grade:
            conn = sqlite3.connect("students.db")
            cursor = conn.cursor()
            cursor.execute("UPDATE students SET name=?, age=?, grade=? WHERE id=?", (name, age, grade, student_id))
            conn.commit()
            conn.close()
            messagebox.showinfo("Updated", "Student updated successfully!")
            load_students()
        else:
            messagebox.showerror("Error", "All fields are required for update!")
    else:
        messagebox.showerror("Error", "Please select a student to update!")

# GUI Setup
root = tk.Tk()
root.title("Student Management System")
root.geometry("500x400")

name_var = tk.StringVar()
age_var = tk.StringVar()
grade_var = tk.StringVar()

tk.Label(root, text="Name:").pack()
tk.Entry(root, textvariable=name_var).pack()

tk.Label(root, text="Age:").pack()
tk.Entry(root, textvariable=age_var).pack()

tk.Label(root, text="Grade:").pack()
tk.Entry(root, textvariable=grade_var).pack()

tk.Button(root, text="Add Student", command=add_student).pack()
tk.Button(root, text="Update Student", command=update_student).pack()

student_table = ttk.Treeview(root, columns=("ID", "Name", "Age", "Grade"), show="headings")
student_table.heading("ID", text="ID")
student_table.heading("Name", text="Name")
student_table.heading("Age", text="Age")
student_table.heading("Grade", text="Grade")
student_table.pack()

tk.Button(root, text="Delete Student", command=delete_student).pack()

connect_db()
load_students()
root.mainloop()