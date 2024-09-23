# Python-Project
import tkinter as tk
from tkinter import messagebox, ttk
import sqlite3


def init_db():
    with sqlite3.connect('customers.db') as conn:
        cursor = conn.cursor()
        cursor.execute('''
        CREATE TABLE IF NOT EXISTS customers (
            customer_id INTEGER PRIMARY KEY,
            name TEXT,
            address TEXT,
            mobile_no TEXT,
            category TEXT
        )
        ''')

def fetch_customers():
    with sqlite3.connect('customers.db') as conn:
        cursor = conn.cursor()
        cursor.execute('SELECT * FROM customers')
        return cursor.fetchall()

def display_customers():
    for row in tree.get_children():
        tree.delete(row)
    for customer in fetch_customers():
        tree.insert('', 'end', values=customer)

def add_customer():
    data = (entry_name.get(), entry_address.get(), entry_mobile.get(), entry_category.get())
    if any(not field for field in data):
        messagebox.showinfo("Error", "All fields are required.")
        return
    with sqlite3.connect('customers.db') as conn:
        cursor = conn.cursor()
        cursor.execute('INSERT INTO customers (name, address, mobile_no, category) VALUES (?, ?, ?, ?)', data)
    messagebox.showinfo("Success", "Customer added successfully.")
    display_customers()

def modify_customer():
    customer_id = entry_id.get()
    data = (entry_name.get(), entry_address.get(), entry_mobile.get(), entry_category.get(), customer_id)
    with sqlite3.connect('customers.db') as conn:
        cursor = conn.cursor()
        cursor.execute('UPDATE customers SET name=?, address=?, mobile_no=?, category=? WHERE customer_id=?', data)
    if cursor.rowcount == 0:
        messagebox.showinfo("Error", "Customer ID not found.")
    else:
        messagebox.showinfo("Success", "Record modified successfully.")
    display_customers()

def delete_customer():
    customer_id = entry_id.get()
    with sqlite3.connect('customers.db') as conn:
        cursor = conn.cursor()
        cursor.execute('DELETE FROM customers WHERE customer_id=?', (customer_id,))
    if cursor.rowcount == 0:
        messagebox.showinfo("Error", "Customer ID not found.")
    else:
        messagebox.showinfo("Success", "Record deleted successfully.")
    display_customers()

def display_customer():
    customer_id = entry_id.get()
    with sqlite3.connect('customers.db') as conn:
        cursor = conn.cursor()
        customer = cursor.execute('SELECT * FROM customers WHERE customer_id=?', (customer_id,)).fetchone()
    if customer:
        entry_name.delete(0, tk.END)
        entry_address.delete(0, tk.END)
        entry_mobile.delete(0, tk.END)
        entry_category.set(customer[4])
        entry_name.insert(0, customer[1])
        entry_address.insert(0, customer[2])
        entry_mobile.insert(0, customer[3])
    else:
        messagebox.showinfo("Error", "Customer ID not found.")

root = tk.Tk()
root.title("Customer Management System")
root.geometry("600x500")
root.configure(bg="lightpink")


frame = tk.Frame(root, bg="lightcoral", bd=5, relief=tk.RAISED)
frame.pack(pady=10, padx=10, fill=tk.BOTH, expand=True)


labels = ["Customer ID:", "Name:", "Address:", "Mobile No:", "Category:"]
entries = [tk.Entry(frame) for _ in labels]
entries[4] = ttk.Combobox(frame, values=["New", "Regular", "Premium"], state="readonly")

for i, label in enumerate(labels):
    tk.Label(frame, text=label, bg="lightcoral", font=("Arial", 12)).grid(row=i, column=0, padx=5, pady=5)
    entries[i].grid(row=i, column=1, padx=5, pady=5)

entry_id, entry_name, entry_address, entry_mobile, entry_category = entries


button_texts = ["Display Customer", "Add Customer", "Update Customer", "Delete Customer"]
commands = [display_customer, add_customer, modify_customer, delete_customer]
for i, text in enumerate(button_texts):
    tk.Button(frame, text=text, command=commands[i], bg='lightgreen', font=("Arial", 10), width=20).grid(row=5+i, columnspan=2, pady=5)

columns = ("Customer ID", "Name", "Address", "Mobile No", "Category")
tree = ttk.Treeview(root, columns=columns, show='headings', height=10)
for col in columns:
    tree.heading(col, text=col)
    tree.column(col, anchor="center")
tree.pack(pady=10, fill=tk.BOTH, expand=True)

init_db()
display_customers()


root.mainloop()
