# -
запис на Онлайн/Офлайн консультацію до Тетяни Гончарової. Простір Пазл Полтава
[bot.py.txt](https://github.com/user-attachments/files/24309162/bot.py.txt)
from telegram import (
    Update,
    ReplyKeyboardMarkup,
    KeyboardButton
)
from telegram.ext import (
    ApplicationBuilder,
    CommandHandler,
    MessageHandler,
    ContextTypes,
    filters
)

# 🔐 TOKEN твого ІСНУЮЧОГО бота
BOT_TOKEN = 8326940007:AAEO9zY0iq9v58_wq3ezRukHETJd7y9OBB8

# ----- /start -----
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [["Онлайн"], ["Офлайн"]]
    await update.message.reply_text(
        "Вітаю! 💙\n"
        "Запис на консультацію\n\n"
        "⏱ 60 хв | 💰 1200 грн\n"
        "Оберіть формат:",
        reply_markup=ReplyKeyboardMarkup(
            keyboard, resize_keyboard=True
        )
    )

# ----- ОБРОБКА ТЕКСТУ -----
async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text
    step = context.user_data.get("step")

    # Формат
    if text in ["Онлайн", "Офлайн"]:
        context.user_data["format"] = text
        context.user_data["step"] = "parent_name"
        await update.message.reply_text(
            "Напишіть імʼя мами або тата"
        )

    # Імʼя батьків
    elif step == "parent_name":
        context.user_data["parent_name"] = text
        context.user_data["step"] = "child_name"
        await update.message.reply_text("Як звати дитину?")

    # Імʼя дитини
    elif step == "child_name":
        context.user_data["child_name"] = text
        context.user_data["step"] = "child_age"
        await update.message.reply_text("Вкажіть вік дитини")

    # Вік
    elif step == "child_age":
        context.user_data["child_age"] = text
        context.user_data["step"] = "request"
        await update.message.reply_text(
            "Коротко опишіть запит"
        )

    # Запит
    elif step == "request":
        context.user_data["request"] = text
        context.user_data["step"] = "phone"

        button = KeyboardButton(
            "📱 Поділитись номером",
            request_contact=True
        )
        await update.message.reply_text(
            "Надішліть номер телефону",
            reply_markup=ReplyKeyboardMarkup(
                [[button]], resize_keyboard=True
            )
        )

# ----- КОНТАКТ -----
async def handle_contact(update: Update, context: ContextTypes.DEFAULT_TYPE):
    contact = update.message.contact
    context.user_data["phone"] = contact.phone_number

    data = context.user_data

    await update.message.reply_text(
        "Дякую! ✅\n\n"
        f"👤 Батьки: {data['parent_name']}\n"
        f"🧒 Дитина: {data['child_name']}, {data['child_age']}\n"
        f"📝 Запит: {data['request']}\n"
        f"📍 Формат: {data['format']}\n\n"
        "Найближчим часом підтверджу дату і час 💙"
    )

    # 🔜 ТУТ буде підключення Google Календаря

# ----- ЗАПУСК -----
app = ApplicationBuilder().token(BOT_TOKEN).build()
app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.CONTACT, handle_contact))
app.add_handler(MessageHandler(filters.TEXT, handle_message))
app.run_polling()
[requirements.txt.txt](https://github.com/user-attachments/files/24309362/requirements.txt.txt)
