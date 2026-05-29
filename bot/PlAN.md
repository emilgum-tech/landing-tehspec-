# 📱 Telegram Бот Салона "Дела салона" - План разработки

## 🎯 Обзор проекта

**Название:** Дела салона  
**Стек:** Python + aiogram + Google Sheets + VPS Beget  
**Пользователи в месяц:** ~10  
**Мастеров:** 3  
**Услуги:** Маникюр, Волосы  
**График работы:** Пн-Пт, 9:00-18:00  
**Длительность услуги:** 1 час  
**Одновременно клиентов:** 3  

---

## 📋 Требования

### ✅ Функциональность
- [x] Запись на услугу только через бот
- [x] 3 мастера (выбор при бронировании)
- [x] Уведомления клиентам после записи
- [x] История заказов для вернувшихся клиентов
- [x] Уведомления админу при новой записи
- [x] Выгрузка клиентов в Google Sheets

### ✅ Сбор данных
- Имя
- Телефон
- Ник в Telegram

### ✅ Администрирование
- 1 админ управляет ботом
- Уведомления в Telegram при новой записи
- Просмотр всех заказов в Google Sheets

---

## 🎯 Разбиение на задачи (20-40 мин каждая)

### Блок 1: Setup & Конфигурация (1-3 задачи)

#### ✅ Задача 1.1: Инициализация проекта (20 мин)

**Что делать:**
- Создать директорию проекта
- Инициализировать git
- Создать `requirements.txt` с зависимостями
- Создать `.env.example`
- Создать структуру папок

**Структура папок:**
```
salon-bot/
├── handlers/
├── sheets/
├── utils/
├── models/
├── main.py
├── config.py
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

**Зависимости (requirements.txt):**
```
aiogram==3.x
gspread==5.x
python-dotenv
google-auth-oauthlib
google-auth-httplib2
requests
```

**Проверка:**
```bash
ls -la                    # видны все папки
cat requirements.txt      # все зависимости на месте
git log                   # git инициализирован
```

**Файлы для создания:**
- `requirements.txt`
- `.env.example`
- `.gitignore`

---

#### ✅ Задача 1.2: Создать config.py (25 мин)

**Что делать:**
- Прочитать переменные окружения (.env)
- Определить константы для бота
- Константы для услуг, мастеров, графика работы
- Настроить логирование

**Содержание config.py:**
```python
import os
import logging
from dotenv import load_dotenv

load_dotenv()

# Telegram
TELEGRAM_TOKEN = os.getenv('TELEGRAM_TOKEN')
ADMIN_ID = int(os.getenv('ADMIN_ID'))

# Google Sheets
GOOGLE_SHEETS_ID = os.getenv('GOOGLE_SHEETS_ID')
GOOGLE_SERVICE_ACCOUNT_FILE = os.getenv('GOOGLE_SERVICE_ACCOUNT_FILE', 'credentials.json')

# Услуги
SERVICES = {
    'manicure': {
        'name': 'Маникюр',
        'price': 1000,
        'duration': 60  # минуты
    },
    'hair': {
        'name': 'Волосы',
        'price': 2000,
        'duration': 60
    }
}

# Мастера
MASTERS = ['Мастер 1', 'Мастер 2', 'Мастер 3']

# График работы
WORKING_HOURS = {
    'start': 9,
    'end': 18
}

WORKING_DAYS = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт']  # Пн-Пт
MAX_CLIENTS_SIMULTANEOUSLY = 3

# Логирование
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)
```

**Проверка:**
```python
from config import SERVICES, MASTERS, TELEGRAM_TOKEN
print(SERVICES['manicure']['name'])  # Маникюр
print(len(MASTERS))                   # 3
print(TELEGRAM_TOKEN is not None)    # True
```

**Файлы:**
- `config.py`
- `.env.example` (обновить)

---

#### ✅ Задача 1.3: Настроить Google Sheets API (30 мин)

**Что делать:**
- Создать Google Cloud проект
- Включить Google Sheets API
- Создать Service Account
- Скачать JSON ключ
- Создать Google Sheets документ "Дела салона"
- Настроить переменные окружения

**Пошаговые инструкции:**
1. Перейти на https://console.cloud.google.com/
2. Создать новый проект: "Salon Bot"
3. Включить Google Sheets API
4. Создать Service Account
5. Скачать JSON ключ
6. Поместить путь к файлу в `.env`: `GOOGLE_SERVICE_ACCOUNT_FILE=./credentials.json`
7. Создать Google Sheets с названием "Дела салона"
8. Поместить ID таблицы в `.env`: `GOOGLE_SHEETS_ID=xxx`

**Проверка подключения:**
```python
import gspread
from google.oauth2.service_account import Credentials

SCOPES = ['https://www.googleapis.com/auth/spreadsheets']
credentials = Credentials.from_service_account_file(
    'credentials.json',
    scopes=SCOPES
)
client = gspread.authorize(credentials)
spreadsheet = client.open_by_key(GOOGLE_SHEETS_ID)
print(f"✅ Подключено к таблице: {spreadsheet.title}")
```

**Файлы:**
- `credentials.json` (в .gitignore!)
- `.env` (обновить)

---

### Блок 2: Google Sheets структура (4-5 задачи)

#### ✅ Задача 2.1: Создать структуру листов в Google Sheets (20 мин)

**Что делать:**
- Создать 3 листа в Google Sheets
- Добавить заголовки в каждый лист
- Добавить тестовые данные

**Лист "Услуги":**
```
| ID        | Название  | Стоимость | Длительность (мин) |
|-----------|-----------|-----------|-------------------|
| manicure  | Маникюр   | 1000      | 60                |
| hair      | Волосы    | 2000      | 60                |
```

**Лист "Расписание":**
```
| Дата       | Время | Мастер 1 | Мастер 2 | Мастер 3 |
|-----------|-------|----------|----------|----------|
| 10.06.2026 | 09:00 | свободно | свободно | свободно |
| 10.06.2026 | 10:00 | занято   | свободно | свободно |
```

**Лист "Заказы":**
```
| ID | Дата | Время | Мастер  | Услуга  | Имя  | Телефон | Telegram | Статус | Дата создания |
|----|------|-------|---------|---------|------|---------|----------|--------|---------------|
| 1  | 15.06.2026 | 10:00 | Мастер 1 | Маникюр | Анна | +79991234567 | @anna | Подтверждён | 29.05.2026 |
```

**Проверка в Google Sheets:**
- Открыть таблицу
- Должны быть 3 листа
- В каждом листе должны быть заголовки
- Должны быть тестовые данные

---

#### ✅ Задача 2.2: Создать sheets/google_sheets.py - чтение услуг (25 мин)

**Что делать:**
- Функция подключения к Google Sheets
- Функция `get_services()` - получить все услуги
- Функция `get_service_by_id(service_id)` - получить одну услугу

**Содержание sheets/google_sheets.py:**
```python
import gspread
from google.oauth2.service_account import Credentials
from config import GOOGLE_SERVICE_ACCOUNT_FILE, GOOGLE_SHEETS_ID, logger

SCOPES = ['https://www.googleapis.com/auth/spreadsheets']

def get_client():
    """Подключиться к Google Sheets"""
    try:
        credentials = Credentials.from_service_account_file(
            GOOGLE_SERVICE_ACCOUNT_FILE,
            scopes=SCOPES
        )
        return gspread.authorize(credentials)
    except Exception as e:
        logger.error(f"Ошибка подключения к Google Sheets: {e}")
        raise

def get_spreadsheet():
    """Получить основную таблицу"""
    client = get_client()
    return client.open_by_key(GOOGLE_SHEETS_ID)

def get_services():
    """Получить все услуги из листа 'Услуги'"""
    try:
        spreadsheet = get_spreadsheet()
        worksheet = spreadsheet.worksheet('Услуги')
        rows = worksheet.get_all_records()
        
        services = {}
        for row in rows:
            services[row['ID']] = {
                'name': row['Название'],
                'price': int(row['Стоимость']),
                'duration': int(row['Длительность (мин)'])
            }
        return services
    except Exception as e:
        logger.error(f"Ошибка получения услуг: {e}")
        return {}

def get_service_by_id(service_id):
    """Получить одну услугу по ID"""
    services = get_services()
    return services.get(service_id)
```

**Проверка:**
```python
from sheets.google_sheets import get_services, get_service_by_id

services = get_services()
print(services)
# Должно вывести:
# {
#   'manicure': {'name': 'Маникюр', 'price': 1000, 'duration': 60},
#   'hair': {'name': 'Волосы', 'price': 2000, 'duration': 60}
# }

service = get_service_by_id('manicure')
print(service['name'])  # Маникюр
```

**Файлы:**
- `sheets/__init__.py` (создать пустой)
- `sheets/google_sheets.py`

---

#### ✅ Задача 2.3: Функции для работы с расписанием (30 мин)

**Что делать:**
- Функция `get_available_slots(date)` - получить свободные слоты на дату
- Функция `get_available_masters(date, time)` - получить свободных мастеров
- Функция `is_slot_available(date, time, master)` - проверить слот

**Добавить в sheets/google_sheets.py:**
```python
from datetime import datetime, timedelta
from config import MASTERS, WORKING_HOURS

