# vinted.botimport requests
import time

WEBHOOK_URL = "https://discord.com/api/webhooks/1387169690849378374/9B3W1pCmsnE5Yqghqk7wLNb8vIuvQqLOWVuJGgbLCJi0ucEJqd7qtxQ-2hGx5g1zphtY"

MARQUES = [
    "Nike", "Ralph Lauren", "Bape", "Stussy", "Adidas", "Tommy Hilfiger",
    "CP Company", "The North Face", "New Balance", "Lacoste", "Stone Island", "Jordan"
]

MAX_PRICE_DEFAULT = 20
MAX_PRICE_VESTE = 30

def send_discord_message(content):
    data = {"content": content}
    response = requests.post(WEBHOOK_URL, json=data)
    if response.status_code == 204:
        print("Message envoyé sur Discord.")
    else:
        print(f"Erreur en envoyant le message : {response.status_code}")

def fetch_vinted_annonces():
    annonces_trouvees = []
    for marque in MARQUES:
        url = f"https://www.vinted.fr/api/v2/catalog/items?search_text={marque}&catalog_category_id=1&brand_id=&price_to={MAX_PRICE_DEFAULT}&currency=EUR&per_page=50&order=newest_first&user_type=0&color_id=&size_id=&item_condition_ids=&page=1&gender=male"
        try:
            response = requests.get(url)
            response.raise_for_status()
            data = response.json()

            for item in data.get("items", []):
                price = item.get("price", {}).get("amount", 0)
                title = item.get("title", "Pas de titre")
                url_item = f"https://www.vinted.fr{item.get('url', '')}"
                is_veste = "veste" in title.lower()

                max_price = MAX_PRICE_VESTE if is_veste else MAX_PRICE_DEFAULT

                if price <= max_price:
                    annonces_trouvees.append(f"**{title}**\nPrix: {price}€\nLien: {url_item}\nMarque: {marque}")

        except Exception as e:
            print(f"Erreur avec la marque {marque}: {e}")

    return annonces_trouvees

def main():
    print("🚀 Bot Vinted Scraping lancé (toutes les 30 secondes)")
    annonces_deja_envoyees = set()

    while True:
        annonces = fetch_vinted_annonces()

        if not annonces:
            print("🔍 0 annonces trouvées sur la page")
        else:
            nouvelles_annonces = [a for a in annonces if a not in annonces_deja_envoyees]

            if nouvelles_annonces:
                for annonce in nouvelles_annonces:
                    send_discord_message(annonce)
                    annonces_deja_envoyees.add(annonce)
                print(f"✅ {len(nouvelles_annonces)} nouvelles annonces envoyées.")
            else:
                print("🔍 Pas de nouvelles annonces à envoyer.")

        print("🔎 Pause de 30 secondes avant la prochaine recherche.")
        time.sleep(30)

if __name__ == "__main__":
    main()
