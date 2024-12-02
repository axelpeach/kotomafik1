import os
from flask import Flask, request
from telegram import Bot, Update
from telegram.ext import Dispatcher, CommandHandler
import random

# Ініціалізація Flask
app = Flask(__name__)

# Бот-токен із секретів середовища
TOKEN = os.getenv("TELEGRAM_TOKEN")
bot = Bot(token=TOKEN)

# Dispatcher для обробки команд
dispatcher = Dispatcher(bot, None, workers=0)

# Дані користувачів
user_data = {}


# Команда /murr — помурчати
def murr(update: Update, context):
    user_id = update.effective_user.id
    username = update.effective_user.first_name

    if user_id not in user_data:
        user_data[user_id] = {'murr_count': 0, 'whiskers_length': 0.0}

    user_data[user_id]['murr_count'] += 1
    murr_count = user_data[user_id]['murr_count']

    update.message.reply_text(f"{username} помурчав. Всього мурчань: {murr_count}.")


# Команда /whiskers — вирощування вусів
def whiskers(update: Update, context):
    user_id = update.effective_user.id
    username = update.effective_user.first_name

    if user_id not in user_data:
        user_data[user_id] = {'murr_count': 0, 'whiskers_length': 0.0}

    change = round(random.uniform(-7, 7), 2)
    user_data[user_id]['whiskers_length'] += change
    total_length = round(user_data[user_id]['whiskers_length'], 2)

    result = f"{username} вай магьошь твой усік {'сталь больше' if change > 0 else 'сталь меньше'} на {change} мм. Загальна довжина: {total_length} мм."
    update.message.reply_text(result)


# Реєстрація команд
dispatcher.add_handler(CommandHandler("murr", murr))
dispatcher.add_handler(CommandHandler("usik", whiskers))


# Обробка запитів від Telegram
@app.route(f"/{TOKEN}", methods=["POST"])
def webhook():
    update = Update.de_json(request.get_json(force=True), bot)
    dispatcher.process_update(update)
    return "OK", 200


# Endpoint для UptimeRobot
@app.route("/")
def index():
    return "Бот працює!", 200


if __name__ == "__main__":
    # Налаштування вебхука
    PORT = int(os.environ.get("PORT", 5000))
    bot.set_webhook(f"https://{os.getenv('RENDER_EXTERNAL_HOSTNAME')}/{TOKEN}")
    app.run(host="0.0.0.0", port=PORT)
