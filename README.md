import telebot
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton
from texts import texts
import json

bot = telebot.TeleBot("TOKEN")

user_data = {}

def load_data():
    global user_data
    try:
        with open("data.json", "r") as f:
            user_data = json.load(f)
    except:
        user_data = {}

def save_data():
    with open("data.json", "w") as f:
        json.dump(user_data, f)

def get_lang(user_id):
    return user_data.get(str(user_id), {}).get("lang", "az")

def set_lang(user_id, lang):
    if str(user_id) not in user_data:
        user_data[str(user_id)] = {}
    user_data[str(user_id)]["lang"] = lang
    save_data()

@bot.message_handler(commands=["start"])
def start(message):
    user_id = message.from_user.id
    lang = get_lang(user_id)
    username = message.from_user.first_name
    markup = InlineKeyboardMarkup(row_width=2)
    markup.add(*[InlineKeyboardButton(btn, callback_data=f"menu_{i}") for i, btn in enumerate(texts[lang]["start_buttons"])])
    photo_url = "https://files.catbox.moe/7ehdoh.jpg"
    bot.send_photo(message.chat.id, photo=photo_url)
    bot.send_message(message.chat.id, texts[lang]["start"].format(username=username, botname="ŞansBot"), reply_markup=markup)

@bot.callback_query_handler(func=lambda call: call.data.startswith("menu_"))
def handle_menu(call):
    user_id = call.from_user.id
    lang = get_lang(user_id)
    index = int(call.data.split("_")[1])

    if index == 0:  # Oyuna Başla
        game_markup = InlineKeyboardMarkup(row_width=2)
        game_markup.add(*[InlineKeyboardButton(btn, callback_data=f"game_{i}") for i, btn in enumerate(texts[lang]["game_menu"])])
        bot.send_message(call.message.chat.id, "🎮 Oyunu seç:", reply_markup=game_markup)

    elif index == 1:  # Bot haqqında
        bot.send_message(call.message.chat.id, texts[lang]["about"].format(username=call.from_user.first_name, botname="ŞansBot"))

    elif index == 2:  # Dil seçimi
        lang_markup = InlineKeyboardMarkup()
        for l in texts:
            lang_markup.add(InlineKeyboardButton(texts[l]["language_buttons"][0], callback_data=f"lang_{l}"))
        bot.send_message(call.message.chat.id, texts[lang]["language"], reply_markup=lang_markup)

@bot.callback_query_handler(func=lambda call: call.data.startswith("lang_"))
def set_language(call):
    lang = call.data.split("_")[1]
    set_lang(call.from_user.id, lang)
    bot.answer_callback_query(call.id, "✅ Dil seçildi")
    start(call.message)
texts = {
    "az": {
        "start": """🎉 Xoş gəldin, {username} şans ovçusu! 🎰

Mən {botname} 🤖

Burada sən öz bəxtini müxtəlif oyunlarla sınaya bilərsən:

🎲 Zar at — bəxtinə nə düşəcək?  
🌀 Təkəri fırlat — sürpriz mükafatlar gözləyir!  
🎁 Qutulardan birini seç — bəlkə də jackpot sənin qismətindir!  
🍒 Slot maşını — 3 simvolu tut, qalib ol!  

💰 Qazandığın xallar yığılır və lider cədvəlində yerini təyin edir. 
🧠 Əmr: /topxal ilə liderlər siyahısına bax!

/help yazaraq bot necə işləyir örgən

Hazırsansa, düymələrdən birini seç və şansa meydan oxu! 🔥
""",
        "language": "🌐 Dil Seçin",
        "about": """Salam {username}, mən {botname} şans oyununa başlamağa hazırsan 🎉

Əgər sən bu oyundan aşağıdakı xalları qazansan Sahibim sənə 🌟 hədiyyə edəcək:

3000 xal → 25🌟  
6000 xal → 50🌟  
10000 xal → 75🌟

Qazandıqdan sonra Sahibimə yaz: @Helios_077
Ulduz almaq üçün ss ilə sübut yetər.

Uğurlar! 💘""",
        "start_buttons": ["🎮 Oyuna Başla", "ℹ️ Bot haqqında", "🌐 Dil", "👤 Bot Sahibi: @Helios_077"],
        "language_buttons": ["🇦🇿 Azərbaycan", "🇹🇷 Türk", "🇬🇧 English", "🇷🇺 Русский", "🇺🇿 O‘zbek", "🇹🇯 Тоҷикӣ"],
        "game_menu": ["🍒 Slot Maşını", "🎲 Zar At", "🌀 Təkəri Fırlat", "🎁 Qutu Seç"]
    },
    ...
}

# game logikası buradan sonra gələcək...

load_data()
bot.polling()
