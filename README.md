from telegram import Update, Bot
from telegram.ext import Updater, MessageHandler, Filters, CallbackContext
import logging
import os

# Konfigurasi Logging
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)

# Ganti dengan token bot yang kamu dapat dari BotFather
TOKEN = "7748807929:AAFCN3G4dJ7YMYZDu9iydC6yniK_F2kl9R8"
CHANNEL_ID = "@Media_ForwarderBot"  # atau gunakan ID numerik channel jika private

# Fungsi untuk menangani pesan masuk
def forward_media(update: Update, context: CallbackContext):
    user = update.message.from_user
    file = None
    caption = f"📤 Forwarded by: {user.full_name} (ID: {user.id})"
    
    if update.message.photo:
        file = update.message.photo[-1].file_id
    elif update.message.video:
        file = update.message.video.file_id
    elif update.message.document:
        file = update.message.document.file_id
    elif update.message.audio:
        file = update.message.audio.file_id
    elif update.message.voice:
        file = update.message.voice.file_id
    
    if file:
        context.bot.send_message(chat_id=CHANNEL_ID, text=caption)
        context.bot.send_document(chat_id=CHANNEL_ID, document=file, caption=caption)

# Setup bot
def main():
    updater = Updater(TOKEN, use_context=True)
    dp = updater.dispatcher
    
    dp.add_handler(MessageHandler(Filters.chat_type.groups & Filters.media, forward_media))
    
    updater.start_polling()
    updater.idle()

if __name__ == "__main__":
    main()
