import tkinter as tk
from tkinter import messagebox
import json
import os

STORAGE_FILE = "todo_data.json"

def load_tasks():
    if not os.path.exists(STORAGE_FILE):
        return []
    with open(STORAGE_FILE, "r", encoding="utf-8") as f:
        try:
            return json.load(f)
        except Exception:
            return []

def save_tasks(tasks):
    with open(STORAGE_FILE, "w", encoding="utf-8") as f:
        json.dump(tasks, f, ensure_ascii=False, indent=2)

class TodoApp:
    def __init__(self, root):
        self.root = root
        self.root.title("To-Do List")

        self.tasks = load_tasks()

        self.frame = tk.Frame(root)
        self.frame.pack(padx=12, pady=12)

        self.task_entry = tk.Entry(self.frame, width=32)
        self.task_entry.grid(row=0, column=0, padx=4)
        self.add_btn = tk.Button(self.frame, text="Add", command=self.add_task)
        self.add_btn.grid(row=0, column=1, padx=4)

        self.listbox = tk.Listbox(self.frame, width=40, height=12)
        self.listbox.grid(row=1, column=0, columnspan=2, pady=8)
        self.listbox.bind('<Double-Button-1>', self.toggle_done)

        self.del_btn = tk.Button(self.frame, text="Delete", command=self.delete_task)
        self.del_btn.grid(row=2, column=0, columnspan=2, pady=6)

        self.refresh_listbox()

    def refresh_listbox(self):
        self.listbox.delete(0, tk.END)
        for i, task in enumerate(self.tasks):
            text = f"[{'✓' if task['done'] else ' '}] {task['text']}"
            self.listbox.insert(tk.END, text)

    def add_task(self):
        text = self.task_entry.get().strip()
        if text:
            self.tasks.append({"text": text, "done": False})
            save_tasks(self.tasks)
            self.task_entry.delete(0, tk.END)
            self.refresh_listbox()
        else:
            messagebox.showwarning("Empty", "Please enter a task.")

    def delete_task(self):
        selection = self.listbox.curselection()
        if selection:
            idx = selection[0]
            self.tasks.pop(idx)
            save_tasks(self.tasks)
            self.refresh_listbox()
        else:
            messagebox.showwarning("Select", "Select a task to delete.")

    def toggle_done(self, event):
        selection = self.listbox.curselection()
        if selection:
            idx = selection[0]
            self.tasks[idx]["done"] = not self.tasks[idx]["done"]
            save_tasks(self.tasks)
            self.refresh_listbox()

if __name__ == "__main__":
    root = tk.Tk()
    app = TodoApp(root)
    root.mainloop()
