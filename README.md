import logging
import asyncio
from aiogram import Bot, Dispatcher, types
from aiogram.types import Message
from aiogram.utils import executor

# Konfigurasi Bot
API_TOKEN = "7748807929:AAFCN3G4dJ7YMYZDu9iydC6yniK_F2kl9R8"
DATABASE_CHAT_ID = "@MilioMilioOhyeah"

bot = Bot(token=API_TOKEN)
dp = Dispatcher(bot)

# Logging
logging.basicConfig(level=logging.INFO)

@dp.message_handler(commands=['start'])
async def start(message: Message):
    await message.reply("Give me the links")

@dp.message_handler()
async def process_link(message: Message):
    link = message.text.strip()
    if not link.startswith("https://t.me/"):
        await message.reply("Invalid link. Please provide a valid Telegram file link.")
        return
    
    try:
        # Mengunduh file dari link
        file = await bot.get_file_by_link(link)
        file_path = file.file_path
        
        # Kirim file ke grup backup (Database)
        await bot.send_document(DATABASE_CHAT_ID, file_path, caption="Backup File")
        
        # Kirim file ke user
        await bot.send_document(message.chat.id, file_path, caption="Here is your file")
    except Exception as e:
        logging.error(f"Error processing file: {e}")
        await message.reply("Failed to retrieve file. Ensure the link is correct and try again.")

if __name__ == '__main__':
    executor.start_polling(dp, skip_updates=True)
