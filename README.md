import tkinter as tk
from tkinter import ttk, messagebox
import json
from datetime import datetime
import os

class WeatherDiary:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Diary / Дневник погоды")
        self.root.geometry("900x600")
        
        # Хранилище записей
        self.records = []
        
        # Имя файла для сохранения
        self.filename = "weather_data.json"
        
        # Загрузка данных при запуске
        self.load_records()
        
        # Создание интерфейса
        self.create_widgets()
        
        # Обновление таблицы
        self.update_table()
    
    def create_widgets(self):
        # Рамка для ввода данных
        input_frame = ttk.LabelFrame(self.root, text="Добавить запись о погоде", padding="10")
        input_frame.grid(row=0, column=0, padx=10, pady=10, sticky="ew")
        
        # Поля ввода
        ttk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=0, sticky="w")
        self.date_entry = ttk.Entry(input_frame, width=20)
        self.date_entry.grid(row=0, column=1, padx=5, pady=5)
        
        ttk.Label(input_frame, text="Температура (°C):").grid(row=1, column=0, sticky="w")
        self.temp_entry = ttk.Entry(input_frame, width=20)
        self.temp_entry.grid(row=1, column=1, padx=5, pady=5)
        
        ttk.Label(input_frame, text="Описание погоды:").grid(row=2, column=0, sticky="w")
        self.desc_entry = ttk.Entry(input_frame, width=40)
        self.desc_entry.grid(row=2, column=1, padx=5, pady=5)
        
        ttk.Label(input_frame, text="Осадки:").grid(row=3, column=0, sticky="w")
        self.precip_var = tk.BooleanVar()
        self.precip_check = ttk.Checkbutton(input_frame, variable=self.precip_var)
        self.precip_check.grid(row=3, column=1, padx=5, pady=5, sticky="w")
        
        # Кнопка добавления
        add_button = ttk.Button(input_frame, text="Добавить запись", command=self.add_record)
        add_button.grid(row=4, column=1, pady=10, sticky="e")
        
        # Рамка для фильтрации
        filter_frame = ttk.LabelFrame(self.root, text="Фильтрация", padding="10")
        filter_frame.grid(row=1, column=0, padx=10, pady=10, sticky="ew")
        
        ttk.Label(filter_frame, text="Фильтр по дате:").grid(row=0, column=0, sticky="w")
        self.filter_date_entry = ttk.Entry(filter_frame, width=20)
        self.filter_date_entry.grid(row=0, column=1, padx=5, pady=5)
        
        ttk.Label(filter_frame, text="Температура выше:").grid(row=1, column=0, sticky="w")
        self.filter_temp_entry = ttk.Entry(filter_frame, width=20)
        self.filter_temp_entry.grid(row=1, column=1, padx=5, pady=5)
        
        filter_button = ttk.Button(filter_frame, text="Применить фильтры", command=self.apply_filters)
        filter_button.grid(row=2, column=1, pady=5, sticky="e")
        
        reset_button = ttk.Button(filter_frame, text="Сбросить фильтры", command=self.reset_filters)
        reset_button.grid(row=2, column=2, pady=5, padx=5)
        
        # Таблица для отображения записей
        table_frame = ttk.LabelFrame(self.root, text="Записи о погоде", padding="10")
        table_frame.grid(row=2, column=0, padx=10, pady=10, sticky="nsew")
        
        # Создание Treeview
        columns = ("Дата", "Температура", "Описание", "Осадки")
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)
        
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=150)
        
        self.tree.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
        
        scrollbar = ttk.Scrollbar(table_frame, orient=tk.VERTICAL, command=self.tree.yview)
        scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
        self.tree.configure(yscrollcommand=scrollbar.set)
        
        # Кнопки управления
        control_frame = ttk.Frame(self.root)
        control_frame.grid(row=3, column=0, pad
self.date - self Ресурсы и информация.
www.self.date
y=10)
        
        save_button = ttk.Button(control_frame, text="Сохранить в файл", command=self.save_records)
        save_button.pack(side=tk.LEFT, padx=5)
        
        load_button = ttk.Button(control_frame, text="Загрузить из файла", command=self.load_records)
        load_button.pack(side=tk.LEFT, padx=5)
        
        delete_button = ttk.Button(control_frame, text="Удалить запись", command=self.delete_record)
        delete_button.pack(side=tk.LEFT, padx=5)
        
        # Настройка растягивания
        self.root.grid_rowconfigure(2, weight=1)
        self.root.grid_columnconfigure(0, weight=1)
    
    def validate_input(self, date, temp, desc):
        """Проверка корректности ввода"""
        # Проверка формата даты
        try:
            datetime.strptime(date, "%Y-%m-%d")
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты! Используйте ГГГГ-ММ-ДД")
            return False
        
        # Проверка температуры
        try:
            float(temp)
        except ValueError:
            messagebox.showerror("Ошибка", "Температура должна быть числом!")
            return False
        
        # Проверка описания
        if not desc.strip():
            messagebox.showerror("Ошибка", "Описание погоды не может быть пустым!")
            return False
        
        return True
    
    def add_record(self):
        """Добавление новой записи"""
        date = self.date_entry.get()
        temp = self.temp_entry.get()
        desc = self.desc_entry.get()
        precip = "Да" if self.precip_var.get() else "Нет"
        
        if self.validate_input(date, temp, desc):
            record = {
                "date": date,
                "temperature": float(temp),
                "description": desc.strip(),
                "precipitation": precip
            }
            
            self.records.append(record)
            self.update_table()
            self.clear_input()
            messagebox.showinfo("Успех", "Запись добавлена!")
    
    def delete_record(self):
        """Удаление выбранной записи"""
        selected = self.tree.selection()
        if selected:
            item = self.tree.item(selected[0])
            values = item['values']
            
            for i, record in enumerate(self.records):
                if (record['date'] == values[0] and 
                    str(record['temperature']) == values[1] and 
                    record['description'] == values[2]):
                    del self.records[i]
                    break
            
            self.update_table()
            messagebox.showinfo("Успех", "Запись удалена!")
        else:
            messagebox.showwarning("Внимание", "Выберите запись для удаления!")
    
    def apply_filters(self):
        """Применение фильтров к записям"""
        filter_date = self.filter_date_entry.get().strip()
        filter_temp = self.filter_temp_entry.get().strip()
        
        filtered_records = self.records.copy()
        
        if filter_date:
            filtered_records = [r for r in filtered_records if r['date'] == filter_date]
        
        if filter_temp:
            try:
                temp_threshold = float(filter_temp)
                filtered_records = [r for r in filtered_records if r['temperature'] > temp_threshold]
            except ValueError:
                messagebox.showerror("Ошибка", "Введите корректное число для фильтра температуры!")
                return
        
        self.update_table(filtered_records)
    
    def reset_filters(self):
        """Сброс фильтров"""
        self.filter_date_entry.delete(0, tk.END)
        self.filter_temp_entry.delete(0, tk.END)
        self.update_table()
    
    def save_records(self):
        """Сохранение записей в JSON файл"""
        try:
            with open(self.filename, 'w', encoding='utf-8') as file:
                json.dump(self.records, file, ensure_ascii=False, indent=4)
            messagebox.showin


