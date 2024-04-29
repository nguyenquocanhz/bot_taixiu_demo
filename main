import random
import json
import os
import telebot
from telebot import types

bot = telebot.TeleBot("TOKEN")

# Check if the user_data.json file exists, if not, create it
if not os.path.exists("user_data.json"):
    with open("user_data.json", "w") as file:
        json.dump([], file)

# Load user data from JSON file
with open("user_data.json", "r") as file:
    user_data = json.load(file)

# Function to save user data to JSON file
def save_user_data():
    with open("user_data.json", "w") as file:
        json.dump(user_data, file)

# Function to start the game
def start_game(message):
    bot.send_message(message.chat.id, "Chọn mức cược:")
    bot.register_next_step_handler(message, select_bet_amount)

# Function to select the bet amount
def select_bet_amount(message):
    user_id = message.from_user.id
    try:
        bet_amount = int(message.text)
        if bet_amount > 0:
            # Check if the user has enough balance
            if user_data["user_id"]["balance"] >= bet_amount:
                # Proceed with the game
                user_data["user_id"]["balance"] -= bet_amount  # Deduct bet amount from balance
                save_user_data()  # Save user data
                # Implement game logic here
                bot.send_message(message.chat.id, f"Bạn đã chọn mức cược {bet_amount}.")
                tai_xiu(message)
            else:
                bot.send_message(message.chat.id, "Số dư không đủ. Vui lòng chọn mức cược nhỏ hơn hoặc bằng số dư hiện tại.")
        else:
            bot.send_message(message.chat.id, "Mức cược phải là một số nguyên dương. Vui lòng nhập lại.")
            bot.register_next_step_handler(message, select_bet_amount)  # Ask user to input bet amount again
    except ValueError:
        bot.send_message(message.chat.id, "Vui lòng nhập một số nguyên dương.")
        bot.register_next_step_handler(message, select_bet_amount)  # Ask user to input bet amount again

# Function for the game "Tài xỉu"
def tai_xiu(message):
    markup = types.ReplyKeyboardMarkup(row_width=2)
    markup.add(types.KeyboardButton("Tài"), types.KeyboardButton("Xỉu"))
    bot.send_message(message.chat.id, "Chọn Tài hoặc Xỉu:", reply_markup=markup)
    bot.register_next_step_handler(message, generate_tai_xiu_result)

# Function to generate the result for "Tài xỉu"
def generate_tai_xiu_result(message):
    user_choice = message.text.lower()
    result = random.randint(1, 6) + random.randint(1, 6) + random.randint(1, 6)
    if result <= 10:
        result_text = "Xỉu"
    else:
        result_text = "Tài"
    if user_choice == result_text.lower():
        bot.send_message(message.chat.id, f"Kết quả: {result}\nChúc mừng, bạn đã thắng!")
        # Update user balance here if they win
    else:
        bot.send_message(message.chat.id, f"Kết quả: {result}\nRất tiếc, bạn đã thua.")

@bot.message_handler(commands=["start"])
def start(message):
    global user_data
    user_id = message.from_user.id
    # Check if user data exists for the user ID
    if not any(data["user_id"] == user_id for data in user_data):
        # If not, create new user data and append to the list
        new_user_data = {"user_id": user_id, "name": "", "balance": 0}
        user_data.append(new_user_data)
        save_user_data()  # Save user data
    else:
        # Find the index of the user data for the given user_id
        user_index = next((index for index, data in enumerate(user_data) if data["user_id"] == user_id), None)
        # Check if user's chat ID exists, if yes, do not save again
        if user_index is not None and "chat_id" in user_data[user_index]:
            return
    # Save chat ID to user data
    user_index = next((index for index, data in enumerate(user_data) if data["user_id"] == user_id), None)
    if user_index is not None:
        user_data[user_index]["chat_id"] = message.chat.id
    bot.reply_to(message, f"Xin chào {message.from_user.first_name}!\n"
                          f"Tài khoản của bạn:\n"
                          f" - Tên: {user_data[user_index]['name']}\n"
                          f" - Số dư: {user_data[user_index]['balance']}")
    # Ask the user to set their name if it's not set
    if not user_data[user_index]["name"]:
        bot.send_message(message.chat.id, "Vui lòng đặt tên của bạn bằng cách gửi tin nhắn với nội dung /setname Tên của bạn")

@bot.message_handler(commands=["setname"])
def set_name(message):
    global user_data
    user_id = message.from_user.id
    if len(message.text.split()) > 1:
        name = " ".join(message.text.split()[1:])
        # Check if the username already exists
        if any(data["name"].lower() == name.lower() for data in user_data):
            bot.reply_to(message, "Tên đã tồn tại. Vui lòng chọn tên khác.")
        else:
            # Find the index of the user data for the given user_id
            user_index = next((index for index, data in enumerate(user_data) if data["user_id"] == user_id), None)
            if user_index is not None:
                user_data[user_index]["name"] = name
                save_user_data()  # Save user data
                bot.reply_to(message, f"Tên của bạn đã được cập nhật thành {name}")
    else:
        bot.reply_to(message, "Vui lòng nhập tên sau lệnh /setname")

@bot.message_handler(commands=["naptien"])
def request_reload_amount(message):
    bot.send_message(message.chat.id, "Vui lòng nhập số tiền bạn muốn nạp.")
    bot.register_next_step_handler(message, reload_balance)

def reload_balance(message):
    global user_data
    user_id = message.from_user.id
    try:
        reload_amount = int(message.text)
        if reload_amount > 0:
            # Find the index of the user data for the given user_id
            user_index = next((index for index, data in enumerate(user_data) if data["user_id"] == user_id), None)
            if user_index is not None:
                user_data[user_index]["balance"] += reload_amount
                save_user_data()  # Save user data
                bot.reply_to(message, f"Số dư của bạn đã được nạp thêm {reload_amount}")
        else:
            bot.reply_to(message, "Số tiền nạp phải là một số nguyên dương.")
    except ValueError:
        bot.reply_to(message, "Vui lòng nhập một số nguyên dương.")

@bot.message_handler(commands=["gameslist"])
def games_list(message):
    bot.send_message(message.chat.id, "Danh sách các trò chơi:\n"
                                      "1. Tài xỉu\n"
                                      "2. Đoán số\n"
                                      "3. Sấp ngửa\n"
                                      "4. Xổ số\n"
                                      "5. Lô đề")
    start_game(message)  # Prompt user to select game after sending the game list

# Other handlers...

bot.polling()
