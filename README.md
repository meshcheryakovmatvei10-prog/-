#import tkinter as tk
from tkinter import ttk, messagebox, filedialog
import json
from datetime import datetime
import os

class WeatherDiary:
    def __init__(self, root):
        self.root = root
        self.root.title("Дневник погоды")

        self.entries = []
        self.data_file = "weather_diary.json"
        self.load_entries()

        # --- Фрейм для ввода данных ---
        self.input_frame = ttk.LabelFrame(root, text="Добавить запись", padding="10")
        self.input_frame.grid(row=0, column=0, padx=10, pady=10, sticky=(tk.W, tk.E, tk.N, tk.S))

        ttk.Label(self.input_frame, text="Дата (YYYY-MM-DD):").grid(row=0, column=0, sticky=tk.W, pady=2)
        self.date_entry = ttk.Entry(self.input_frame, width=20)
        self.date_entry.grid(row=0, column=1, sticky=(tk.W, tk.E), pady=2)
        self.date_entry.insert(0, datetime.now().strftime("%Y-%m-%d"))

        ttk.Label(self.input_frame, text="Температура (°C):").grid(row=1, column=0, sticky=tk.W, pady=2)
        self.temp_entry = ttk.Entry(self.input_frame, width=20)
        self.temp_entry.grid(row=1, column=1, sticky=(tk.W, tk.E), pady=2)

        ttk.Label(self.input_frame, text="Описание:").grid(row=2, column=0, sticky=tk.W, pady=2)
        self.description_entry = ttk.Entry(self.input_frame, width=30)
        self.description_entry.grid(row=2, column=1, sticky=(tk.W, tk.E), pady=2)

        ttk.Label(self.input_frame, text="Осадки:").grid(row=3, column=0, sticky=tk.W, pady=2)
        self.precipitation_var = tk.StringVar(value="нет")
        self.precipitation_yes = ttk.Radiobutton(self.input_frame, text="Да", variable=self.precipitation_var, value="да")
        self.precipitation_yes.grid(row=3, column=1, sticky=tk.W)
        self.precipitation_no = ttk.Radiobutton(self.input_frame, text="Нет", variable=self.precipitation_var, value="нет")
        self.precipitation_no.grid(row=3, column=2, sticky=tk.W)

        ttk.Button(self.input_frame, text="Добавить запись", command=self.add_entry).grid(row=4, column=0, columnspan=3, pady=10)

        # --- Фрейм для фильтрации ---
        self.filter_frame = ttk.LabelFrame(root, text="Фильтр", padding="10")
        self.filter_frame.grid(row=0, column=1, padx=10, pady=10, sticky=(tk.W, tk.E, tk.N, tk.S))

        ttk.Label(self.filter_frame, text="По дате:").grid(row=0, column=0, sticky=tk.W)
        self.filter_date_entry = ttk.Entry(self.filter_frame, width=15)
        self.filter_date_entry.grid(row=0, column=1, padx=5)
        ttk.Button(self.filter_frame, text="Применить", command=self.apply_filter).grid(row=0, column=2)
        ttk.Button(self.filter_frame, text="Сбросить", command=self.reset_filter).grid(row=0, column=3)

        ttk.Label(self.filter_frame, text="Температура >").grid(row=1, column=0, sticky=tk.W)
        self.filter_temp_entry = ttk.Entry(self.filter_frame, width=10)
        self.filter_temp_entry.grid(row=1, column=1, padx=5, pady=5)
        ttk.Button(self.filter_frame, text="Применить", command=self.apply_filter).grid(row=1, column=2)

        # --- Фрейм для отображения записей ---
        self.list_frame = ttk.Frame(root, padding="10")
        self.list_frame.grid(row=1, column=0, columnspan=2, sticky=(tk.W, tk.E, tk.N, tk.S))

        self.tree = ttk.Treeview(self.list_frame, columns=("Date", "Temperature", "Description", "Precipitation"), show="headings")
        self.tree.heading("Date", text="Дата")
        self.tree.heading("Temperature", text="Температура (°C)")
        self.tree.heading("Description", text="Описание")
        self.tree.heading("Precipitation", text="Осадки")

        self.tree.column("Date", width=100, anchor=tk.CENTER)
        self.tree.column("Temperature", width=120, anchor=tk.CENTER)
        self.tree.column("Description", width=200)
        self.tree.column("Precipitation", width=80, anchor=tk.CENTER)

        self.tree.grid(row=0, column=0, sticky=(tk.W, tk.E, tk.N, tk.S))

        scrollbar = ttk.Scrollbar(self.list_frame, orient=tk.VERTICAL, command=
Welcome to nginx!
tk.CENTER


self.tree.yview)
        scrollbar.grid(row=0, column=1, sticky=(tk.N, tk.S))
        self.tree.config(yscrollcommand=scrollbar.set)

        # --- Кнопки управления файлами ---
        self.file_buttons_frame = ttk.Frame(root, padding="10")
        self.file_buttons_frame.grid(row=0, column=2, rowspan=2, sticky=(tk.N, tk.S))
        ttk.Button(self.file_buttons_frame, text="Сохранить как...", command=self.save_entries_as).grid(row=0, column=0, pady=5, padx=5, sticky=tk.W+tk.E)
        ttk.Button(self.file_buttons_frame, text="Загрузить...", command=self.load_entries_from).grid(row=1, column=0, pady=5, padx=5, sticky=tk.W+tk.E)

        self.update_treeview()

    def validate_input(self, date_str, temp_str, description):
        if not date_str:
            messagebox.showerror("Ошибка ввода", "Поле 'Дата' не может быть пустым.")
            return False
        try:
            datetime.strptime(date_str, "%Y-%m-%d")
        except ValueError:
            messagebox.showerror("Ошибка ввода", "Неверный формат даты. Используйте YYYY-MM-DD.")
            return False

        if not temp_str:
            messagebox.showerror("Ошибка ввода", "Поле 'Температура' не может быть пустым.")
            return False
        try:
            float(temp_str)
        except ValueError:
            messagebox.showerror("Ошибка ввода", "Температура должна быть числом.")
            return False

        if not description:
            messagebox.showerror("Ошибка ввода", "Поле 'Описание' не должно быть пустым.")
            return False
        return True

    def add_entry(self):
        date = self.date_entry.get().strip()
        temp = self.temp_entry.get().strip()
        description = self.description_entry.get().strip()
        precipitation = self.precipitation_var.get()

        if self.validate_input(date, temp, description):
            self.entries.append({
                "date": date,
                "temperature": float(temp),
                "description": description,
                "precipitation": precipitation
            })
            self.update_treeview()
            self.clear_input_fields()
            self.save_entries() # Автоматическое сохранение после добавления

    def clear_input_fields(self):
        self.date_entry.delete(0, tk.END)
        self.date_entry.insert(0, datetime.now().strftime("%Y-%m-%d"))
        self.temp_entry.delete(0, tk.END)
        self.description_entry.delete(0, tk.END)
        self.precipitation_var.set("нет")

    def update_treeview(self, data_to_display=None):
        # Очищаем текущие записи в таблице
        for item in self.tree.get_children():
            self.tree.delete(item)

        # Заполняем таблицу данными
        data = data_to_display if data_to_display is not None else self.entries
        for entry inas(self):
        filepath = filedialog.asksaveasfilename(defaultextension=".json",
                                               filetypes=[("JSON files", "*.json"), ("All files", "*.*")],
                                               title="Сохранить записи дневника погоды как...")
        if not filepath:
            return
        self.data_file = filepath
        self.save_entries()
        messagebox.showinfo("Сохранено", f"Записи сохранены в файл: {self.data_file}")

    def save_entries(self):
        try:
            with open(self.data_file, "w", encoding="utf-8") as f:
                json.dump(self.entries, f, indent=4, ensure_ascii=False)
        except IOError:
            messagebox.showerror("Ошибка сохранения", f"Не удалось сохранить данные в файл {self.data_file}.")

    def load_entries_from(self):
        filepath = filedialog.askopenfilename(defaultextension=".json",
                                              filetypes=[("JSON files", "*.json"), ("All files", "*.*")],
                                              title="Загрузить записи дневника погоды из...")
        if not filepath:
            return
        self.data_file = filepath


