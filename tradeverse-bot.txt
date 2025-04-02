# نصب پکیج‌های مورد نیاز:
# pip install python-telegram-bot beautifulsoup4 requests

import requests
from bs4 import BeautifulSoup
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, ContextTypes

TOKEN = "7579475210:AAHKlIujveCnOmc1YB3HxKA8IKDSgpoCikc"

# گرفتن قیمت لحظه‌ای رمزارزها از CoinGecko
def get_crypto_prices():
    url = "https://api.coingecko.com/api/v3/simple/price"
    params = {
        "ids": "bitcoin,ethereum,binancecoin,solana,ripple",
        "vs_currencies": "usd",
        "include_24hr_change": "true"
    }
    response = requests.get(url, params=params)
    return response.json()

# فرمت‌دهی قیمت رمزارزها
def format_crypto_message(data):
    def format_price(name, price, change):
        arrow = "🟢" if change >= 0 else "🔴"
        symbol = "▲" if change >= 0 else "▼"
        return f"{arrow} {name}: ${price:,.2f} ({symbol}{abs(change):.2f}%)"

    msg = "⚡️ TradeVerse Market Pulse | نبض بازار ⚡️\n\n📈 قیمت لحظه‌ای رمزارزها:\n"
    msg += format_price("BTC", data['bitcoin']['usd'], data['bitcoin']['usd_24h_change']) + "\n"
    msg += format_price("ETH", data['ethereum']['usd'], data['ethereum']['usd_24h_change']) + "\n"
    msg += format_price("BNB", data['binancecoin']['usd'], data['binancecoin']['usd_24h_change']) + "\n"
    msg += format_price("SOL", data['solana']['usd'], data['solana']['usd_24h_change']) + "\n"
    msg += format_price("XRP", data['ripple']['usd'], data['ripple']['usd_24h_change']) + "\n"
    return msg

# گرفتن قیمت طلا، ارز و سکه از tgju.org
def get_iran_market():
    url = "https://www.tgju.org/"
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')

    def get_price(code):
        try:
            return soup.find("td", {"data-market-row": code}).text.strip()
        except:
            return "نامشخص"

    return {
        "usd": get_price("price_dollar_rl"),
        "eur": get_price("price_eur"),
        "aed": get_price("price_aed"),
        "seke_emami": get_price("sekeb"),
        "gold_18": get_price("geram18"),
        "mesghal": get_price("mesghal")
    }

# فرمت‌دهی پیام بازار ایران
def format_iran_message(data):
    msg = "\n💱 نرخ بازار ایران:\n"
    msg += f"💵 دلار: {data['usd']} تومان\n"
    msg += f"💶 یورو: {data['eur']} تومان\n"
    msg += f"🇦🇪 درهم: {data['aed']} تومان\n"
    msg += "\n🌕 طلا و سکه:\n"
    msg += f"🥇 سکه امامی: {data['seke_emami']} تومان\n"
    msg += f"🥈 طلا ۱۸ عیار: {data['gold_18']} تومان\n"
    msg += f"🥉 مثقال طلا: {data['mesghal']} تومان\n"
    msg += "\n⏱ داده‌ها به‌صورت لحظه‌ای از CoinGecko و tgju.org گرفته شده‌اند."
    return msg

# دستور /market در بات تلگرام
async def market(update: Update, context: ContextTypes.DEFAULT_TYPE):
    crypto_data = get_crypto_prices()
    iran_data = get_iran_market()
    message = format_crypto_message(crypto_data) + format_iran_message(iran_data)
    await update.message.reply_text(message)

# اجرای بات
if __name__ == '__main__':
    app = ApplicationBuilder().token(TOKEN).build()
    app.add_handler(CommandHandler("market", market))
    print("Bot is running...")
    app.run_polling()