def get_all_slots_for_date(date):
    """Получить все возможные слоты на дату (09:00-18:00)"""
    slots = []
    for hour in range(WORKING_HOURS['start'], WORKING_HOURS['end']):
        slots.append(f"{hour:02d}:00")
    return slots

def get_available_slots(date_str, master):
    """Получить свободные слоты на дату для конкретного мастера
    
    Args:
        date_str: дата в формате '%d.%m.%Y'
        master: имя мастера (например 'Мастер 1')
    
    Returns:
        список свободных слотов для этого мастера
    """
    try:
        spreadsheet = get_spreadsheet()
        worksheet = spreadsheet.worksheet('Расписание')
        rows = worksheet.get_all_records()
        
        all_slots = get_all_slots_for_date(date_str)
        booked_slots = set()
        
        for row in rows:
            if row['Дата'] == date_str:
                # Проверить только столбец выбранного мастера (не все мастера!)
                if row.get(master) == 'занято':
                    booked_slots.add(row['Время'])
        
        available = [s for s in all_slots if s not in booked_slots]
        return available
    except Exception as e:
        logger.error(f"Ошибка получения свободных слотов для {master}: {e}")
        return []

def get_available_masters(date_str, time_str):
    """Получить свободных мастеров на дату и время"""
    try:
        spreadsheet = get_spreadsheet()
        worksheet = spreadsheet.worksheet('Расписание')
        rows = worksheet.get_all_records()
        
        available = []
        for row in rows:
            if row['Дата'] == date_str and row['Время'] == time_str:
                for master in MASTERS:
                    if row.get(master) != 'занято':
                        available.append(master)
                break
        
        return available if available else MASTERS  # Если строки нет, все свободны
    except Exception as e:
        logger.error(f"Ошибка получения свободных мастеров: {e}")
        return MASTERS

def is_slot_available(date_str, time_str, master):
    """Проверить доступность слота для конкретного мастера"""
    slots = get_available_slots(date_str, master)
    return time_str in slots
```

**Проверка:**
```python
from sheets.google_sheets import get_available_slots, get_available_masters

# Теперь передаём мастера!
slots = get_available_slots('2026-06-10', 'Мастер 1')
print(slots)  # ['09:00', '11:00', '13:00', ...] - только свободное для Мастера 1

masters = get_available_masters('2026-06-10', '10:00')
print(masters)  # ['Мастер 1', 'Мастер 3']

available = is_slot_available('2026-06-10', '10:00', 'Мастер 1')
print(available)  # True или False
```

**Файлы:**
- `sheets/google_sheets.py` (добавить функции)

---

#### ✅ Задача 2.4: Функция для сохранения заказа (25 мин)

**Что делать:**
- Функция `save_booking(booking_data)` - записать заказ в Google Sheets
- Функция `mark_slot_as_busy(date, time, master)` - пометить слот как занятый
- Функция `get_next_booking_id()` - сгенерировать ID заказа

**Добавить в sheets/google_sheets.py:**
```python
from datetime import datetime

def get_next_booking_id():
    """Получить следующий ID заказа"""
    try:
        spreadsheet = get_spreadsheet()
        worksheet = spreadsheet.worksheet('Заказы')
        rows = worksheet.get_all_records()
        
        if not rows:
            return '001'
        
        last_id = int(rows[-1]['ID'])
        return str(last_id + 1).zfill(3)
    except Exception as e:
        logger.error(f"Ошибка получения ID заказа: {e}")
        return '001'

def save_booking(booking_data):
    """Сохранить заказ в Google Sheets
    
    booking_data = {
        'date': '2026-06-15',
        'time': '10:00',
        'master': 'Мастер 1',
        'service': 'manicure',
        'name': 'Анна',
        'phone': '+79991234567',
        'telegram': '@anna_user'
    }
    """
    try:
        booking_id = get_next_booking_id()
        
        spreadsheet = get_spreadsheet()
        worksheet = spreadsheet.worksheet('Заказы')
        
        # Получить имя услуги
        service = get_service_by_id(booking_data['service'])
        service_name = service['name'] if service else booking_data['service']
        
        # Добавить новую строку
        worksheet.append_row([
            booking_id,
            booking_data['date'],
            booking_data['time'],
            booking_data['master'],
            service_name,
            booking_data['name'],
            booking_data['phone'],
            booking_data['telegram'],
            'Подтверждён',  # Статус
            datetime.now().strftime('%d.%m.%Y')  # Дата создания
        ])
        
        # Пометить слот как занятый
        mark_slot_as_busy(booking_data['date'], booking_data['time'], booking_data['master'])
        
        logger.info(f"Заказ {booking_id} сохранён")
        return booking_id
    except Exception as e:
        logger.error(f"Ошибка сохранения заказа: {e}")
        return None

def mark_slot_as_busy(date_str, time_str, master):
    """Пометить слот как занятый
    
    Важно: использует MASTERS.index() для определения столбца, 
    поэтому не зависит от значения ячейки ('занято'/'свободно')
    """
    try:
        spreadsheet = get_spreadsheet()
        worksheet = spreadsheet.worksheet('Расписание')
        rows = worksheet.get_all_records()
        
        # Определить номер столбца мастера
        # Колонки: 1-Дата, 2-Время, 3-Мастер1, 4-Мастер2, 5-Мастер3
        master_col = MASTERS.index(master) + 3
        
        # Найти строку с нужной датой и временем
        for idx, row in enumerate(rows):
            if row['Дата'] == date_str and row['Время'] == time_str:
                row_idx = idx + 2  # +1 за заголовок, +1 за индексацию с 1
                worksheet.update_cell(row_idx, master_col, 'занято')
                logger.info(f"Слот отмечен: {date_str} {time_str} {master}")
                return True
        
        # Если строки нет, создать новую
        new_row = [date_str, time_str] + ['свободно'] * len(MASTERS)
        worksheet.append_row(new_row)
        
        # Обновить свежий список и найти созданную строку
        rows = worksheet.get_all_records()
        for idx, row in enumerate(rows):
            if row['Дата'] == date_str and row['Время'] == time_str:
                row_idx = idx + 2
                worksheet.update_cell(row_idx, master_col, 'занято')
                logger.info(f"Создан новый слот и отмечен: {date_str} {time_str} {master}")
                return True
        
        logger.warning(f"Не удалось отметить слот: {date_str} {time_str} {master}")
        return False
    except Exception as e:
        logger.error(f"Ошибка отметить слот как занятый: {e}")
        return False
```

**Проверка:**
```python
from sheets.google_sheets import save_booking, get_next_booking_id

booking = {
    'date': '2026-06-15',
    'time': '10:00',
    'master': 'Мастер 1',
    'service': 'manicure',
    'name': 'Анна',
    'phone': '+79991234567',
    'telegram': '@anna_user'
}

booking_id = save_booking(booking)
print(f"Заказ создан с ID: {booking_id}")  # Заказ создан с ID: 001

# Проверить в Google Sheets - должна быть новая строка в "Заказы"
# Слот должен быть отмечен как "занято" в "Расписание"
```

**Файлы:**
- `sheets/google_sheets.py` (добавить функции)

---

### Блок 3: Базовый Telegram бот (6-8 задачи)

#### ✅ Задача 3.1: Создать основной файл бота (20 мин)

**Что делать:**
- Создать `main.py` с инициализацией бота и диспетчера
- Подключить logger
- Написать функцию `main()` для запуска

**Содержание main.py:**
```python
import asyncio
import logging
from aiogram import Bot, Dispatcher
from config import TELEGRAM_TOKEN, logger

# Инициализация
bot = Bot(token=TELEGRAM_TOKEN)
dp = Dispatcher()

async def main():
    """Главная функция запуска бота"""
    logger.info("🤖 Бот запущен")
    
    try:
        # Удалить все предыдущие webhook'и (если были)
        await bot.delete_webhook(drop_pending_updates=True)
        
        # Запустить polling
        await dp.start_polling(bot)
    except Exception as e:
        logger.error(f"Ошибка при запуске бота: {e}")
    finally:
        await bot.session.close()

if __name__ == '__main__':
    logging.basicConfig(level=logging.INFO)
    asyncio.run(main())
```

**Проверка:**
```bash
python main.py
# Должен вывести: 🤖 Бот запущен
# Бот должен быть в сети (проверить отправить /start в Telegram)
```

**Файлы:**
- `main.py`

---

#### ✅ Задача 3.2: Создать обработчик /start (25 мин)

**Что делать:**
- Создать `handlers/start.py`
- Обработчик команды `/start`
- Главное меню с 4 кнопками

**Содержание handlers/start.py:**
```python
from aiogram import Router, types
from aiogram.filters import CommandStart
from aiogram.utils.keyboard import ReplyKeyboardBuilder
from config import logger

router = Router()

@router.message(CommandStart())
async def start_handler(message: types.Message):
    """Обработчик команды /start"""
    logger.info(f"Пользователь {message.from_user.id} запустил бот")
    
    # Создать клавиатуру
    kb = ReplyKeyboardBuilder()
    kb.add(
        types.KeyboardButton(text="📅 Запись"),
        types.KeyboardButton(text="📋 Мои записи")
    )
    kb.add(
        types.KeyboardButton(text="ℹ️ Контакты"),
        types.KeyboardButton(text="❓ FAQ")
    )
    kb.adjust(2)
    
    await message.answer(
        "👋 Добро пожаловать в салон красоты <b>Дела салона</b>!\n\n"
        "Что вы хотите сделать?",
        reply_markup=kb.as_markup(resize_keyboard=True)
    )