self.load_entries()
        self.update_treeview()

    def load_entries(self):
        if os.path.exists(self.data_file):
            try:
                with open(self.data_file, "r", encoding="utf-8") as f:
                    loaded": str(item.get("date")),
                                "temperature": float(item.get("temperature")),
                                "description": str(item.get("description")),
                                "precipitation": str(item.get("precipitation"))
                            }
                            datetime.strptime(valid_item["date"], "%Y-%m-%d") # Проверка формата даты
                            self.entries.append(valid_item)
                        except (ValueError, TypeError, KeyError):
                            messagebox.showwarning("Предупреждение", f"Пропущена некорректная запись при загрузке: {item}")
            except (json.JSONDecodeError, IOError):
                messagebox.showerror("Ошибка загрузки", f"Не удалось загрузить данные из файла {self.data_file}. Файл может быть поврежден.")
                self.entries = []
        else:
            self.entries = [] # Если файла нет, начинаем с пустого списка

    def apply_filter(self):
        filter_date = self.filter_date_entry.get().strip()
        filter_temp_str = self.filter_temp_entry.get().strip()

        filtered_entries = self.entries

        # Фильтр по дате
        if filter_date:
            try:
                datetime.strptime(filter_date, "%Y-%m-%d")
                filtered_entries = [entry for entry in filtered_entries if entry["date"] == filter_date]
            except ValueError:
                messagebox.showerror("Ошибка фильтра", "Неверный формат даты для фильтра. Используйте YYYY-MM-DD.")
                return

        # Фильтр по температуре
        if filter_temp_str:
            try:
                filter_temp = float(filter_temp_str)
                filtered_entries = [entry for entry in filtered_entries if entry["temperature"] > filter_temp]
            except ValueError:
                messagebox.showerror("Ошибка фильтра", "Температура для фильтра должна быть числом.")
                return

        self.update_treeview(filtered_entries)

    def reset_filter(self):
        self.filter_date_entry.delete(0, tk.END)
        self.filter_temp_entry.delete(0, tk.END)
        self.update_treeview() # Показать все записи

if __name__ == "__main__":
    root = tk.Tk()
    app = WeatherDiary(root)
    root.mainloop() -
Приложение помогает записывать и фильтровать записи о погоде