fo("Успех", f"Данные сохранены в {self.filename}")
        except Exception as e:
            messagebox.showerror("Ошибка", f"Не удалось сохранить файл: {str(e)}")
    
    def load_records(self):
        """Загрузка записей из JSON файла"""
        if os.path.exists(self.filename):
            try:
                with open(self.filename, 'r', encoding='utf-8') as file:
                    self.records = json.load(file)
                messagebox.showinfo("Загрузка", f"Загружено {len(self.records)} записей")
            except Exception as e:
                messagebox.showerror("Ошибка", f"Не удалось загрузить файл: {str(e)}")
                self.records = []
        else:
            self.records = []
        
        if hasattr(self, 'tree'):
            self.update_table()
    
    def update_table(self, records=None):
        """Обновление таблицы"""
        if records is None:
            records = self.records
        
        # Очистка таблицы
        for item in self.tree.get_children():
            self.tree.delete(item)
        
        # Заполнение данными
        for record in records:
            self.tree.insert("", tk.END, values=(
                record['date'],
                record['temperature'],
                record['description'],
                record.get('precipitation', 'Нет')
            ))
    
    def clear_input(self):
        """Очистка полей ввода"""
        self.date_entry.delete(0, tk.END)
        self.temp_entry.delete(0, tk.END)
        self.desc_entry.delete(0, tk.END)
        self.precip_var.set(False)

