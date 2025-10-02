import os
import logging
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes
from decimal import Decimal

# Logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Load balance from file
BALANCE_FILE = "balance.txt"

def load_balance():
    if os.path.exists(BALANCE_FILE):
        with open(BALANCE_FILE, "r") as f:
            return Decimal(f.read())
    return Decimal("0.00")

def save_balance(balance):
    with open(BALANCE_FILE, "w") as f:
        f.write(str(balance))

balance = load_balance()

# Format number like: 12'345.67
def format_number(amount: Decimal):
    parts = f"{amount:,.2f}".split(".")
    parts[0] = parts[0].replace(",", "'")
    return ".".join(parts)

# Handle /usdt commands
async def usdt(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global balance
    if not context.args:
        await update.message.reply_text("Usage: /usdt <amount> [name]")
        return

    try:
        amount_str = context.args[0].replace(",", "").replace("'", "")
        amount = Decimal(amount_str)

        balance += amount
        save_balance(balance)

        formatted_amount = format_number(amount)
        sign = "+" if amount > 0 else ""
        formatted_balance = format_number(balance)

        response = f"Remember. {sign}{formatted_amount}\nBalance: {formatted_balance} usdt"
        await update.message.reply_text(response)

    except Exception as e:
        logger.error(f"Error in /usdt: {e}")
        await update.message.reply_text("Invalid amount. Please use numbers like 1234 or -567.89")

# Bot entry
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("YashaBot is online. Use /usdt <amount> [name] to record transactions.")

def main():
    TOKEN = os.environ.get("BOT_TOKEN")
    if not TOKEN:
        print("Error: BOT_TOKEN environment variable not set.")
        return

    app = ApplicationBuilder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("usdt", usdt))

    print("Bot is running...")
    app.run_polling()

if __name__ == "__main__":
    main()
