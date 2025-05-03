# 🚀 BTC Trade Alerts

Monitoramento automatizado do preço do Bitcoin com alertas via WhatsApp.  
Utiliza níveis de Fibonacci, RSI e Média Móvel para sugerir momentos ideais de compra.

## 🔧 Tecnologias utilizadas
- Python
- Twilio API (para envio de mensagens WhatsApp)
- API CoinGecko (para dados do Bitcoin)
- Pandas, NumPy
- TA-Lib (via `ta`)
- Matplotlib (opcional)
- Requests

## 📲 Alerta no WhatsApp
Você receberá um alerta **automatizado no WhatsApp** quando:
- O preço atual estiver abaixo do nível de 23.6% de Fibonacci
- E o RSI estiver abaixo de 30 (sobrevendido)

---

## ⚙️ Como usar

1. Instale as dependências: 
``bash
pip install pandas ta numpy matplotlib requests twilio

2. Insira suas credenciais do Twilio no código:
account_sid = 'SUA_SID_AQUI'
auth_token = 'SEU_AUTH_TOKEN_AQUI'
from_whatsapp_number = 'whatsapp:+14155238886'
to_whatsapp_number = 'whatsapp:+55SEUNUMEROAQUI' 

3. Execute o script:
python main.py

📁 Código-fonte:
# btc-trade-alerts
Monitoramento automatizado do preço do Bitcoin com alertas via WhatsApp. Utiliza níveis de Fibonacci para sugerir momentos ideais de compra.

import smtplib
from twilio.rest import Client
import requests
import time
import datetime
import pandas as pd
import ta  # Biblioteca para indicadores técnicos
import numpy as np
import matplotlib.pyplot as plt

# Twilio configurations
account_sid = ''
auth_token = ''
from_whatsapp_number = 'whatsapp:+14155238886'  # Número Twilio WhatsApp
to_whatsapp_number = 'whatsapp:+5511917099910'  # Seu número pessoal

client = Client(account_sid, auth_token)

def enviar_whatsapp(message):
    try:
        message = client.messages.create(
            body=message,
            from_=from_whatsapp_number,
            to=to_whatsapp_number
        )
        print("Mensagem enviada com sucesso!")
    except Exception as e:
        print(f"Erro ao enviar WhatsApp: {e}")

def obter_dados_bitcoin():
    url = "https://api.coingecko.com/api/v3/coins/bitcoin/market_chart"
    params = {
        'vs_currency': 'usd',
        'days': '182',
        'interval': 'daily'
    }
    response = requests.get(url, params=params)
    data = response.json()
    return data['prices']

def calcular_fibonacci(preco_maximo, preco_minimo):
    niveis_fibonacci = {
        '0%': preco_minimo,
        '23.6%': preco_minimo + (preco_maximo - preco_minimo) * 0.236,
        '38.2%': preco_minimo + (preco_maximo - preco_minimo) * 0.382,
        '50%': preco_minimo + (preco_maximo - preco_minimo) * 0.5,
        '61.8%': preco_minimo + (preco_maximo - preco_minimo) * 0.618,
        '100%': preco_maximo
    }
    return niveis_fibonacci

def calcular_indicadores_tecnicos(dados):
    df = pd.DataFrame(dados, columns=['timestamp', 'preco'])
    df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
    df['rsi'] = ta.momentum.RSIIndicator(df['preco'], window=14).rsi()
    df['sma_30'] = ta.trend.SMAIndicator(df['preco'], window=30).sma_indicator()
    return df

def analisar_compra(fibonacci, sma_30, rsi, preco_atual):
    analise = "⏳ Ainda não é um momento ideal para compra. Acompanhe os indicadores."
    if preco_atual < fibonacci['23.6%'] and rsi < 30:
        analise = "✅ Momento bom para compra! Preço baixo e RSI abaixo de 30 (indicado como sobrevendido). Compre Bitcoin."
    return analise

def monitorar_bitcoin():
    print(f"Iniciando monitoramento de Bitcoin... {datetime.datetime.now()}")
    dados = obter_dados_bitcoin()
    preco_atual = dados[-1][1]
    preco_maximo = max([item[1] for item in dados])
    preco_minimo = min([item[1] for item in dados])
    fibonacci = calcular_fibonacci(preco_maximo, preco_minimo)
    df = calcular_indicadores_tecnicos(dados)
    sma_30 = df['sma_30'].iloc[-1]
    rsi = df['rsi'].iloc[-1]
    analise = analisar_compra(fibonacci, sma_30, rsi, preco_atual)

    print(f"📊 Preço atual do Bitcoin: ${preco_atual} ({datetime.datetime.now()})")
    print(f"📌 Máximo histórico analisado: ${preco_maximo}")
    print(f"📌 Mínimo histórico analisado: ${preco_minimo}")
    print(f"📈 Níveis de Fibonacci: {fibonacci}")
    print(f"📉 Indicadores Técnicos:")
    print(f"- Média Móvel 30 dias (SMA_30): ${sma_30}")
    print(f"- RSI (14 dias): {rsi}")
    print(f"\n🔍 Análise: {analise}")

    if "✅" in analise:
        enviar_whatsapp(analise)

def countdown():
    for i in range(60, 0, -1):
        print(f"🔄 Atualizando em: {i} segundos", end="\r")
        time.sleep(1)

enviar_whatsapp("✅ O monitoramento do Bitcoin está funcionando! Você será notificado no WhatsApp quando for o momento ideal para compra. Fique de olho!")

while True:
    monitorar_bitcoin()
    countdown()
