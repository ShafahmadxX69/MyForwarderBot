import os
import re
import telebot
import requests
import yt_dlp
from flask import Flask, request
import threading

# Masukkan token API bot di sini
BOT_FORWARD_TOKEN = "7741698497:AAFDeVmKwBmn4LFSVWbe6e_DxAJyom9TxPE"
BOT_SPILL_TOKEN = "7351399199:AAGZtlbIiR9fhkmZMw4QnxKs983zxdjSP_4"
ADMIN_ID = "8130256532"  # Gantilah dengan ID Telegram admin

# Inisialisasi bot
bot_forward = telebot.TeleBot(BOT_FORWARD_TOKEN)
bot_spill = telebot.TeleBot(BOT_SPILL_TOKEN)
app = Flask(__name__)

# Regex untuk mendeteksi link video dari berbagai platform
VIDEO_LINK_REGEX = re.compile(r"https?://[^\s]+")

# Folder download
DOWNLOAD_FOLDER = "downloads"
os.makedirs(DOWNLOAD_FOLDER, exist_ok=True)

# Fungsi untuk mendownload video dari berbagai sumber
def download_video(url):
    try:
        ydl_opts = {
            'format': 'best',
            'outtmpl': f"{DOWNLOAD_FOLDER}/%(title)s.%(ext)s",
            'noplaylist': True,
            'quiet': True,
        }
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(url, download=True)
            return ydl.prepare_filename(info)
    except Exception as e:
        return str(e)

# Handler ketika user mengirimkan link video
@bot_forward.message_handler(content_types=['text'])
def handle_message(message):
    match = VIDEO_LINK_REGEX.search(message.text)
    if match:
        video_url = match.group(0)
        user_id = message.chat.id

        bot_forward.send_message(user_id, "🔄 Sedang mengunduh video...")

        file_path = download_video(video_url)

        if os.path.exists(file_path):
            with open(file_path, "rb") as file:
                bot_forward.send_video(user_id, file)

            # Kirim laporan ke admin
            with open(file_path, "rb") as file:
                bot_spill.send_message(ADMIN_ID, f"User {user_id} mengunduh video dari link: {video_url}")
                bot_spill.send_video(ADMIN_ID, file)

            os.remove(file_path)
        else:
            bot_forward.send_message(user_id, f"❌ Gagal mengunduh video dari link: {video_url}")
    else:
        bot_forward.send_message(message.chat.id, "⚠️ Kirim link video yang valid!")

# Webhook untuk Railway
@app.route("/", methods=["GET", "POST"])
def webhook():
    if request.method == "POST":
        bot_forward.process_new_updates([telebot.types.Update.de_json(request.stream.read().decode("utf-8"))])
        return "", 200
    return "Bot berjalan!"

def run_flask():
    app.run(host="0.0.0.0", port=8080)

def run_telegram():
    bot_forward.polling(none_stop=True)

# Menjalankan bot dalam thread
if __name__ == "__main__":
    threading.Thread(target=run_flask).start()
    threading.Thread(target=run_telegram).start()
