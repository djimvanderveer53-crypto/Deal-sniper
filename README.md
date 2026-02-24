# Deal-sniper
import requests
import time
from bs4 import BeautifulSoup

TELEGRAM_TOKEN = "8633244691:AAHpz4yWsxGiehCNw0NeCeHsQ8oJgn13AeA"
CHAT_ID = "6411875539"

SEARCH_URL = "https://www.marktplaats.nl/q/iphone/"

seen_ads = set()


def send_telegram(message):
    url = f"https://api.telegram.org/bot{TELEGRAM_TOKEN}/sendMessage"
    data = {"chat_id": CHAT_ID, "text": message}
    requests.post(url, data=data)


def calculate_offer(model, battery, storage, front_crack=False, back_crack=False):
    base_prices = {
        "iphone 12": 75,
        "iphone 13": 125,
        "iphone 14": 200,
        "iphone 15": 275,
        "iphone 16": 375
    }

    model = model.lower()
    if model not in base_prices:
        return 0

    price = base_prices[model]

    if storage == 256:
        price += 15
    elif storage == 512:
        price += 30

    if battery < 85:
        price -= (85 - battery)

    if front_crack:
        price -= 75
    if back_crack:
        price -= 75

    return max(price, 0)


def scan():
    response = requests.get(SEARCH_URL)
    soup = BeautifulSoup(response.text, "html.parser")

    ads = soup.find_all("a")

    for ad in ads:
        link = ad.get("href")
        title = ad.text.lower()

        if link and "iphone" in title:
            full_link = "https://www.marktplaats.nl" + link

            if full_link in seen_ads:
                continue

            seen_ads.add(full_link)

            # simpele detectie
            model = None
            for m in ["iphone 12", "iphone 13", "iphone 14", "iphone 15", "iphone 16"]:
                if m in title:
                    model = m

            if not model:
                continue

            offer = calculate_offer(model, 85, 128)

            message = f"""
🔥 NIEUWE DEAL

Model: {model}
Bod: €{offer}

Link:
{full_link}

Bericht om te sturen:
Hi, ik bied €{offer} en kan vandaag ophalen.
"""

            send_telegram(message)


while True:
    scan()
    time.sleep(60)