```

**В main.py добавить:**
```python
from handlers import start

# После инициализации dp
dp.include_router(start.router)
```

**Проверка:**
```
Написать /start в Telegram
Должно быть сообщение: "👋 Добро пожаловать в салон красоты Дела салона!"
Должны быть 4 кнопки в правильном макете
```

**Файлы:**
- `handlers/__init__.py` (создать пустой)
- `handlers/start.py`
- обновить `main.py`

---

#### ✅ Задача 3.3: Подключить FSM для управления состояниями (25 мин)

**Что делать:**
- Создать `models/booking_state.py` с FSM состояниями
- Определить все состояния записи
- Подключить FSM в main.py

**Содержание models/booking_state.py:**
```python
from aiogram.fsm.state import State, StatesGroup

class BookingStates(StatesGroup):
    """Состояния процесса бронирования"""
    choosing_service = State()         # Выбор услуги
    choosing_master = State()          # Выбор мастера
    choosing_date = State()            # Выбор даты
    choosing_time = State()            # Выбор времени
    choosing_name = State()            # Ввод имени
    choosing_phone = State()           # Ввод телефона
    choosing_telegram = State()        # Ввод Telegram ника
    confirming_booking = State()       # Подтверждение
```

**В main.py добавить:**
```python
from aiogram.fsm.storage.memory import MemoryStorage

# После Bot и Dispatcher
storage = MemoryStorage()
dp = Dispatcher(storage=storage)
```

**Проверка:**
```python
from models.booking_state import BookingStates

print(BookingStates.choosing_service)
print(BookingStates.choosing_master)
# Все состояния должны быть доступны без ошибок
```

**Файлы:**
- `models/__init__.py` (создать пустой)
- `models/booking_state.py`
- обновить `main.py`

---

#### ✅ Задача 3.4: Обработчик "Запись" - выбор услуги (25 мин)

**Что делать:**
- Создать `handlers/booking.py`
- Обработчик нажатия кнопки "📅 Запись"
- Показать список услуг
- Перейти в состояние `choosing_service`

**Содержание handlers/booking.py:**
```python
from aiogram import Router, types, F
from aiogram.fsm.context import FSMContext
from aiogram.utils.keyboard import InlineKeyboardBuilder, ReplyKeyboardBuilder
from models.booking_state import BookingStates
from sheets.google_sheets import get_services
from config import logger

router = Router()

@router.message(F.text == "📅 Запись")
async def booking_start(message: types.Message, state: FSMContext):
    """Начало процесса записи"""
    logger.info(f"Пользователь {message.from_user.id} начал запись")
    
    # Установить состояние
    await state.set_state(BookingStates.choosing_service)
    
    # Получить услуги
    services = get_services()
    
    # Создать клавиатуру с услугами
    kb = InlineKeyboardBuilder()
    for service_id, service_data in services.items():
        kb.add(
            types.InlineKeyboardButton(
                text=f"{service_data['name']} ({service_data['price']}₽)",
                callback_data=f"service_{service_id}"
            )
        )
    kb.adjust(1)
    
    await message.answer(
        "Выберите услугу:",
        reply_markup=kb.as_markup()
    )

@router.callback_query(BookingStates.choosing_service, F.data.startswith("service_"))
async def process_service_choice(callback: types.CallbackQuery, state: FSMContext):
    """Обработка выбора услуги"""
    service_id = callback.data.split("_", 1)[1]
    
    # Сохранить выбор в контексте
    await state.update_data(service_id=service_id)
    
    service = get_services().get(service_id)
    logger.info(f"Пользователь {callback.from_user.id} выбрал услугу: {service['name']}")
    
    # Перейти к выбору мастера
    await state.set_state(BookingStates.choosing_master)
    
    await callback.message.edit_text(
        f"✅ Выбрана услуга: <b>{service['name']}</b> ({service['price']}₽)\n\n"
        "Выбирите мастера:"
    )
    
    await callback.answer()
```

**В main.py добавить:**
```python
from handlers import booking

dp.include_router(booking.router)
```

**Проверка:**
```
Нажать "📅 Запись"
Должны появиться кнопки: "Маникюр (1000₽)" и "Волосы (2000₽)"
Нажать "Маникюр"
Должно перейти к выбору мастера
```

**Файлы:**
- `handlers/booking.py`
- обновить `main.py`

---

#### ✅ Задача 3.5: Выбор мастера (25 мин)

**Что делать:**
- Показать список мастеров после выбора услуги
- Перейти в состояние `choosing_master`
- Сохранить выбор мастера

**Добавить в handlers/booking.py:**
```python
from config import MASTERS

@router.callback_query(BookingStates.choosing_master, F.data.startswith("master_"))
async def process_master_choice(callback: types.CallbackQuery, state: FSMContext):
    """Обработка выбора мастера и показ выбора даты"""
    master = callback.data.split("_", 1)[1]
    
    # Сохранить выбор
    await state.update_data(master=master)
    
    logger.info(f"Пользователь {callback.from_user.id} выбрал мастера: {master}")
    
    # Перейти к выбору даты
    await state.set_state(BookingStates.choosing_date)
    
    # Сразу показать кнопки дат (не ждать message от пользователя!)
    working_days = get_next_working_days(7)
    kb = InlineKeyboardBuilder()
    for day in working_days:
        date_str = day.strftime('%d.%m.%Y')
        day_name = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'][day.weekday()]
        kb.add(
            types.InlineKeyboardButton(
                text=f"{day_name} {date_str}",
                callback_data=f"date_{date_str}"
            )
        )
    kb.adjust(1)
    
    await callback.message.edit_text(
        f"✅ Выбран мастер: <b>{master}</b>\n\n"
        "Выберите дату:",
        reply_markup=kb.as_markup()
    )
    
    await callback.answer()

# УДАЛИТЬ хендлер @router.message(BookingStates.choosing_master) - он не нужен!
# Логика встроена в callback_query выше
```

**Проверка:**
```
После выбора услуги должны быть кнопки: "Мастер 1", "Мастер 2", "Мастер 3"
Нажать на мастера
Должно перейти к выбору даты
```

**Файлы:**
- `handlers/booking.py` (добавить)

---

#### ✅ Задача 3.6: Выбор даты (25 мин)

**Что делать:**
- Создать функцию для генерации календаря
- Показать 5-7 следующих рабочих дней (пн-пт)
- Перейти в состояние `choosing_date`

**Создать handlers/calendar_helper.py:**
```python
from datetime import datetime, timedelta
from config import WORKING_DAYS

def get_next_working_days(days_count=7):
    """Получить следующие рабочие дни (пн-пт)"""
    working_days = []
    current_date = datetime.now()
    
    while len(working_days) < days_count:
        current_date += timedelta(days=1)
        
        # Проверить, рабочий ли день (пн-пт = 0-4)
        if current_date.weekday() < 5:
            working_days.append(current_date)
    
    return working_days
```

**Добавить в handlers/booking.py:**
```python
from sheets.google_sheets import get_available_slots

@router.callback_query(BookingStates.choosing_date, F.data.startswith("date_"))
async def process_date_choice(callback: types.CallbackQuery, state: FSMContext):
    """Обработка выбора даты и показ выбора времени"""
    date_str = callback.data.split("_", 1)[1]
    data = await state.get_data()
    master = data.get('master')
    
    # Сохранить выбор даты
    await state.update_data(date=date_str)
    
    logger.info(f"Пользователь {callback.from_user.id} выбрал дату: {date_str}")
    
    # Перейти к выбору времени
    await state.set_state(BookingStates.choosing_time)
    
    # Сразу получить свободные слоты для этого мастера (не ждать message!)
    available_slots = get_available_slots(date_str, master)
    
    if not available_slots:
        await callback.message.edit_text(
            f"❌ К сожалению, на {date_str} нет свободных слотов у мастера {master}.\n"
            "Пожалуйста, выберите другую дату."
        )
        # Вернуться к выбору даты
        await state.set_state(BookingStates.choosing_date)
        await callback.answer()
        return
    
    # Создать кнопки со временем
    kb = InlineKeyboardBuilder()
    for slot in available_slots:
        kb.add(
            types.InlineKeyboardButton(
                text=slot,
                callback_data=f"time_{slot}"
            )
        )
    kb.adjust(3)  # 3 кнопки в строке
    
    await callback.message.edit_text(
        f"✅ Выбрана дата: <b>{date_str}</b>\n\n"
        f"Свободное время у мастера {master}:",
        reply_markup=kb.as_markup()
    )
    
    await callback.answer()

# УДАЛИТЬ хендлер @router.message(BookingStates.choosing_date) - логика в callback выше
```

**Проверка:**
```
После выбора мастера должны быть кнопки с датами (только рабочие дни)
Дни должны быть в формате: "Пн 15.06.2026"
Нажать на дату
Должно перейти к выбору времени
```

**Файлы:**
- `handlers/calendar_helper.py` (создать)
- `handlers/booking.py` (добавить)

---

#### ✅ Задача 3.7: Выбор времени (25 мин)

**Что делать:**
- Получить свободные слоты на выбранную дату
- Показать их кнопками (9:00-18:00)
- Перейти в состояние `choosing_time`

**Добавить в handlers/booking.py:**
```python
from sheets.google_sheets import get_available_slots

