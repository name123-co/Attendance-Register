# Attendance-Registeimport sqlite3
import tkinter as tk
from tkinter import ttk, messagebox
from datetime import datetime

# Database Setup
def init_db():
    conn = sqlite3.connect("attendance.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS attendance (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            teacher_name TEXT,
            status TEXT,
            date TEXT,
            time TEXT
        )
    """)
    conn.commit()
    conn.close()

# Mark Attendance with Punctuality Check
def mark_attendance():
    teacher_name = entry_name.get().strip()
    if teacher_name:
        # Get current time and compare it with 8:30 AM
        current_time = datetime.now().strftime("%H:%M:%S")
        punctual_time = "08:30:00"
       
        # Check if the teacher is on time or late
        if current_time < punctual_time:
            status = "On Time"
        else:
            status = "Late"
       
        current_date = datetime.now().strftime("%Y-%m-%d")
       
        # Insert attendance record into the database
        conn = sqlite3.connect("attendance.db")
        cursor = conn.cursor()
        cursor.execute("INSERT INTO attendance (teacher_name, status, date, time) VALUES (?, ?, ?, ?)",
                       (teacher_name, status, current_date, current_time))
        conn.commit()
        conn.close()
       
        entry_name.delete(0, tk.END)
        update_attendance_list()
        messagebox.showinfo("Success", f"Attendance marked for {teacher_name} as {status} at {current_time}")
    else:
        messagebox.showwarning("Input Error", "Please enter a valid name!")

# View Attendance
def update_attendance_list():
    for row in tree.get_children():
        tree.delete(row)
   
    conn = sqlite3.connect("attendance.db")
    cursor = conn.cursor()
    cursor.execute("SELECT teacher_name, status, date, time FROM attendance")
    records = cursor.fetchall()
    conn.close()
   
    for record in records:
        tree.insert("", "end", values=record)

# Delete All Attendance Records
def delete_all_attendance():
    if messagebox.askyesno("Delete All", "Are you sure you want to delete all attendance records?"):
        conn = sqlite3.connect("attendance.db")
        cursor = conn.cursor()
        cursor.execute("DELETE FROM attendance")
        conn.commit()
        conn.close()
        update_attendance_list()
        messagebox.showinfo("Success", "All attendance records deleted.")

# Exit Application
def exit_app():
    if messagebox.askyesno("Exit", "Are you sure you want to exit?"):
        root.destroy()

# GUI Setup
root = tk.Tk()
root.title("Attendance System")
root.geometry("900x600")
root.configure(bg="#0F0F0F")

# Frame for header and title
frame = tk.Frame(root, bg="#00FF00")
frame.pack(fill="x")
tk.Label(frame, text="Attendance System", font=("Consolas", 24, "bold"), fg="#0F0F0F", bg="#00FF00").pack(pady=10)

# Teacher Name Entry
tk.Label(root, text="Enter Teacher Name:", font=("Consolas", 12, "bold"), fg="#00FF00", bg="#0F0F0F").pack(pady=5)
entry_name = tk.Entry(root, font=("Consolas", 12), width=30, bg="#1F1F1F", fg="#00FF00", relief="sunken", borderwidth=2)
entry_name.pack(pady=5)

# Buttons for functionalities
tk.Button(root, text="[+] Mark Present", font=("Consolas", 12, "bold"), bg="#00FF00", fg="#0F0F0F", relief="raised", command=mark_attendance).pack(pady=5, ipadx=10, ipady=5)
tk.Button(root, text="[X] Delete All Records", font=("Consolas", 12, "bold"), bg="#FF0000", fg="#FFFFFF", relief="raised", command=delete_all_attendance).pack(pady=5, ipadx=10, ipady=5)
tk.Button(root, text="[!] Exit", font=("Consolas", 12, "bold"), bg="#FFA500", fg="#0F0F0F", relief="raised", command=exit_app).pack(pady=5, ipadx=10, ipady=5)

# Attendance Table
tree_frame = tk.Frame(root, bg="#0F0F0F", bd=2, relief="ridge")
tree_frame.pack(pady=10, padx=10, fill="both", expand=True)
tree_scroll = tk.Scrollbar(tree_frame)
tree_scroll.pack(side="right", fill="y")

tree = ttk.Treeview(tree_frame, columns=("Teacher Name", "Status", "Date", "Time"), show="headings", yscrollcommand=tree_scroll.set, style="Treeview")
tree.heading("Teacher Name", text="Teacher Name")
tree.heading("Status", text="Status")
tree.heading("Date", text="Date")
tree.heading("Time", text="Time")

tree.column("Teacher Name", width=200, anchor="center")
tree.column("Status", width=120, anchor="center")
tree.column("Date", width=120, anchor="center")
tree.column("Time", width=120, anchor="center")

tree.pack(pady=10, padx=10, fill="both", expand=True)
tree_scroll.config(command=tree.yview)

# Initialize database and update attendance list
init_db()
update_attendance_list()
