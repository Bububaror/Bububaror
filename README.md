import telebot
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton, CallbackQuery

API_TOKEN = '7484490077:AAGfd-4PJekERnP_ZaLlIgWrUVqA-DjoqlA'
CHANNEL_USERNAME = '@birthdaystatusdaily'  # Replace with your channel's username without quotes

bot = telebot.TeleBot(API_TOKEN)

# Remove any existing webhook (important for avoiding conflict errors)
bot.remove_webhook()

# Start command handler
@bot.message_handler(commands=['start'])
def start(message):
    user_id = message.from_user.id
    if is_user_subscribed(user_id):
        bot.send_message(user_id, "✅ Welcome! You are subscribed to the channel. You can now use the bot's features. \nThis bot helps you to find specific videos.")
    else:
        send_subscription_prompt(user_id)

# Function to check if user is subscribed or an admin
def is_user_subscribed(user_id):
    try:
        member_status = bot.get_chat_member(CHANNEL_USERNAME, user_id).status
        return member_status in ['member', 'administrator', 'creator']
    except Exception as e:
        print(f"Error checking subscription status: {e}")
        return False

# Function to send subscription prompt with verification button
def send_subscription_prompt(user_id):
    markup = InlineKeyboardMarkup()
    subscribe_button = InlineKeyboardButton(
        text='📢 Subscribe to Channel', 
        url=f'https://t.me/{CHANNEL_USERNAME.strip("@")}'
    )
    check_button = InlineKeyboardButton(
        text='✅ I have Subscribed', 
        callback_data='check_subscription'
    )
    markup.add(subscribe_button)
    markup.add(check_button)
    bot.send_message(
        user_id,
        "🔔 To use this bot, you need to subscribe to our channel. Please subscribe and then click 'I have Subscribed'.",
        reply_markup=markup
    )

# Callback handler for subscription check
@bot.callback_query_handler(func=lambda call: call.data == 'check_subscription')
def callback_check_subscription(call: CallbackQuery):
    user_id = call.from_user.id
    if is_user_subscribed(user_id):
        bot.answer_callback_query(call.id, "🎉 Thank you for subscribing!")
        bot.send_message(user_id, "✅ You have successfully subscribed. You can now use the bot's features. \nThis bot helps you to find specific videos.")
        send_video_link(user_id, f'https://t.me/{CHANNEL_USERNAME.strip("@")}', '🎥 Get Video')
    else:
        bot.answer_callback_query(call.id, "❌ You are not subscribed yet. Please subscribe to the channel first.")

# Function to send the video link button based on user input
def send_video_link(user_id, link, button_text='🎥 Get Video'):
    markup = InlineKeyboardMarkup()
    video_button = InlineKeyboardButton(
        text=button_text, 
        url=link
    )
    markup.add(video_button)
    bot.send_message(
        user_id,
        "🎬 Click the button below to download or watch the video:",
        reply_markup=markup
    )

# Handler for all other text messages
@bot.message_handler(func=lambda message: True)
def handle_all_messages(message):
    user_id = message.from_user.id
    if is_user_subscribed(user_id):
        if message.text == '/1':
            send_video_link(user_id, 'https://t.me/birthdaystatusdaily/8', '🎥 Get Video')
        elif message.text == '/2':
            send_video_link(user_id, 'https://t.me/birthdaystatusdaily/2', '🎥 Get Video')
        else:
            send_video_link(user_id, f'https://t.me/{CHANNEL_USERNAME.strip("@")}', '🎥 Get Video')
    else:
        send_subscription_prompt(user_id)

# Start the bot
if __name__ == '__main__':
    print("Bot is running...")
    bot.infinity_polling()