# Удалить @router.message(BookingStates.choosing_time) - время выбирается только через callback!

@router.callback_query(BookingStates.choosing_time, F.data.startswith("time_"))
async def process_time_choice(callback: types.CallbackQuery, state: FSMContext):
    """Обработка выбора времени и переход к вводу имени"""
    time_str = callback.data.split("_", 1)[1]
    
    # Сохранить выбор
    await state.update_data(time=time_str)
    
    logger.info(f"Пользователь {callback.from_user.id} выбрал время: {time_str}")
    
    # Перейти к вводу имени
    await state.set_state(BookingStates.choosing_name)
    
    await callback.message.edit_text(
        f"✅ Выбрано время: <b>{time_str}</b>\n\n"
        "Введите ваше имя:"
    )
    
    await callback.answer()
```

**Проверка:**
```
После выбора даты должны быть кнопки с временем (9:00, 10:00, 11:00, ...)
Кнопки должны быть по 3 в строке
Нажать на время
Должно попросить ввести имя
```

**Файлы:**
- `handlers/booking.py` (добавить)

---

### Блок 4: Сбор контактов (9-10 задачи)

#### ✅ Задача 4.1: Ввод имени (20 мин)

**Что делать:**
- Перейти в состояние `choosing_name`
- Попросить ввести имя текстом
- Валидировать имя
- Сохранить в контексте

**Создать utils/validators.py:**
```python
import re
from config import logger

def validate_name(name: str) -> tuple[bool, str]:
    """Валидировать имя
    
    Возвращает: (valid, error_message)
    """
    name = name.strip()
    
    if not name:
        return False, "❌ Пожалуйста, введите имя"
    
    if len(name) > 50:
        return False, "❌ Имя не должно быть длиннее 50 символов"
    
    # Проверить что только буквы и пробелы
    if not re.match(r'^[а-яА-ЯёЁa-zA-Z\s\-]+$', name):
        return False, "❌ Имя должно содержать только буквы"
    
    return True, ""

def validate_phone(phone: str) -> tuple[bool, str]:
    """Валидировать телефон"""
    phone = phone.strip()
    
    if not phone:
        return False, "❌ Пожалуйста, введите телефон"
    
    # Проверить формат (позволяем +7, 8, или 7)
    pattern = r'^(\+7|7|8)\d{10}$'
    if not re.match(pattern, phone.replace('-', '').replace(' ', '')):
        return False, "❌ Неверный формат телефона (пример: +79991234567)"
    
    return True, ""

def validate_telegram(telegram_id: str) -> tuple[bool, str]:
    """Валидировать Telegram ник"""
    telegram_id = telegram_id.strip()
    
    if not telegram_id:
        return False, "❌ Пожалуйста, введите ник в Telegram"
    
    # Telegram ник начинается с @ или может быть просто буквы/цифры
    pattern = r'^@?[a-zA-Z0-9_]{5,32}$'
    if not re.match(pattern, telegram_id):
        return False, "❌ Неверный формат Telegram ника"
    
    if not telegram_id.startswith('@'):
        telegram_id = '@' + telegram_id
    
    return True, ""
```

**Добавить в handlers/booking.py:**
```python
from utils.validators import validate_name

@router.message(BookingStates.choosing_name)
async def process_name(message: types.Message, state: FSMContext):
    """Обработка ввода имени"""
    name = message.text
    
    # Валидировать
    valid, error = validate_name(name)
    if not valid:
        await message.answer(error)
        return
    
    # Сохранить
    await state.update_data(name=name)
    logger.info(f"Пользователь {message.from_user.id} ввел имя: {name}")
    
    # Перейти к вводу телефона
    await state.set_state(BookingStates.choosing_phone)
    await message.answer("✅ Имя сохранено!\n\nТеперь введите ваш номер телефона:")
```

**Проверка:**
```
После выбора времени бот просит имя
Написать "Анна"
Должно: "✅ Имя сохранено!"
Написать пустую строку
Должно: "❌ Пожалуйста, введите имя"
Написать "123456"
Должно: "❌ Имя должно содержать только буквы"
```

**Файлы:**
- `utils/__init__.py` (создать пустой)
- `utils/validators.py` (создать)
- `handlers/booking.py` (добавить)

---

#### ✅ Задача 4.2: Ввод телефона (20 мин)

**Что делать:**
- Перейти в состояние `choosing_phone`
- Попросить телефон
- Валидировать телефон
- Сохранить в контексте

**Добавить в handlers/booking.py:**
```python
from utils.validators import validate_phone

@router.message(BookingStates.choosing_phone)
async def process_phone(message: types.Message, state: FSMContext):
    """Обработка ввода телефона"""
    phone = message.text
    
    # Валидировать
    valid, error = validate_phone(phone)
    if not valid:
        await message.answer(error)
        return
    
    # Нормализировать телефон
    phone = phone.replace('-', '').replace(' ', '')
    if phone.startswith('8'):
        phone = '+7' + phone[1:]
    elif phone.startswith('7') and not phone.startswith('+'):
        phone = '+' + phone
    
    # Сохранить
    await state.update_data(phone=phone)
    logger.info(f"Пользователь {message.from_user.id} ввел телефон: {phone}")
    
    # Перейти к вводу Telegram ника
    await state.set_state(BookingStates.choosing_telegram)
    await message.answer("✅ Телефон сохранен!\n\nТеперь введите ваш ник в Telegram (например @anna):")
```

**Проверка:**
```
После имени бот просит телефон
Написать "+79991234567"
Должно: "✅ Телефон сохранен!"
Написать "123abc"
Должно: "❌ Неверный формат телефона"
```

**Файлы:**
- `handlers/booking.py` (добавить)

---

#### ✅ Задача 4.3: Ввод Telegram ника (20 мин)

**Что делать:**
- Перейти в состояние `choosing_telegram`
- Попросить Telegram ник
- Валидировать
- Сохранить в контексте

**Добавить в handlers/booking.py:**
```python
from utils.validators import validate_telegram

@router.message(BookingStates.choosing_telegram)
async def process_telegram(message: types.Message, state: FSMContext):
    """Обработка ввода Telegram ника"""
    telegram = message.text
    
    # Валидировать
    valid, error = validate_telegram(telegram)
    if not valid:
        await message.answer(error)
        return
    
    # Нормализировать (добавить @ если его нет)
    if not telegram.startswith('@'):
        telegram = '@' + telegram
    
    # Сохранить
    await state.update_data(telegram=telegram)
    logger.info(f"Пользователь {message.from_user.id} ввел Telegram: {telegram}")
    
    # Перейти к подтверждению
    await state.set_state(BookingStates.confirming_booking)
    
    await message.answer("✅ Telegram ник сохранен!\n\nПожалуйста, подождите...")
```

**Проверка:**
```
После телефона бот просит Telegram ник
Написать "@anna"
Должно: "✅ Telegram ник сохранен!"
Написать "123"
Должно: "❌ Неверный формат Telegram ника"
```

**Файлы:**
- `handlers/booking.py` (добавить)

---

#### ✅ Задача 4.4: Подтверждение заказа (25 мин)

**Что делать:**
- Показать сводку всего заказа
- Две кнопки: "✅ Подтвердить" и "❌ Отменить"
- Перейти в состояние `confirming_booking`

**Добавить в handlers/booking.py:**
```python
from sheets.google_sheets import get_service_by_id

@router.message(BookingStates.confirming_booking)
async def show_confirmation(message: types.Message, state: FSMContext):
    """Показать подтверждение заказа"""
    data = await state.get_data()
    
    # Получить информацию об услуге
    service = get_service_by_id(data['service_id'])
    
    # Создать сводку
    summary = (
        "📋 <b>Проверьте ваш заказ:</b>\n\n"
        f"<b>Услуга:</b> {service['name']} ({service['price']}₽)\n"
        f"<b>Дата:</b> {data['date']}\n"
        f"<b>Время:</b> {data['time']}\n"
        f"<b>Мастер:</b> {data['master']}\n"
        f"<b>Имя:</b> {data['name']}\n"
        f"<b>Телефон:</b> {data['phone']}\n"
        f"<b>Telegram:</b> {data['telegram']}\n\n"
        "Всё правильно?"
    )
    
    # Создать клавиатуру
    kb = InlineKeyboardBuilder()
    kb.add(
        types.InlineKeyboardButton(text="✅ Подтвердить", callback_data="confirm_yes"),
        types.InlineKeyboardButton(text="❌ Отменить", callback_data="confirm_no")
    )
    kb.adjust(2)
    
    await message.answer(
        summary,
        reply_markup=kb.as_markup(),
        parse_mode="HTML"
    )

@router.callback_query(BookingStates.confirming_booking, F.data == "confirm_no")
async def cancel_booking(callback: types.CallbackQuery, state: FSMContext):
    """Отменить бронирование"""
    logger.info(f"Пользователь {callback.from_user.id} отменил бронирование")
    
    await state.clear()
    
    kb = ReplyKeyboardBuilder()
    kb.add(
        types.KeyboardButton(text="📅 Запись"),
        types.KeyboardButton(text="📋 Мои записи")
    )
    kb.add(
        types.KeyboardButton(text="ℹ️ Контакты"),
        types.KeyboardButton(text="❓ FAQ")
    )
    kb.adjust(2)
    
    await callback.message.edit_text("❌ Бронирование отменено. Возвращаемся в главное меню.")
    await callback.message.answer(
        "Что вы хотите сделать?",
        reply_markup=kb.as_markup(resize_keyboard=True)
    )
    
    await callback.answer()
