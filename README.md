import json
import os
from tkinter import *
from tkinter import ttk, messagebox

DATA_FILE = "books.json"

class BookTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Book Tracker — Трекер прочитанных книг")
        self.root.geometry("900x500")

        self.books = []
        self.load_data()

        # Поля ввода
        input_frame = LabelFrame(root, text="Добавить книгу", padx=10, pady=10)
        input_frame.pack(pady=10, fill="x", padx=10)

        Label(input_frame, text="Название:").grid(row=0, column=0, sticky="w")
        self.title_entry = Entry(input_frame, width=25)
        self.title_entry.grid(row=0, column=1, padx=5, pady=2)

        Label(input_frame, text="Автор:").grid(row=0, column=2, sticky="w")
        self.author_entry = Entry(input_frame, width=20)
        self.author_entry.grid(row=0, column=3, padx=5, pady=2)

        Label(input_frame, text="Жанр:").grid(row=0, column=4, sticky="w")
        self.genre_entry = Entry(input_frame, width=15)
        self.genre_entry.grid(row=0, column=5, padx=5, pady=2)

        Label(input_frame, text="Страниц:").grid(row=0, column=6, sticky="w")
        self.pages_entry = Entry(input_frame, width=8)
        self.pages_entry.grid(row=0, column=7, padx=5, pady=2)

        self.add_btn = Button(input_frame, text="Добавить книгу", command=self.add_book, bg="lightgreen")
        self.add_btn.grid(row=0, column=8, padx=10)

        # Фильтры
        filter_frame = LabelFrame(root, text="Фильтрация", padx=10, pady=10)
        filter_frame.pack(pady=5, fill="x", padx=10)

        Label(filter_frame, text="Фильтр по жанру:").grid(row=0, column=0, sticky="w")
        self.genre_filter = Entry(filter_frame, width=20)
        self.genre_filter.grid(row=0, column=1, padx=5)
        self.genre_filter.bind("<KeyRelease>", self.apply_filters)

        Label(filter_frame, text="Страниц >").grid(row=0, column=2, sticky="w", padx=(10,0))
        self.pages_filter = Entry(filter_frame, width=8)
        self.pages_filter.grid(row=0, column=3, padx=5)
        self.pages_filter.bind("<KeyRelease>", self.apply_filters)

        self.filter_btn = Button(filter_frame, text="Сбросить фильтры", command=self.reset_filters)
        self.filter_btn.grid(row=0, column=4, padx=10)

        # Таблица книг
        columns = ("Название", "Автор", "Жанр", "Страниц")
        self.tree = ttk.Treeview(root, columns=columns, show="headings")
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=180)
        self.tree.pack(fill=BOTH, expand=True, padx=10, pady=10)

        # Кнопки сохранения/загрузки
        btn_frame = Frame(root)
        btn_frame.pack(pady=5)

        self.save_btn = Button(btn_frame, text="Сохранить в JSON", command=self.save_to_json, bg="lightblue")
        self.save_btn.pack(side=LEFT, padx=5)

        self.load_btn = Button(btn_frame, text="Загрузить из JSON", command=self.load_from_json, bg="lightyellow")
        self.load_btn.pack(side=LEFT, padx=5)

        self.refresh_table()

    def add_book(self):
        title = self.title_entry.get().strip()
        author = self.author_entry.get().strip()
        genre = self.genre_entry.get().strip()
        pages_str = self.pages_entry.get().strip()

        if not title or not author or not genre or not pages_str:
            messagebox.showerror("Ошибка", "Все поля должны быть заполнены!")
            return

        if not pages_str.isdigit():
            messagebox.showerror("Ошибка", "Количество страниц должно быть числом!")
            return

        pages = int(pages_str)
        self.books.append({
            "title": title,
            "author": author,
            "genre": genre,
            "pages": pages
        })
        self.clear_entries()
        self.refresh_table()

    def clear_entries(self):
        self.title_entry.delete(0, END)
        self.author_entry.delete(0, END)
        self.genre_entry.delete(0, END)
        self.pages_entry.delete(0, END)

    def apply_filters(self, event=None):
        self.refresh_table()

    def reset_filters(self):
        self.genre_filter.delete(0, END)
        self.pages_filter.delete(0, END)
        self.refresh_table()

    def get_filtered_books(self):
        genre_filter = self.genre_filter.get().strip().lower()
        pages_filter = self.pages_filter.get().strip()

        filtered = self.books[:]
        if genre_filter:
            filtered = [b for b in filtered if genre_filter in b["genre"].lower()]
        if pages_filter and pages_filter.isdigit():
            min_pages = int(pages_filter)
            filtered = [b for b in filtered if b["pages"] > min_pages]
        return filtered

    def refresh_table(self):
        for row in self.tree.get_children():
            self.tree.delete(row)

        for book in self.get_filtered_books():
            self.tree.insert("", END, values=(book["title"], book["author"], book["genre"], book["pages"]))

    def save_to_json(self):
        try:
            with open(DATA_FILE, "w", encoding="utf-8") as f:
                json.dump(self.books, f, indent=4, ensure_ascii=False)
            messagebox.showinfo("Успех", f"Данные сохранены в {DATA_FILE}")
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось сохранить: {e}")

    def load_from_json(self):
        try:
            with open(DATA_FILE, "r", encoding="utf-8") as f:
                self.books = json.load(f)
            self.refresh_table()
            messagebox.showinfo("Успех", f"Данные загружены из {DATA_FILE}")
        except FileNotFoundError:
            messagebox.showerror("Ошибка", f"Файл {DATA_FILE} не найден")
        except Exception as e:
            messagebox.showerror("Ошибка", f"Ошибка загрузки: {e}")

    def load_data(self):
        if os.path.exists(DATA_FILE):
            self.load_from_json()

if __name__ == "__main__":
    root = Tk()
    app = BookTracker(root)
    root.mainloop()
