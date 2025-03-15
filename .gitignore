# RusLotto

RusLotto — это скрипт для поиска лотерейных билетов на сайте Столото. Он позволяет пользователю проверять, совпадают ли номера их билетов с выигрышными номерами тиражей. Скрипт использует компонент Selenium для автоматизации поиска и позволяет останавливать или приостанавливать поиск.

Требования:
Для запуска скрипта необходимо установить несколько библиотек. Убедитесь, что у вас установлен Python 3, и выполните следующие команды в терминале или командной строке:
`pip install selenium`
`pip install webdriver-manager`
`pip install fake-useragent`

Script:
# RusLotto.py - Скрипт для поиска билетов в лотерее РусЛото на сайте Stoloto
```
import tkinter as tk
from tkinter import messagebox, ttk
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from webdriver_manager.chrome import ChromeDriverManager
from fake_useragent import UserAgent
import threading
import time
import logging

# Настройка логирования
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s")

DEFAULT_TICKETS = [
    "999272100", "99927199", "999270",
    "9992722", "99927214", "99927215"
]

class TicketCheckerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Поиск билетов РусЛото")
        self.root.geometry("700x600")
        
        self.is_paused = threading.Event()
        self.is_stopped = threading.Event()
        self.driver = None
        self.total_checked = 0
        self.total_found = 0
        self.current_draw = ""
        self.start_time = None
        
        # Ввод номера тиража
        tk.Label(root, text="Введите номер тиража:").pack()
        self.draw_entry = tk.Entry(root, width=40)
        self.draw_entry.pack()
        self.draw_entry.insert(0, "1623")
        
        # Ввод номеров билетов
        frame = tk.Frame(root)
        frame.pack()
        self.entries = []
        
        for value in DEFAULT_TICKETS:
            entry = tk.Entry(frame, width=20)
            entry.pack()
            entry.insert(0, value)
            self.entries.append(entry)
        
        # Кнопки для старта и паузы
        self.start_button = tk.Button(root, text="Старт", command=self.start_search)
        self.start_button.pack()
        self.pause_button = tk.Button(root, text="Пауза", command=self.pause_search, state=tk.DISABLED)
        self.pause_button.pack()
        
        # Таблица для отображения результатов поиска
        tk.Label(root, text="Результаты поиска:").pack()
        self.result_table = ttk.Treeview(root, columns=("Билет", "Тираж"), show="headings")
        self.result_table.heading("Билет", text="Номер билета")
        self.result_table.heading("Тираж", text="Тираж")
        self.result_table.pack()
        
        # Статистика поиска
        self.stats_label = tk.Label(root, text="Проверено билетов: 0 | Найдено: 0 | Время: 0 мин")
        self.stats_label.pack()
        
        # Обновление статистики каждую секунду
        self.root.after(1000, self.update_stats)
    
    def validate_input(self):
        """Проверка правильности введённых данных."""
        search_values = [entry.get().strip() for entry in self.entries if entry.get().strip()]
        if not search_values:
            messagebox.showerror("Ошибка", "Введите корректные данные!")
            return None
        return search_values
    
    def pause_search(self):
        """Пауза/возобновление поиска."""
        if self.is_paused.is_set():
            self.is_paused.clear()
            self.pause_button.config(text="Пауза")
            self.disable_entries()  # Заблокировать поля ввода
        else:
            self.is_paused.set()
            self.pause_button.config(text="Возобновить")
            self.enable_entries()  # Разблокировать поля ввода
    
    def stop_search(self):
        """Остановить поиск."""
        self.is_stopped.set()
        self.is_paused.clear()
        if self.driver:
            self.driver.quit()
    
    def start_search(self):
        """Запуск поиска билетов."""
        search_values = self.validate_input()
        if not search_values:
            return
        
        self.is_stopped.clear()
        self.is_paused.clear()
        self.total_checked = 0
        self.total_found = 0
        self.start_time = time.time()
        self.current_draw = self.draw_entry.get().strip()
        
        self.start_button.config(state=tk.DISABLED)
        self.pause_button.config(state=tk.NORMAL)
        self.disable_entries()  # Заблокировать поля ввода
        
        threading.Thread(target=self.search_process, args=(search_values,), daemon=True).start()
    
    def search_process(self, search_values):
        """Основной процесс поиска билетов."""
        options = Options()
        options.add_argument(f"user-agent={UserAgent().random}")
        options.add_argument("--disable-gpu")
        options.add_argument("--disable-software-rasterizer")
        options.add_argument("--disable-webgl")
        options.add_argument("--disable-usb")
               
        prefs = {
            "profile.default_content_setting_values.notifications": 2,
            "profile.managed_default_content_settings.images": 2,
            "profile.default_content_settings.popups": 0,
        }
        options.add_experimental_option("prefs", prefs)
        
        try:
            self.driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=options)
            self.driver.get(f"https://www.stoloto.ru/ruslotto/game?from=bingo&draw={self.current_draw}")
            found_tickets = set()
            
            while not self.is_stopped.is_set():
                if self.is_paused.is_set():
                    time.sleep(0.5)
                    continue
                
                try:
                    tickets = WebDriverWait(self.driver, 5).until(
                        EC.presence_of_all_elements_located((By.CSS_SELECTOR, "span[data-test-id='ticket-number']"))
                    )
                except:
                    tickets = []
                
                for ticket in tickets:
                    try:
                        ticket_text = ticket.text.strip()
                        if ticket_text:
                            self.total_checked += 1
                            for value in search_values:
                                if value in ticket_text and value not in found_tickets:
                                    found_tickets.add(value)
                                    self.total_found += 1
                                    self.root.after(0, lambda txt=ticket_text: self.add_result(txt))
                                    
                                    self.is_paused.set()
                                    self.pause_button.config(text="Возобновить")
                                    self.enable_entries()
                                    
                                    messagebox.showinfo("Билет найден!", f"Найден билет: {ticket_text}\n\nПоиск приостановлен.")
                                    self.disable_entries()  # Заблокировать поля ввода перед возобновлением
                                    self.is_paused.clear()
                                    self.pause_button.config(text="Пауза")
                                    break
                    except Exception as e:
                        logging.error(f"Ошибка при обработке билета: {e}")
                        continue
                
                time.sleep(1.5)
                self.driver.refresh()
        except Exception as e:
            logging.error(f"Ошибка при поиске: {e}")
        finally:
            self.stop_search()
    
    def add_result(self, ticket_text):
        """Добавление найденного билета в таблицу и подсветка."""
        self.result_table.insert("", "end", values=(ticket_text, self.current_draw), tags=("highlight",))
        self.highlight_ticket(ticket_text)
    
    def highlight_ticket(self, ticket_text):
        """Подсветка найденного билета в таблице."""
        for item in self.result_table.get_children():
            if self.result_table.item(item, "values")[0] == ticket_text:
                self.result_table.item(item, tags=("highlight",))
                break
    
    def update_stats(self):
        """Обновление статистики поиска."""
        elapsed_time = round((time.time() - self.start_time) / 60, 1) if self.start_time else 0
        self.stats_label.config(text=f"Проверено билетов: {self.total_checked} | Найдено: {self.total_found} | Время: {elapsed_time} мин")
        self.root.after(1000, self.update_stats)
    
    def disable_entries(self):
        """Заблокировать поля ввода билетов во время поиска."""
        for entry in self.entries:
            entry.config(state=tk.DISABLED)
    
    def enable_entries(self):
        """Разблокировать поля ввода билетов только на паузе."""
        if self.is_paused.is_set():
            for entry in self.entries:
                entry.config(state=tk.NORMAL)

if __name__ == "__main__":
    # Создание и настройка главного окна
    root = tk.Tk()
    
    # Подсветка найденных билетов
    style = ttk.Style(root)
    style.configure("highlight", background="yellow", foreground="black")
    
    # Запуск приложения
    app = TicketCheckerApp(root)
    root.mainloop()
```

# Запуск:
Клонируйте репозиторий на свой компьютер или загрузите скрипт RusLotto.py.