```

**Проверка:**
```
После ввода всех полей должна быть сводка:
📋 Проверьте ваш заказ:

Услуга: Маникюр (1000₽)
Дата: 15.06.2026
Время: 10:00
Мастер: Мастер 1
Имя: Анна
Телефон: +79991234567
Telegram: @anna

Кнопки: "✅ Подтвердить" и "❌ Отменить"
```

**Файлы:**
- `handlers/booking.py` (добавить)

---

### Блок 5: Сохранение и уведомления (11-13 задачи)

#### ✅ Задача 5.1: Сохранить заказ в Google Sheets (25 мин)

**Что делать:**
- Обработчик нажатия "✅ Подтвердить"
- Вызвать `save_booking()` из Google Sheets
- Получить ID заказа
- Завершить процесс

**Добавить в handlers/booking.py:**
```python
from sheets.google_sheets import save_booking

@router.callback_query(BookingStates.confirming_booking, F.data == "confirm_yes")
async def confirm_booking(callback: types.CallbackQuery, state: FSMContext):
    """Подтвердить и сохранить заказ"""
    data = await state.get_data()
    
    try:
        # Подготовить данные для сохранения
        booking_data = {
            'date': data['date'],
            'time': data['time'],
            'master': data['master'],
            'service': data['service_id'],
            'name': data['name'],
            'phone': data['phone'],
            'telegram': data['telegram']
        }
        
        # Сохранить в Google Sheets
        booking_id = save_booking(booking_data)
        
        if booking_id:
            logger.info(f"Заказ {booking_id} сохранён для пользователя {callback.from_user.id}")
            
            # Очистить состояние
            await state.clear()
            
            # Ответить клиенту
            await callback.message.edit_text(
                f"✅ <b>Заказ подтверждён!</b>\n\n"
                f"<b>Номер вашего заказа:</b> {booking_id}\n"
                f"Благодарим за то, что выбрали нас! 💅\n\n"
                "Вы можете посмотреть его в разделе 'Мои записи'"
            )
            
            # Отправить уведомление админу (будет в задаче 5.2)
            # await send_admin_notification(...)
        else:
            await callback.message.edit_text(
                "❌ Произошла ошибка при сохранении заказа.\n"
                "Пожалуйста, свяжитесь с администратором."
            )
            logger.error(f"Ошибка сохранения заказа для пользователя {callback.from_user.id}")
        
        await callback.answer()
    except Exception as e:
        logger.error(f"Ошибка при подтверждении заказа: {e}")
        await callback.message.edit_text(
            "❌ Произошла ошибка. Пожалуйста, попробуйте позже."
        )
        await callback.answer()
```

**Проверка:**
```
Нажать "✅ Подтвердить"
Должно: "✅ Заказ подтверждён! Номер вашего заказа: 001"
Открыть Google Sheets
В листе "Заказы" должна быть новая строка
Слот в "Расписание" должен быть отмечен как "занято"
```

**Файлы:**
- `handlers/booking.py` (добавить)

---

#### ✅ Задача 5.2: Отправить уведомление админу (25 мин)

**Что делать:**
- После сохранения заказа отправить сообщение админу
- Сообщение содержит все детали заказа
- Создать функцию для отправки уведомлений

**Создать handlers/notifications.py:**
```python
from aiogram import Bot
from config import ADMIN_ID, logger

async def send_admin_notification(bot: Bot, booking_id: str, booking_data: dict):
    """Отправить уведомление админу о новом заказе"""
    try:
        message_text = (
            f"🔔 <b>Новая запись!</b>\n\n"
            f"<b>ID заказа:</b> {booking_id}\n"
            f"<b>Клиент:</b> {booking_data['name']}\n"
            f"<b>Услуга:</b> {booking_data['service']}\n"
            f"<b>Дата и время:</b> {booking_data['date']} в {booking_data['time']}\n"
            f"<b>Мастер:</b> {booking_data['master']}\n"
            f"<b>Телефон:</b> {booking_data['phone']}\n"
            f"<b>Telegram:</b> {booking_data['telegram']}\n"
        )
        
        await bot.send_message(
            ADMIN_ID,
            message_text,
            parse_mode="HTML"
        )
        
        logger.info(f"Уведомление админу о заказе {booking_id} отправлено")
    except Exception as e:
        logger.error(f"Ошибка при отправке уведомления админу: {e}")

async def send_client_confirmation(bot: Bot, user_id: int, booking_id: str, booking_data: dict):
    """Отправить подтверждение клиенту"""
    try:
        message_text = (
            f"✅ <b>Ваша запись подтверждена!</b>\n\n"
            f"<b>Номер заказа:</b> {booking_id}\n"
            f"<b>Дата и время:</b> {booking_data['date']} в {booking_data['time']}\n"
            f"<b>Мастер:</b> {booking_data['master']}\n"
            f"<b>Стоимость:</b> {booking_data['price']}₽\n\n"
            "Увидимся скоро! 💅"
        )
        
        await bot.send_message(
            user_id,
            message_text,
            parse_mode="HTML"
        )
        
        logger.info(f"Подтверждение заказа {booking_id} отправлено клиенту")
    except Exception as e:
        logger.error(f"Ошибка при отправке подтверждения клиенту: {e}")
```

**Обновить handlers/booking.py:**
```python
from handlers.notifications import send_admin_notification

# В функции confirm_booking:
if booking_id:
    # ... предыдущий код ...
    
    # Подготовить данные для уведомления админу
    service = get_service_by_id(data['service_id'])
    booking_data = {
        'name': data['name'],
        'service': service['name'],
        'date': data['date'],
        'time': data['time'],
        'master': data['master'],
        'phone': data['phone'],
        'telegram': data['telegram'],
        'price': service['price']
    }
    
    # Отправить уведомление админу
    await send_admin_notification(callback.bot, booking_id, booking_data)
```

**Проверка:**
```
Создать новый заказ
Админ (ID из ADMIN_ID) должен получить сообщение:
"🔔 Новая запись!
ID заказа: 001
Клиент: Анна
Услуга: Маникюр
Дата и время: 15.06.2026 в 10:00
Мастер: Мастер 1
Телефон: +79991234567
Telegram: @anna"
```

**Файлы:**
- `handlers/notifications.py` (создать)
- `handlers/booking.py` (обновить)

---

#### ✅ Задача 5.3: Отправить подтверждение клиенту (20 мин)

**Что делать:**
- После подтверждения отправить клиенту полное сообщение
- Добавить информацию о том, как отменить

**Обновить handlers/booking.py в функции confirm_booking:**
```python
from handlers.notifications import send_admin_notification, send_client_confirmation

# После send_admin_notification:
if booking_id:
    # ... предыдущий код ...
    
    # Отправить подтверждение клиенту
    await send_client_confirmation(
        callback.bot,
        callback.from_user.id,
        booking_id,
        booking_data
    )
```

**Проверка:**
```
Создать новый заказ
Клиент должен получить сообщение:
"✅ Ваша запись подтверждена!

Номер заказа: 001
Дата и время: 15.06.2026 в 10:00
Мастер: Мастер 1
Стоимость: 1000₽

Увидимся скоро! 💅"
```

**Файлы:**
- `handlers/notifications.py` (обновить)
- `handlers/booking.py` (обновить)

---

### Блок 6: История заказов (14-15 задачи)

#### ✅ Задача 6.1: Получить заказы клиента из Google Sheets (25 мин)

**Что делать:**
- Функция `get_user_bookings(telegram_id)` в Google Sheets
- Получить все заказы пользователя
- Отфильтровать по Telegram ID

**Добавить в sheets/google_sheets.py:**
```python
def get_user_bookings(telegram_id: str):
    """Получить все заказы пользователя по Telegram ID
    
    Возвращает список заказов в формате:
    [
        {
            'id': '001',
            'date': '15.06.2026',
            'time': '10:00',
            'master': 'Мастер 1',
            'service': 'Маникюр',
            'status': 'Подтверждён'
        },
        ...
    ]
    """
    try:
        spreadsheet = get_spreadsheet()
        worksheet = spreadsheet.worksheet('Заказы')
        rows = worksheet.get_all_records()
        
        user_bookings = []
        for row in rows:
            if row['Telegram'] == telegram_id:
                user_bookings.append({
                    'id': row['ID'],
                    'date': row['Дата'],
                    'time': row['Время'],
                    'master': row['Мастер'],
                    'service': row['Услуга'],
                    'status': row['Статус']
                })
        
        return user_bookings
    except Exception as e:
        logger.error(f"Ошибка получения заказов пользователя: {e}")
        return []
```

**Проверка:**
```python
from sheets.google_sheets import get_user_bookings

bookings = get_user_bookings('@anna')
print(bookings)
# Должно вернуть список всех заказов
```

**Файлы:**
- `sheets/google_sheets.py` (добавить функцию)

---

#### ✅ Задача 6.2: Обработчик "📋 Мои записи" (25 мин)

**Что делать:**
- Обработчик кнопки "📋 Мои записи"
- Получить заказы клиента
- Показать их в виде списка
- Если нет - сообщение "У вас нет записей"

**Создать handlers/user_bookings.py:**
```python
from aiogram import Router, types, F
from sheets.google_sheets import get_user_bookings
from config import logger

