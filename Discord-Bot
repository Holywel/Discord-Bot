import discord
from discord.ext import commands
import requests
from bs4 import BeautifulSoup
import urllib.parse
import re
import os
from http.server import BaseHTTPRequestHandler, HTTPServer
import threading

class KeepAliveHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b"Bot 7/24 Aktif!")

def run_keep_alive():
    server = HTTPServer(('0.0.0.0', 10000), KeepAliveHandler)
    server.serve_forever()

intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix="!", intents=intents)

HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36'
}

def get_player_data(player_name):
    try:
        search_query = urllib.parse.quote_plus(f"{player_name} transfermarkt")
        search_url = f"https://html.duckduckgo.com/html/?q={search_query}"
        
        search_response = requests.get(search_url, headers=HEADERS)
        if search_response.status_code != 200:
            return None
            
        search_soup = BeautifulSoup(search_response.content, 'html.parser')
        player_profile_url = None
        
        for a_tag in search_soup.find_all("a", class_="result__url"):
            href = a_tag.get("href", "")
            actual_url = urllib.parse.unquote(href)
            if "transfermarkt.com" in actual_url and "/profil/" in actual_url:
                match = re.search(r'(https://www\.transfermarkt\.com/[^&]+)', actual_url)
                if match:
                    player_profile_url = match.group(1)
                    break
                    
        if not player_profile_url:
            return None
            
        response = requests.get(player_profile_url, headers=HEADERS)
        if response.status_code != 200:
            return None
            
        soup = BeautifulSoup(response.content, 'html.parser')
        
        full_name_element = soup.find("h1")
        if full_name_element:
            full_name = full_name_element.get_text(" ", strip=True)
            full_name = re.sub(r'#\d+\s*', '', full_name)
        else:
            full_name = player_name.title()
        
        market_value_div = soup.select_one("a.data-header__market-value-wrapper")
        if market_value_div:
            market_value = market_value_div.get_text(strip=True).split("Last")[0].strip()
            if market_value.startswith("€"):
                market_value = market_value.replace("€", "").strip() + " €"
        else:
            market_value = "Bilinmiyor"
        
        club_span = soup.select_one("span.data-header__club a")
        club = club_span.get_text(strip=True) if club_span else "Kulüpsüz"

        club_logo_url = None
        if club_span and "href" in club_span.attrs:
            club_href = club_span["href"]
            club_id_match = re.search(r'verein/(\d+)', club_href)
            if club_id_match:
                club_id = club_id_match.group(1)
                club_logo_url = f"https://tmssl.akamaized.net/images/wappen/verylarge/{club_id}.png"

        return {
            "name": full_name,
            "value": market_value,
            "club": club,
            "logo": club_logo_url,
            "url": player_profile_url
        }
    except Exception as e:
        print(f"Hata oluştu: {e}")
        return None

@bot.event
async def on_ready():
    print(f'Bot aktif! Giriş yapılan hesap: {bot.user.name}')

@bot.command(name="deger")
async def deger_sorgula(ctx, *, oyuncu_ismi: str):
    proper_name = oyuncu_ismi.title()
    await ctx.send(f"🔍 `{proper_name}` Transfermarkt üzerinde aranıyor, lütfen bekleyin...")
    
    player_info = get_player_data(oyuncu_ismi)
    
    if not player_info:
        await ctx.send("❌ Oyuncu bulunamadı veya veri çekme hatası oluştu. Lütfen ismi doğru yazdığınızdan emin olun.")
        return
        
    embed = discord.Embed(
        title=f"🏆 {player_info['name']}",
        url=player_info["url"],
        description="Transfermarkt Güncel Oyuncu Verileri",
        color=discord.Color.green()
    )
    
    if player_info["logo"]:
        embed.set_thumbnail(url=player_info["logo"])
        
    embed.add_field(name="💰 Piyasa Değeri", value=f"**{player_info['value']}**", inline=False)
    embed.add_field(name="🛡️ Güncel Kulüp", value=f"**{player_info['club']}**", inline=False)
    
    await ctx.send(embed=embed)

if __name__ == "__main__":
    threading.Thread(target=run_keep_alive, daemon=True).start()
    bot.run(os.getenv('DISCORD_TOKEN'))

