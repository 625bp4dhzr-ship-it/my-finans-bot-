# my-finans-bot-
bot.py
import os
import logging
from datetime import datetime
from telegram import Update
from telegram.ext import ApplicationBuilder, ContextTypes, MessageHandler, filters

logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)

user_balances = {}

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.message.from_user.id
    
    if user_id not in user_balances:
        user_balances[user_id] = {"kirim_1": 0.0, "kirim_2": 0.0, "history": []}

    text_to_process = ""

    if update.message.voice:
        text_to_process = "1 150000"
        await update.message.reply_text("🎙 Ovozli xabar qabul qilindi! (Test: 1-kirimga 150,000 qo'shildi)")
    else:
        text_to_process = update.message.text

    now = datetime.now().strftime("%Y-%m-%d %H:%M")

    try:
        parts = text_to_process.strip().split()
        category = int(parts[0])
        amount = float(parts[1])

        if category == 1:
            user_balances[user_id]["kirim_1"] += amount
            cat_name = "1-Kirim (Asosiy)"
        elif category == 2:
            user_balances[user_id]["kirim_2"] += amount
            cat_name = "2-Kirim (Qo'shimcha)"
        else:
            await update.message.reply_text("❌ Xato! Faqat `1` yoki `2` raqamini yozing. Masalan: `1 300000`")
            return

        user_balances[user_id]["history"].append(f"[{now}] {cat_name}: +{amount:,.0f} so'm")

        k1 = user_balances[user_id]["kirim_1"]
        k2 = user_balances[user_id]["kirim_2"]
        total = k1 + k2

        response = (
            f"✅ **Muvaffaqiyatli qo'shildi!**\n"
            f"🕒 Vaqt: {now}\n\n"
            f"📥 **1-Kirim:** {k1:,.0f} so'm\n"
            f"📥 **2-Kirim:** {k2:,.0f} so'm\n"
            f"---------------------------\n"
            f"💰 **Umumiy summa:** {total:,.0f} so'm"
        )
        await update.message.reply_text(response, parse_mode="Markdown")

    except Exception:
        await update.message.reply_text(
            "⚠️ Xato format! Iltimos, quyidagicha yuboring:\n"
            "`[Kirim raqami] [Summa]`\n"
            "Misol: `1 500000` (1-kirimga 500 ming) yoki `2 150000`",
            parse_mode="Markdown"
        )

if __name__ == '__main__':
    TOKEN = "SIZNING_BOT_TOKENINGIZNI_SHUYERGA_YOZING"
    
    app = ApplicationBuilder().token(TOKEN).build()
    app.add_handler(MessageHandler(filters.TEXT | filters.VOICE & (~filters.COMMAND), handle_message))
    
    print("🤖 Bot ishga tushdi...")
    app.run_polling()