router = Router()

@router.message(F.text == "📋 Мои записи")
async def show_user_bookings(message: types.Message):
    """Показать все заказы пользователя"""
    telegram_id = message.from_user.username
    
    logger.info(f"Пользователь {message.from_user.id} ({telegram_id}) запросил свои записи")
    
    # Получить заказы
    bookings = get_user_bookings(f"@{telegram_id}" if telegram_id else None)
    
    if not bookings:
        await message.answer(
            "📋 У вас пока нет записей.\n\n"
            "Нажмите '📅 Запись' чтобы записаться на услугу."
        )
        return
    
    # Сформировать сообщение со всеми заказами
    bookings_text = "📋 <b>Ваши записи:</b>\n\n"
    for i, booking in enumerate(bookings, 1):
        status_emoji = "✅" if booking['status'] == "Подтверждён" else "❌"
        bookings_text += (
            f"{i}️⃣ <b>{booking['service']}</b>\n"
            f"   📅 {booking['date']} ⏰ {booking['time']}\n"
            f"   👨‍💼 Мастер: {booking['master']}\n"
            f"   {status_emoji} Статус: {booking['status']}\n"
            f"   🆔 ID: {booking['id']}\n\n"
        )
    
    await message.answer(bookings_text, parse_mode="HTML")
```

**В main.py добавить:**
```python
from handlers import user_bookings

dp.include_router(user_bookings.router)
```

**Проверка:**
```
Нажать "📋 Мои записи"
Должны быть все заказы:
"1️⃣ Маникюр
   📅 15.06.2026 ⏰ 10:00
   👨‍💼 Мастер: Мастер 1
   ✅ Статус: Подтверждён
   🆔 ID: 001"
```

**Файлы:**
- `handlers/user_bookings.py` (создать)
- обновить `main.py`

---

### Блок 7: Отмена записей (16 задачи)

#### ✅ Задача 7.1: Функция отмены заказа (25 мин)

**Что делать:**
- Функция `cancel_booking(booking_id)` в Google Sheets
- Функция `free_up_slot(date, time, master)` - освободить слот
- Обновить статус на "Отменен"

**Добавить в sheets/google_sheets.py:**
```python
def cancel_booking(booking_id: str) -> bool:
    """Отменить заказ
    
    Возвращает: True если успешно, False если ошибка
    """
    try:
        spreadsheet = get_spreadsheet()
        worksheet = spreadsheet.worksheet('Заказы')
        rows = worksheet.get_all_records()
        
        # Найти заказ
        booking_row = None
        row_idx = None
        
        for idx, row in enumerate(rows):
            if row['ID'] == booking_id:
                booking_row = row
                row_idx = idx + 2  # +1 за заголовок, +1 за индексацию с 1
                break
        
        if not booking_row:
            logger.warning(f"Заказ {booking_id} не найден")
            return False
        
        # Обновить статус на "Отменен"
        worksheet.update_cell(row_idx, 9, 'Отменен')  # Столбец Статус
        
        # Освободить слот
        free_up_slot(booking_row['Дата'], booking_row['Время'], booking_row['Мастер'])
        
        logger.info(f"Заказ {booking_id} отменён")
        return True
    except Exception as e:
        logger.error(f"Ошибка отмены заказа: {e}")
        return False

def free_up_slot(date_str: str, time_str: str, master: str) -> bool:
    """Освободить слот в расписании
    
    Использует MASTERS.index() поэтому не зависит от конкретных номеров мастеров
    """
    try:
        spreadsheet = get_spreadsheet()
        worksheet = spreadsheet.worksheet('Расписание')
        rows = worksheet.get_all_records()
        
        # Определить номер столбца мастера (3 = первый мастер)
        master_col = MASTERS.index(master) + 3
        
        for idx, row in enumerate(rows):
            if row['Дата'] == date_str and row['Время'] == time_str:
                row_idx = idx + 2  # +1 за заголовок, +1 за индексацию
                worksheet.update_cell(row_idx, master_col, 'свободно')
                logger.info(f"Слот освобождён: {date_str} {time_str} {master}")
                return True
        
        logger.warning(f"Слот не найден: {date_str} {time_str} {master}")
        return False
    except Exception as e:
        logger.error(f"Ошибка освобождения слота: {e}")
        return False
```

**Проверка:**
```python
from sheets.google_sheets import cancel_booking

result = cancel_booking('001')
print(result)  # True

# В Google Sheets:
# - Статус заказа 001 должен быть "Отменен"
# - Слот должен быть свободен в "Расписание"
```

**Файлы:**
- `sheets/google_sheets.py` (добавить функции)

---

#### ✅ Задача 7.2: Обработчик отмены из сообщения клиента (25 мин)

**Что делать:**
- Обработчик команды типа `/cancel_001`
- Показать подтверждение
- При подтверждении отменить заказ

**Создать handlers/cancellation.py:**
```python
from aiogram import Router, types, F
from aiogram.utils.keyboard import InlineKeyboardBuilder
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup
from sheets.google_sheets import cancel_booking, get_user_bookings
from handlers.notifications import send_admin_notification
from config import logger

router = Router()

class CancelStates(StatesGroup):
    confirming = State()

@router.message(F.text.startswith('/cancel_'))
async def start_cancel(message: types.Message, state: FSMContext):
    """Начать процесс отмены заказа"""
    booking_id = message.text.replace('/cancel_', '')
    
    logger.info(f"Пользователь {message.from_user.id} начал отмену заказа {booking_id}")
    
    # Сохранить ID в контексте
    await state.update_data(booking_id=booking_id)
    await state.set_state(CancelStates.confirming)
    
    # Создать клавиатуру подтверждения
    kb = InlineKeyboardBuilder()
    kb.add(
        types.InlineKeyboardButton(text="✅ Да, отменить", callback_data="cancel_confirm"),
        types.InlineKeyboardButton(text="❌ Отменить", callback_data="cancel_no")
    )
    kb.adjust(2)
    
    await message.answer(
        f"⚠️ <b>Вы уверены?</b>\n\n"
        f"Вы хотите отменить заказ № {booking_id}?\n"
        "Это действие нельзя отменить.",
        reply_markup=kb.as_markup(),
        parse_mode="HTML"
    )

@router.callback_query(CancelStates.confirming, F.data == "cancel_confirm")
async def confirm_cancel(callback: types.CallbackQuery, state: FSMContext):
    """Подтвердить отмену"""
    data = await state.get_data()
    booking_id = data['booking_id']
    
    # Отменить заказ
    result = cancel_booking(booking_id)
    
    if result:
        logger.info(f"Заказ {booking_id} успешно отменён пользователем {callback.from_user.id}")
        
        await callback.message.edit_text(
            f"✅ <b>Заказ {booking_id} отменён</b>\n\n"
            "Если у вас есть вопросы, свяжитесь с администратором салона."
        )
        
        # Уведомить админа об отмене (отправить простое сообщение)
        try:
            admin_message = (
                f"❌ <b>Заказ отменён!</b>\n\n"
                f"<b>ID заказа:</b> {booking_id}\n"
                f"<b>Пользователь:</b> {callback.from_user.first_name or 'Неизвестный'}\n"
                f"<b>Telegram ID:</b> {callback.from_user.id}\n"
                "Клиент отменил запись через бота."
            )
            await callback.bot.send_message(
                ADMIN_ID,
                admin_message,
                parse_mode="HTML"
            )
        except Exception as e:
            logger.error(f"Ошибка при отправке уведомления админу об отмене: {e}")
    else:
        await callback.message.edit_text(
            f"❌ Ошибка при отмене заказа {booking_id}"
        )
    
    await state.clear()
    await callback.answer()

@router.callback_query(CancelStates.confirming, F.data == "cancel_no")
async def cancel_cancel(callback: types.CallbackQuery, state: FSMContext):
    """Отменить отмену"""
    await state.clear()
    
    await callback.message.edit_text("Отмена отменена. Ваш заказ остаётся в силе.")
    await callback.answer()
```

**В main.py добавить:**
```python
from handlers import cancellation

dp.include_router(cancellation.router)
```

**Проверка:**
```
В разделе "Мои записи" отправить сообщение: /cancel_001
Должно быть подтверждение: "⚠️ Вы уверены?"
Нажать "✅ Да, отменить"
Должно: "✅ Заказ 001 отменён"
Админ должен получить уведомление об отмене
В Google Sheets статус должен быть "Отменен"
```

**Файлы:**
- `handlers/cancellation.py` (создать)
- обновить `main.py`

---

### Блок 8: Дополнительные функции (17-19 задачи)

#### ✅ Задача 8.1: Контакты и FAQ (20 мин)

**Что делать:**
- Обработчик кнопки "ℹ️ Контакты"
- Обработчик кнопки "❓ FAQ"
- Показать информацию

**Создать handlers/info.py:**
```python
from aiogram import Router, types, F
from aiogram.utils.keyboard import InlineKeyboardBuilder
from config import logger

router = Router()

