import requests
from bs4 import BeautifulSoup
import streamlit as st

# TTL=600 tarkoittaa, että data haetaan Averiosta vain kerran 10 minuutissa.
# Tämä on tärkeää, jotta Averion palvelin ei estä (bannaa) meitä liiallisesta liikenteestä.
@st.cache_data(ttl=600)
def hae_oikeat_laivat_averiosta():
    url = "https://averio.fi/laivat"
    
    # Valekäyttäjä-agentti (User-Agent): Kertoo Averion palvelimelle, että 
    # olemme tavallinen selain, emmekä haitallinen botti.
    headers = {
        'User-Agent': 'Mozilla/5.0 (iPhone; CPU iPhone OS 16_0 like Mac OS X) AppleWebKit/605.1.15'
    }
    
    laivat = []
    
    try:
        # Haetaan sivu netistä
        vastaus = requests.get(url, headers=headers)
        vastaus.raise_for_status() # Tarkistaa, ettei tullut virhettä (esim. 404)
        
        # Jäsennellään HTML-koodi BeautifulSoupilla
        soup = BeautifulSoup(vastaus.text, 'html.parser')
        
        # HUOM: 'laiva-rivi' on esimerkki. Tähän pitää vaihtaa Averion oikea HTML-luokka.
        # Tämä etsii kaikki laivat sivulta.
        laiva_elementit = soup.find_all('div', class_='laiva-rivi') 
        
        # Jos emme heti löydä oikeaa luokkaa, palautetaan turvaverkko-dataa:
        if not laiva_elementit:
            return [
                {"ship": "Live-haku käynnissä...", "port": "Odottaa HTML-tarkennusta", "time": "-", "pax": "?"}
            ]
            
        for elementti in laiva_elementit[:3]: # Otetaan vain 3 ensimmäistä (seuraavaa)
            nimi = elementti.find('h3').text.strip()
            aika = elementti.find('span', class_='aika').text.strip()
            pax = elementti.find('span', class_='matkustajat').text.strip()
            
            laivat.append({
                "ship": nimi,
                "port": "Länsisatama T2", # Voidaan myös hakea sivulta
                "time": aika,
                "pax": pax
            })
            
        return laivat

    except Exception as e:
        # Jos nettiyhteys pätkii tai sivu on alhaalla, agentti ei kaadu vaan näyttää virheen.
        return [{"ship": f"Virhe haussa: {e}", "port": "-", "time": "-", "pax": "-"}]

# Näin sitä kutsuttaisiin app.py -koodissa:
# ships = hae_oikeat_laivat_averiosta()
