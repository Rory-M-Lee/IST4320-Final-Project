# IST4320-Final-Project
For my project I decided to creat an app that allows users to enter their birthdays into a database using buttons for their birth day, month, and year. A function this app has is that it adds all the birthdays together by month, day, and year and allows the user to see which birthday month/day/year is the most popular between the users on the app. This can be used by people as a whole or it could be tailored to see which birthday month/day/year is the most popular between a group of friends or within a company.


import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3
from collections import Counter


def init_db():
    conn = sqlite3.connect('birthdays.db')
    cursor = conn.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS birthdays (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT,
            day TEXT,
            month TEXT,
            year TEXT
        )
    ''')
    conn.commit()
    conn.close()

def save_to_db(name, day, month, year):
    conn = sqlite3.connect('birthdays.db')
    cursor = conn.cursor()
    cursor.execute('INSERT INTO birthdays (name, day, month, year) VALUES (?, ?, ?, ?)', (name, day, month, year))
    conn.commit()
    conn.close()

def get_all_birthdays():
    conn = sqlite3.connect('birthdays.db')
    cursor = conn.cursor()
    cursor.execute('SELECT name, day, month, year FROM birthdays')
    rows = cursor.fetchall()
    conn.close()
    return rows


def show_counts(group_by):
    birthdays = get_all_birthdays()
    if not birthdays:
        messagebox.showinfo("No Data", "No birthdays saved yet.")
        return

  if group_by == "month":
        values = [month for _, _, month, _ in birthdays]
        title = "Monthly Birthday Counts"
  elif group_by == "day":
        values = [day for _, day, _, _ in birthdays]
        title = "Birthdays by Day of Month"
  elif group_by == "year":
        values = [year for _, _, _, year in birthdays]
        title = "Birthdays by Year"
   else:
        return

   counts = Counter(values)

  count_window = tk.Toplevel(root)
    count_window.title(title)

   label = tk.Label(count_window, text=title, font=("Arial", 14, "bold"))
    label.pack(pady=10)

  if group_by == "month":
        for month in sorted(counts, key=lambda m: months.index(m)):
            tk.Label(count_window, text=f"{month}: {counts[month]} birthdays", font=("Arial", 12)).pack(anchor='w', padx=20)
  else:
        for value in sorted(counts, key=lambda x: int(x)):
            tk.Label(count_window, text=f"{value}: {counts[value]} birthdays", font=("Arial", 12)).pack(anchor='w', padx=20)


def update_label():
    day = day_var.get()
    month = month_var.get()
    year = year_var.get()
    
  if day != "Day" and month != "Month" and year != "Year":
        formatted_date = f"{month} {day}, {year}"
        result_var.set(formatted_date)
    else:
        result_var.set("Please select all fields.")

def on_save():
    name = name_var.get().strip()
    day = day_var.get()
    month = month_var.get()
    year = year_var.get()
    
   if not name:
        messagebox.showwarning("Missing Name", "Please enter a name.")
        return
    if day == "Day" or month == "Month" or year == "Year":
        messagebox.showwarning("Incomplete Data", "Please select all date fields.")
        return

  save_to_db(name, day, month, year)
    messagebox.showinfo("Saved", f"Saved: {name} - {month} {day}, {year}")
    name_entry.delete(0, tk.END)

def show_birthdays():
    birthdays = get_all_birthdays()
    if not birthdays:
        messagebox.showinfo("No Data", "No birthdays saved yet.")
        return

   view_window = tk.Toplevel(root)
    view_window.title("Saved Birthdays")

  label = tk.Label(view_window, text="Saved Birthdays", font=("Arial", 14, "bold"))
    label.pack(pady=10)

   listbox = tk.Listbox(view_window, width=50, font=("Arial", 12))
    listbox.pack(padx=10, pady=10)

   for name, day, month, year in birthdays:
        listbox.insert(tk.END, f"{name} - {month} {day}, {year}")


init_db()

root = tk.Tk()
root.title("Birthday Manager")
root.geometry("400x520")


name_var = tk.StringVar()
day_var = tk.StringVar(value="Day")
month_var = tk.StringVar(value="Month")
year_var = tk.StringVar(value="Year")
result_var = tk.StringVar()


months = ["January", "February", "March", "April", "May", "June",
          "July", "August", "September", "October", "November", "December"]


name_frame = tk.Frame(root)
name_frame.pack(pady=10)

tk.Label(name_frame, text="Name:", font=("Arial", 12)).pack(side=tk.LEFT)
name_entry = tk.Entry(name_frame, textvariable=name_var, font=("Arial", 12), width=30)
name_entry.pack(side=tk.LEFT, padx=5)


top_frame = tk.Frame(root)
top_frame.pack(pady=10)

day_menu = ttk.OptionMenu(top_frame, day_var, "Day", *list(range(1, 32)), command=lambda _: update_label())
day_menu.grid(row=0, column=0, padx=5)

month_menu = ttk.OptionMenu(top_frame, month_var, "Month", *months, command=lambda _: update_label())
month_menu.grid(row=0, column=1, padx=5)

years = list(range(1900, 2025))
year_menu = ttk.OptionMenu(top_frame, year_var, "Year", *years, command=lambda _: update_label())
year_menu.grid(row=0, column=2, padx=5)


output_box = tk.Entry(root, textvariable=result_var, font=("Arial", 14), state="readonly", width=35, justify="center")
output_box.pack(pady=10)


save_button = ttk.Button(root, text="Save Birthday", command=on_save)
save_button.pack(pady=5)

view_button = ttk.Button(root, text="View Saved Birthdays", command=show_birthdays)
view_button.pack(pady=5)

count_month_btn = ttk.Button(root, text="Count by Month", command=lambda: show_counts("month"))
count_month_btn.pack(pady=5)

count_day_btn = ttk.Button(root, text="Count by Day", command=lambda: show_counts("day"))
count_day_btn.pack(pady=5)

count_year_btn = ttk.Button(root, text="Count by Year", command=lambda: show_counts("year"))
count_year_btn.pack(pady=5)

root.mainloop()