@router.message(F.text == "ℹ️ Контакты")
async def show_contacts(message: types.Message):
    """Показать контакты салона"""
    logger.info(f"Пользователь {message.from_user.id} запросил контакты")
    
    kb = InlineKeyboardBuilder()
    kb.add(
        types.InlineKeyboardButton(text="📞 Позвонить", url="tel:+79991234567"),
        types.InlineKeyboardButton(text="📍 На карте", url="https://maps.google.com/")
    )
    kb.adjust(2)
    
    contacts_text = (
        "📍 <b>Контакты салона "Дела салона"</b>\n\n"
        "📌 <b>Адрес:</b>\n"
        "ул. Красивая, д. 1, офис 10\n"
        "Москва, Россия\n\n"
        "📞 <b>Телефон:</b>\n"
        "+7 (999) 123-45-67\n\n"
        "⏰ <b>Время работы:</b>\n"
        "Пн-Пт: 09:00 - 18:00\n"
        "Сб-Вс: Выходной\n\n"
        "💬 <b>Социальные сети:</b>\n"
        "Instagram: @salon_deals\n"
        "VK: vk.com/salon_deals\n"
        "WhatsApp: +7 (999) 123-45-67"
    )
    
    await message.answer(
        contacts_text,
        reply_markup=kb.as_markup(),
        parse_mode="HTML"
    )

@router.message(F.text == "❓ FAQ")
async def show_faq(message: types.Message):
    """Показать часто задаваемые вопросы"""
    logger.info(f"Пользователь {message.from_user.id} запросил FAQ")
    
    faq_text = (
        "❓ <b>Часто задаваемые вопросы</b>\n\n"
        
        "<b>❓ Как отменить запись?</b>\n"
        "Нажмите на кнопку '📋 Мои записи', найдите нужную запись и отправьте команду /cancel_XXX (где XXX - номер заказа)\n\n"
        
        "<b>❓ Можно ли перенести запись?</b>\n"
        "Да! Отмените текущую запись и создайте новую на удобное время.\n\n"
        
        "<b>❓ Где находится салон?</b>\n"
        "Мы расположены на ул. Красивая, д. 1, офис 10 в Москве. Нажмите на кнопку 'Контакты' чтобы открыть адрес на карте.\n\n"
        
        "<b>❓ Какие способы оплаты?</b>\n"
        "Мы принимаем наличные и безналичный расчёт. Оплата происходит в салоне после оказания услуги.\n\n"
        
        "<b>❓ Что делать, если я опоздаю?</b>\n"
        "Пожалуйста, позвоните нам как можно скорее. Если вы опоздали более чем на 15 минут, место может быть предоставлено другому клиенту.\n\n"
        
        "<b>❓ Есть ли скидки?</b>\n"
        "У нас часто проводятся акции и скидки. Следите за нашими социальными сетями или позвоните для подробной информации.\n"
    )
    
    await message.answer(faq_text, parse_mode="HTML")
```

**В main.py добавить:**
```python
from handlers import info

dp.include_router(info.router)
```

**Проверка:**
```
Нажать "ℹ️ Контакты"
Должна быть информация с контактами и кнопки для звонка и карты

Нажать "❓ FAQ"
Должны быть вопросы и ответы
```

**Файлы:**
- `handlers/info.py` (создать)
- обновить `main.py`

---

#### ✅ Задача 8.2: Логирование всех действий (25 мин)

**Что делать:**
- Логировать все события (старт, выбор услуги, ошибки)
- Создать файл логов на VPS
- Использовать правильный уровень логирования

**Создать utils/logger.py:**
```python
import logging
import os
from logging.handlers import RotatingFileHandler
from pathlib import Path

def setup_logger(name: str = 'salon_bot') -> logging.Logger:
    """Настроить логирование (кроссплатформное - работает на Windows и Linux)"""
    logger = logging.getLogger(name)
    logger.setLevel(logging.INFO)
    
    # Определить директорию логов (./logs на Windows, /var/log/salon-bot на Linux)
    if os.name == 'nt':  # Windows
        logs_dir = Path('./logs')
    else:  # Linux/Mac
        logs_dir = Path('/var/log/salon-bot')
    
    logs_dir.mkdir(parents=True, exist_ok=True)
    
    # Формат логов
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    
    # Файловый обработчик (ротирующийся файл)
    file_handler = RotatingFileHandler(
        logs_dir / 'bot.log',
        maxBytes=10485760,  # 10MB
        backupCount=5
    )
    file_handler.setFormatter(formatter)
    logger.addHandler(file_handler)
    
    # Консольный обработчик
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(formatter)
    logger.addHandler(console_handler)
    
    return logger

# Используется в config.py
logger = setup_logger()
```

**Обновить config.py:**
```python
# Вместо встроенного логирования использовать:
from utils.logger import setup_logger

logger = setup_logger()
```

**Проверка:**
```bash
tail -f /var/log/salon-bot/bot.log
# Должны видны все действия в реальном времени
```

**Файлы:**
- `utils/logger.py` (создать)
- обновить `config.py`

---

#### ✅ Задача 8.3: Обработка ошибок и исключений (30 мин)

**Что делать:**
- Try/except блоки для Google Sheets функций
- Middleware для обработки ошибок
- Уведомления админу об ошибках

**Создать handlers/errors.py:**
```python
from aiogram import Router, types
from aiogram.types import Update
from config import logger, ADMIN_ID

router = Router()

class BotException(Exception):
    """Базовое исключение бота"""
    pass

@router.message()
async def unknown_message_handler(message: types.Message):
    """Обработчик неизвестных команд"""
    await message.answer(
        "❌ Извините, я не понял вашу команду.\n"
        "Используйте кнопки в меню ниже."
    )
```

**Создать utils/error_middleware.py (для глобальной обработки ошибок):**
```python
from typing import Callable, Any, Awaitable
from aiogram import BaseMiddleware, types
from config import logger, ADMIN_ID

class ErrorMiddleware(BaseMiddleware):
    """Middleware для обработки ошибок"""
    
    async def __call__(
        self,
        handler: Callable[[types.Update], Awaitable[Any]],
        event: types.Update,
        data: dict
    ) -> Any:
        try:
            return await handler(event, data)
        except Exception as e:
            logger.error(f"Ошибка в обработчике: {e}", exc_info=True)
            
            # Попытаться отправить сообщение админу
            try:
                error_message = (
                    f"❌ <b>Ошибка бота!</b>\n\n"
                    f"<b>Тип:</b> {type(e).__name__}\n"
                    f"<b>Сообщение:</b> {str(e)[:200]}\n"
                )
                
                # Отправить админу только если есть bot в data
                if 'bot' in data:
                    await data['bot'].send_message(ADMIN_ID, error_message, parse_mode="HTML")
            except Exception as notify_error:
                logger.error(f"Ошибка при отправке уведомления об ошибке: {notify_error}")
            
            # Повторно выбросить ошибку для обработки aiogram
            raise
```

**В main.py добавить middleware:**
```python
from utils.error_middleware import ErrorMiddleware
from handlers import errors

# После создания dp
dp.message.middleware(ErrorMiddleware())

# Подключить router ошибок (для неизвестных команд)
dp.include_router(errors.router)
```

**Проверка:**
```
Если Google API недоступен - бот должен сказать:
"Извините, технические проблемы. Попробуйте позже"

В логе должны быть записи об ошибке
Админ должен получить уведомление об ошибке (опционально)
```

**Файлы:**
- `handlers/errors.py` (создать)
- `utils/error_middleware.py` (создать)
- обновить `main.py`

---

### Блок 9: Деплой (20-22 задачи)

#### ✅ Задача 9.1: Создать systemd сервис (20 мин)

**Что делать:**
- Создать файл `/etc/systemd/system/salon-bot.service`
- Настроить автозапуск и перезапуск
- Настроить логирование

**Содержание /etc/systemd/system/salon-bot.service:**
```ini
[Unit]
Description=Salon Bot Service
After=network.target

[Service]
Type=simple
User=salon-bot
WorkingDirectory=/home/salon-bot/salon-bot
ExecStart=/usr/bin/python3 /home/salon-bot/salon-bot/main.py

# Переменные окружения
Environment="TELEGRAM_TOKEN=your-token-here"
Environment="ADMIN_ID=123456789"
Environment="GOOGLE_SHEETS_ID=your-sheet-id"
Environment="GOOGLE_SERVICE_ACCOUNT_FILE=/home/salon-bot/salon-bot/credentials.json"

# Автозапуск при ошибке
Restart=always
RestartSec=10

# Логирование
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**Команды для установки:**
```bash
# Создать пользователя
sudo useradd -m -s /bin/bash salon-bot

# Скопировать файл сервиса
sudo cp /path/to/salon-bot.service /etc/systemd/system/

# Перезагрузить systemd
sudo systemctl daemon-reload

# Включить автозапуск
sudo systemctl enable salon-bot

# Запустить сервис
sudo systemctl start salon-bot

# Проверить статус
sudo systemctl status salon-bot

# Смотреть логи
sudo journalctl -u salon-bot -f
```

**Проверка:**
```bash
sudo systemctl status salon-bot
# Должно быть: Active: active (running)

sudo journalctl -u salon-bot -f
# Должны быть логи: "🤖 Бот запущен"
```

**Файлы:**
- `/etc/systemd/system/salon-bot.service`

---

#### ✅ Задача 9.2: Настроить окружение на VPS (25 мин)

**Что делать:**
- Создать пользователя
- Клонировать репо
- Установить зависимости
- Создать .env файл

