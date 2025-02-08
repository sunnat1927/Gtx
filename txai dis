import telebot
import re
from telebot import types

# Укажите токен вашего бота
API_TOKEN = "7716349680:AAGGdmeBsQi6b64_GMldazC8SotlB-UVLFI"
bot = telebot.TeleBot(API_TOKEN)

# Словарь для хранения заявок
applications = {}

# Шаги анкеты
APPLICATION_STEPS = ['fio', 'phone', 'location_choice', 'passengers']

# Функция обработки команды /start
@bot.message_handler(commands=['start'])
def start(message):
    chat_id = message.chat.id

    # Создаем клавиатуру с 1 кнопкой
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, one_time_keyboard=True)

    # Кнопка для начала анкеты
    btn1 = types.KeyboardButton("💾Anketani to'ldirish💾")

    # Добавляем кнопку анкеты в клавиатуру
    markup.add(btn1)

    # Отправляем сообщение с кнопками
    bot.send_message(chat_id, "Assalomu aleykum! Taksi zakaz qilish uchun quyidagi tugmani tanlang:", reply_markup=markup)

# Обработка нажатия на кнопку "Anketani to'ldirish"
@bot.message_handler(func=lambda message: message.text == "💾Anketani to'ldirish💾")
def request_details(message):
    chat_id = message.chat.id
    bot.send_message(chat_id, "Iltimos, ismingizni yozing (masalan: Palonchiev Pistonchi).")
    bot.register_next_step_handler(message, handle_fio)

# Обработка фамилии и имени
def handle_fio(message):
    chat_id = message.chat.id
    fio = message.text.strip()

    if re.match(r'^[A-Za-zА-Яа-яЁё]+ [A-Za-zА-Яа-яЁё]+$', fio):
        if chat_id not in applications:
            applications[chat_id] = {}
        applications[chat_id]['fio'] = fio
        bot.send_message(chat_id, "Telefon raqamingizni yozing (masalan: 901234567).")
        bot.register_next_step_handler(message, handle_phone)
    else:
        bot.send_message(chat_id, "Iltimos, to'liq familya va ismni kiriting (masalan: Palonchiev Pistonchi).")
        bot.register_next_step_handler(message, handle_fio)

# Обработка телефонного номера
def handle_phone(message):
    chat_id = message.chat.id
    phone = message.text.strip()

    if re.match(r'^\d{9}$', phone):
        applications[chat_id]['phone'] = phone

        # Создание кнопок с количеством пассажиров
        markup = types.ReplyKeyboardMarkup(resize_keyboard=True, one_time_keyboard=True)
        btn1 = types.KeyboardButton("1 Yo‘lovchi")
        btn2 = types.KeyboardButton("2 Yo‘lovchi")
        btn3 = types.KeyboardButton("3 Yo‘lovchi")
        btn4 = types.KeyboardButton("4 Yo‘lovchi")

        markup.add(btn1, btn2, btn3, btn4)

        # Отправка сообщения с кнопками для выбора количества пассажиров
        bot.send_message(chat_id, "Necha odam uchun joy kerak? Tanlang:", reply_markup=markup)
        bot.register_next_step_handler(message, handle_passengers)
    else:
        bot.send_message(chat_id, "Iltimos, to'liq telefon raqamingizni yozing (masalan: 901234567).")
        bot.register_next_step_handler(message, handle_phone)

# Обработка выбора количества пассажиров (yo‘lovchilar soni)
def handle_passengers(message):
    chat_id = message.chat.id
    passengers = message.text.strip()

    if passengers == "1 Yo‘lovchi":
        applications[chat_id]['passengers'] = 1
    elif passengers == "2 Yo‘lovchi":
        applications[chat_id]['passengers'] = 2
    elif passengers == "3 Yo‘lovchi":
        applications[chat_id]['passengers'] = 3
    elif passengers == "4 Yo‘lovchi":
        applications[chat_id]['passengers'] = 4
    else:
        bot.send_message(chat_id, "Iltimos, to'g'ri tanlovni kiriting.")
        return

    # Создание кнопок с выбором направления
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, one_time_keyboard=True)
    btn1 = types.KeyboardButton("🚗 Toshkentdan Namanganga")
    btn2 = types.KeyboardButton("🚗 Namangandan Toshkentga")
    markup.add(btn1, btn2)

    # Отправка сообщения с кнопками
    bot.send_message(chat_id, "Toshkentdan Namangangami yoki Namangandan Toshkentgami? Tanlang:", reply_markup=markup)
    bot.register_next_step_handler(message, handle_location_choice)

# Обработка выбора направления
def handle_location_choice(message):
    chat_id = message.chat.id
    location_choice = message.text.strip()

    if location_choice == "🚗 Toshkentdan Namanganga":
        applications[chat_id]['location'] = "Toshkentdan Namanganga"
    elif location_choice == "🚗 Namangandan Toshkentga":
        applications[chat_id]['location'] = "Namangandan Toshkentga"
    else:
        bot.send_message(chat_id, "Iltimos, to'g'ri tanlovni kiriting.")
        return

    # Подтверждение заявки
    bot.send_message(chat_id, "Tanlovingiz qabul qilindi. Arizangiz yuborildi! 🚗💨")

    # Отправка заявки в супергруппу
    application_text = f"Yangi ariza:\n\n" \
                       f"Ism: {applications[chat_id]['fio']}\n" \
                       f"Telefon: {applications[chat_id]['phone']}\n" \
                       f"Yo‘lovchilar soni: {applications[chat_id]['passengers']}\n" \
                       f"Manzil: {applications[chat_id]['location']}"

    GROUP_ID = "-1002489191359"  # Новый GROUP_ID для супергруппы
    bot.send_message(GROUP_ID, f"@{message.from_user.username}\n" + application_text)

    # Отправка завершающего текста пользователю
    bot.send_message(chat_id, "Sizning so‘rovingiz shafyorlar guruhiga yubirildi ✅\n"
                              "Tez orada shafyorlarimiz siz bilan aloqaga chiqishadi 📞\n"
                              "Tolingiz behatar bo‘lsin! ✨")

    # Очистка истории чата
    bot.delete_message(chat_id, message.message_id)  # Удалить последнего сообщения (после анкеты)
    
    # Оставляем кнопку для новой анкеты
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True, one_time_keyboard=True)
    btn1 = types.KeyboardButton("💾Anketani to'ldirish💾")
    markup.add(btn1)
    bot.send_message(chat_id, "Agar yana so'rov yubormoqchi bo'lsangiz, quyidagi tugmani bosing:", reply_markup=markup)

# Основной цикл
if __name__ == "__main__":