def main():
    root = tk.Tk()
    app = WeatherDiary(root)
    root.mainloop()

if __name__ == "__main__":
    main()
```

2. Файл .gitignore

```gitignore
# Python
__pycache__/
*.py[cod]
*.so
*.egg
*.egg-info/
dist/
build/
*.spec

# Virtual environments
venv/
env/
ENV/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Project specific
weather_data.json
*.log
```

3. README.md

```markdown
# Weather Diary / Дневник погоды

## Автор
[Ваше Имя и Фамилия]

## Краткое описание
GUI-приложение для ведения дневника погоды. Позволяет добавлять, просматривать, фильтровать и сохранять записи о погодных условиях. Данные сохраняются в формате JSON.

## Функциональность
- Добавление записей о погоде (дата, температура, описание, осадки)
- Просмотр всех записей в табличном виде
- Фильтрация по дате и температуре
- Сохранение данных в JSON файл
- Загрузка данных из JSON файла
- Валидация вводимых данных
- Удаление записей

## Как запустить приложение

### Требования
- Python 3.6 или выше
- tkinter (обычно входит в стандартную установку Python)

### Запуск
1. Клонируйте репозиторий:
```bash
git clone https://github.com/yourusername/weather-diary.git
cd weather-diary
```

1. Запустите приложение:

```bash
python weather_diary.py
```

Примеры использования

Добавление записи

1. Введите дату в формате ГГГГ-ММ-ДД (например: 2024-01-15)
2. Введите температуру (например: -5.5)
3. Введите описание погоды (например: "Солнечно, небольшой ветер")
4. Отметьте наличие осадков
5. Нажмите "Добавить запись"

Фильтрация

· По дате: введите дату в поле "Фильтр по дате" и нажмите "Применить фильтры"
· По температуре: введите пороговую температуру в поле "Температура выше" для отображения записей с температурой выше указанной

Сохранение и загрузка

· Сохранить: нажмите "Сохранить в файл" для сохранения текущих записей
· Загрузить: нажмите "Загрузить из файла" для загрузки ранее сохраненных записей

Тестирование

Тест 1: Добавление записи

· Ввод: дата="2024-01-15", температура="-5.5", описание="Снег", осадки=Да
· Ожидаемый результат: Запись добавлена в таблицу

Тест 2: Фильтрация по дате

· Ввод: фильтр по дате="2024-01-15"
· Ожидаемый результат: Отображается только запись за 2024-01-15

Тест 3: Фильтрация по температуре

· Ввод: температура выше="0"
· Ожидаемый результат: Отображаются только записи с положительной температурой

Тест 4: Валидация

· Ввод: некорректная дата "15/01/2024"
· Ожидаемый результат: Сообщение об ошибке формата даты

Лицензия

MIT License

```

## Установка и запуск

1. Сохраните все файлы в структуру проекта как показано выше
2. Убедитесь, что у вас установлен Python 3.x
3. Запустите приложение командой:
```bash
python weather_diary.py
```

Использование Git

```bash
# Инициализация репозитория
git init

# Добавление файлов
git add .

# Первый коммит
git commit -m "Initial commit: Weather Diary application"

# Создание репозитория на GitHub и связывание
git remote add origin https://github.com/yourusername/weather-diary.git
git branch -M main
git push -u origin main
```