**Пошаговые команды:**
```bash
# 1. Подключиться к VPS
ssh user@your-vps-ip

# 2. Обновить систему
sudo apt update && sudo apt upgrade -y

# 3. Установить Python (если не установлен)
sudo apt install -y python3 python3-pip python3-venv

# 4. Создать директорию проекта
sudo mkdir -p /home/salon-bot/salon-bot
sudo chown salon-bot:salon-bot /home/salon-bot/salon-bot

# 5. Перейти в директорию
cd /home/salon-bot/salon-bot

# 6. Инициализировать git репо (или клонировать)
git clone https://github.com/your-repo/salon-bot.git .

# 7. Создать виртуальное окружение
python3 -m venv venv
source venv/bin/activate

# 8. Установить зависимости
pip install -r requirements.txt

# 9. Создать .env файл
cat > .env << EOF
TELEGRAM_TOKEN=your-token-here
ADMIN_ID=123456789
GOOGLE_SHEETS_ID=your-sheet-id
GOOGLE_SERVICE_ACCOUNT_FILE=/home/salon-bot/salon-bot/credentials.json
EOF

# 10. Скопировать credentials.json
# (скопировать файл с локальной машины на VPS)
scp credentials.json user@your-vps-ip:/home/salon-bot/salon-bot/

# 11. Дать права доступа
sudo chown salon-bot:salon-bot /home/salon-bot/salon-bot/credentials.json
chmod 600 /home/salon-bot/salon-bot/credentials.json

# 12. Тестовый запуск
sudo -u salon-bot /home/salon-bot/salon-bot/venv/bin/python3 /home/salon-bot/salon-bot/main.py
# Должно вывести: "🤖 Бот запущен"
```

**Проверка:**
```bash
cd /home/salon-bot/salon-bot
python3 main.py
# Должно запуститься без ошибок
```

**Файлы:**
- `.env` на VPS (со значениями)
- `credentials.json` на VPS

---

#### ✅ Задача 9.3: Запустить и проверить (30 мин)

**Что делать:**
- Запустить сервис
- Проверить что бот в сети
- Пройти полный цикл записи
- Проверить Google Sheets и уведомления

**Пошаговая проверка:**
```bash
# 1. Запустить сервис
sudo systemctl start salon-bot

# 2. Проверить статус
sudo systemctl status salon-bot
# Должно: Active: active (running)

# 3. Смотреть логи
sudo journalctl -u salon-bot -f
```

**В Telegram:**
```
1. Отправить /start боту
   ✅ Должен ответить с главным меню

2. Нажать "📅 Запись"
   ✅ Должны быть кнопки услуг

3. Выбрать услугу, мастера, дату, время
   ✅ Процесс должен работать

4. Ввести контакты
   ✅ Валидация должна работать

5. Подтвердить заказ
   ✅ Должна быть сводка и кнопки подтверждения

6. Нажать "✅ Подтвердить"
   ✅ Заказ должен быть создан (ID: 001)
   ✅ Должна быть сводка заказа

7. Админ должен получить уведомление
   ✅ Сообщение с деталями заказа

8. Открыть Google Sheets
   ✅ В листе "Заказы" должна быть новая строка
   ✅ В листе "Расписание" слот должен быть "занято"

9. Нажать "📋 Мои записи"
   ✅ Должна быть запись в истории

10. Попробовать отменить запись
    ✅ Команда /cancel_001
    ✅ Запрос подтверждения
    ✅ После отмены - статус "Отменен"
```

**Проверить логи:**
```bash
sudo journalctl -u salon-bot | grep -E "Пользователь|Заказ|ошибка"
# Должны быть записи обо всех действиях
```

**Финальная проверка:**
```bash
# Сервис работает
sudo systemctl status salon-bot
# Active: active (running)

# Логи без ошибок
sudo journalctl -u salon-bot -n 50
# Должны быть только информационные сообщения

# Google Sheets заполнена
# Все заказы в таблице

# Уведомления работают
# Админ получает сообщения
```

**Файлы:**
- `/etc/systemd/system/salon-bot.service` (проверить)
- `.env` на VPS (проверить значения)

---

## 📊 ИТОГОВАЯ ТАБЛИЦА ВСЕХ ЗАДАЧ

| Блок | № | Задача | Время | Статус |
|------|---|--------|-------|--------|
| **Setup** | 1.1 | Инициализация проекта | 20 мин | ⬜ |
| | 1.2 | Создать config.py | 25 мин | ⬜ |
| | 1.3 | Настроить Google Sheets API | 30 мин | ⬜ |
| **Google Sheets** | 2.1 | Структура листов | 20 мин | ⬜ |
| | 2.2 | Читать услуги | 25 мин | ⬜ |
| | 2.3 | Функции расписания | 30 мин | ⬜ |
| | 2.4 | Сохранить заказ | 25 мин | ⬜ |
| **Бот основа** | 3.1 | Основной файл | 20 мин | ⬜ |
| | 3.2 | Обработчик /start | 25 мин | ⬜ |
| | 3.3 | Подключить FSM | 25 мин | ⬜ |
| | 3.4 | Выбор услуги | 25 мин | ⬜ |
| | 3.5 | Выбор мастера | 25 мин | ⬜ |
| | 3.6 | Выбор даты | 25 мин | ⬜ |
| | 3.7 | Выбор времени | 25 мин | ⬜ |
| **Контакты** | 4.1 | Ввод имени | 20 мин | ⬜ |
| | 4.2 | Ввод телефона | 20 мин | ⬜ |
| | 4.3 | Ввод Telegram | 20 мин | ⬜ |
| | 4.4 | Подтверждение | 25 мин | ⬜ |
| **Сохранение** | 5.1 | Сохранить заказ | 25 мин | ⬜ |
| | 5.2 | Уведомление админу | 25 мин | ⬜ |
| | 5.3 | Подтверждение клиенту | 20 мин | ⬜ |
| **История** | 6.1 | Получить заказы | 25 мин | ⬜ |
| | 6.2 | "Мои записи" | 25 мин | ⬜ |
| **Отмена** | 7.1 | Функция отмены | 25 мин | ⬜ |
| | 7.2 | Обработчик отмены | 25 мин | ⬜ |
| **Доп функции** | 8.1 | Контакты и FAQ | 20 мин | ⬜ |
| | 8.2 | Логирование | 25 мин | ⬜ |
| | 8.3 | Обработка ошибок | 30 мин | ⬜ |
| **Деплой** | 9.1 | Systemd сервис | 20 мин | ⬜ |
| | 9.2 | Настроить VPS | 25 мин | ⬜ |
| | 9.3 | Запустить и проверить | 30 мин | ⬜ |
| | | **ИТОГО** | **~750 мин** | |
| | | **ИТОГО** | **~12.5 часов** | |

---

## 🎯 Рекомендации по выполнению

### ✅ Порядок выполнения
1. **Сначала выполнять задачи по порядку** - они зависят друг от друга
2. **После каждого блока тестировать** - проверять что работает
3. **Коммитить в git** - сохранять прогресс

### ✅ Как отслеживать прогресс
- Копировать эту таблицу в To-Do приложение
- Отмечать ✅ когда задача готова
- Вести лог того, что заняло больше времени

### ✅ Если что-то сломалось
- Прочитать логи: `journalctl -u salon-bot -f`
- Проверить Google Sheets структуру
- Проверить переменные окружения в `.env`
- Смотреть ошибки в консоли Python

### ✅ После завершения
- [ ] Весь функционал работает
- [ ] Google Sheets заполняется корректно
- [ ] Уведомления приходят админу
- [ ] Логирование работает
- [ ] Systemd сервис перезагружается при ошибке
- [ ] Можно начинать использовать салоном

---

## 📞 Для справки

**Основные файлы проекта:**
```
salon-bot/
├── main.py                    # Точка входа
├── config.py                  # Конфигурация
├── requirements.txt           # Зависимости
├── .env                       # Переменные окружения
├── credentials.json           # Google Service Account
│
├── handlers/
│   ├── __init__.py
│   ├── start.py              # /start
│   ├── booking.py            # Процесс записи
│   ├── calendar_helper.py    # Календарь
│   ├── user_bookings.py      # "Мои записи"
│   ├── cancellation.py       # Отмена заказов
│   ├── notifications.py      # Уведомления
│   ├── info.py               # Контакты и FAQ
│   └── errors.py             # Обработка ошибок
│
├── sheets/
│   ├── __init__.py
│   └── google_sheets.py      # Работа с Google Sheets
│
├── models/
│   ├── __init__.py
│   └── booking_state.py      # FSM состояния
│
└── utils/
    ├── __init__.py
    ├── logger.py             # Логирование
    └── validators.py         # Валидация данных
```

**Структура Google Sheets:**
- Лист "Услуги" - каталог услуг
- Лист "Расписание" - расписание мастеров
- Лист "Заказы" - история всех заказов

**Systemd командыдля управления:**
```bash
sudo systemctl start salon-bot     # Запустить
sudo systemctl stop salon-bot      # Остановить
sudo systemctl restart salon-bot   # Перезагрузить
sudo systemctl status salon-bot    # Статус
sudo journalctl -u salon-bot -f    # Логи в реальном времени
```

---

**Удачи в разработке! 🚀**

Если возникнут вопросы по какой-то задаче - напиши номер задачи и её описание.
