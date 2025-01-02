import json
import requests
from websocket import create_connection
import time

# Constants
WS_URL = 'wss://wss.nobitex.ir/connection/websocket'
TG_BOT_TOKEN = '8157042543:AAHSjhGQqsbNCeIi6nokhAh1iLyTMBNH2jU'
TG_CHANNEL = '@arz_updates'
SYMBOLS = [
    {"symbol": "USDTIRT", "title": "دلار", "unit": "تومن", "factor": 0.1},
    {"symbol": "BTCIRT", "title": "بیتکوین", "unit": "تومن", "factor": 0.1},
    {"symbol": "BTCUSDT", "title": "بیتکوین", "unit": "دلار", "factor": 1},
]

# Key-Value storage simulation
kv_store = {}

def save_price_to_kv(key, price):
    kv_store[key] = price

def get_last_price_from_kv(key):
    return kv_store.get(key, None)

def send_to_telegram(messages):
    tg_api_url = f'https://api.telegram.org/bot{TG_BOT_TOKEN}/sendMessage'
    body = {
        'chat_id': TG_CHANNEL,
        'text': '\n-----------------------\n'.join(messages)
    }
    try:
        response = requests.post(tg_api_url, json=body)
        if response.status_code != 200:
            print('Failed to send message to Telegram:', response.text)
    except Exception as e:
        print('Error sending message to Telegram:', e)

def fetch_price(symbol, title, unit, factor):
    try:
        ws = create_connection(WS_URL)
        print(f"Connected to WebSocket for {symbol}.")

        # Send connection message
        ws.send(json.dumps({
            "connect": {"name": "py"},
            "id": 3
        }))

        # Send subscription message
        ws.send(json.dumps({
            "subscribe": {
                "channel": f"public:orderbook-{symbol}",
                "recover": True,
                "offset": 0,
                "epoch": "0",
                "delta": "fossil"
            },
            "id": 4
        }))

        while True:
            message = ws.recv()
            data = json.loads(message)

            if data.get("id") == 4 and "subscribe" in data and "publications" in data["subscribe"]:
                publication = data["subscribe"]["publications"][0]
                if publication and "data" in publication:
                    parsed_data = json.loads(publication["data"])
                    if parsed_data.get("asks"):
                        current_price = float(parsed_data["asks"][0][0]) * factor

                        last_price = get_last_price_from_kv(symbol)
                        trend = ''
                        if last_price is not None:
                            if current_price > last_price:
                                trend = '🟢'
                            elif current_price < last_price:
                                trend = '🔴'
                            else:
                                trend = '⚪️'

                        save_price_to_kv(symbol, current_price)
                        formatted_price = f"{int(current_price):,}"
                        ws.close()
                        return f"{trend} {title}: {formatted_price} {unit}"

    except Exception as e:
        print(f"Error fetching price for {symbol}: {e}")
        return None

def main():
    messages = []
    for symbol_info in SYMBOLS:
        symbol = symbol_info["symbol"]
        title = symbol_info["title"]
        unit = symbol_info["unit"]
        factor = symbol_info["factor"]
        
        message = fetch_price(symbol, title, unit, factor)
        if message:
            messages.append(message)
        time.sleep(1)  # To avoid rate limits

    if messages:
        send_to_telegram(messages)

if __name__ == "__main__":
    main()
