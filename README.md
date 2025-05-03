"""
🚀 **BTC Trade Alerts** - Monitoramento Automatizado de Bitcoin via WhatsApp 📈
📝 Autor: Leonardo
📃 Descrição: Script que monitora o preço do Bitcoin, analisa níveis de Fibonacci, RSI e Média Móvel para identificar boas oportunidades de compra e envia alertas no WhatsApp quando necessário. 🔔

"""

# === 🌐 IMPORTAÇÕES E CONFIGURAÇÕES ===

import requests  # Requisição de dados
import pandas as pd  # Manipulação de dados
import ta  # Bibliotecas para análise técnica
import numpy as np  # Manipulação de arrays
import time  # Temporizador
import datetime  # Para pegar as datas e horários atuais
from twilio.rest import Client  # Para enviar mensagens via WhatsApp

# === CONFIGURAÇÕES DO USUÁRIO 📲 ===
# ⚙️ Insira suas credenciais do Twilio abaixo:
account_sid = 'SUA_ACCOUNT_SID'  # 🔑 Sua SID do Twilio
auth_token = 'SEU_AUTH_TOKEN'    # 🔑 Seu Auth Token do Twilio
from_whatsapp_number = 'whatsapp:+14155238886'  # 🚪 Número do Twilio (deve ser o mesmo do seu Sandbox)
to_whatsapp_number = 'whatsapp:+55SEUNUMEROAQUI'  # 📱 Seu número de WhatsApp com DDD (ex: +5511999999999)

# =================

# Inicializando o cliente Twilio 💬
client = Client(account_sid, auth_token)

# Função para enviar mensagem no WhatsApp 🚀
def enviar_whatsapp(mensagem):
    try:
        msg = client.messages.create(
            body=mensagem,
            from_=from_whatsapp_number,
            to=to_whatsapp_number
        )
        print("✅ Mensagem enviada com sucesso!")
    except Exception as e:
        print(f"❌ Erro ao enviar mensagem: {e}")

# Função para obter os dados históricos do Bitcoin (últimos 182 dias) 💰
def obter_dados_bitcoin():
    url = "https://api.coingecko.com/api/v3/coins/bitcoin/market_chart"
    params = {'vs_currency': 'usd', 'days': '182', 'interval': 'daily'}
    response = requests.get(url, params=params)
    data = response.json()
    return data['prices']  # Retorna a lista de [timestamp, price]

# Função para calcular os níveis de Fibonacci 🔢
def calcular_fibonacci(preco_max, preco_min):
    return {
        '0%': preco_min,
        '23.6%': preco_min + (preco_max - preco_min) * 0.236,
        '38.2%': preco_min + (preco_max - preco_min) * 0.382,
        '50%': preco_min + (preco_max - preco_min) * 0.5,
        '61.8%': preco_min + (preco_max - preco_min) * 0.618,
        '100%': preco_max
    }

# Função para calcular os indicadores técnicos (RSI e SMA de 30 dias) 📊
def calcular_indicadores(dados):
    df = pd.DataFrame(dados, columns=['timestamp', 'preco'])
    df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
    df['rsi'] = ta.momentum.RSIIndicator(df['preco'], window=14).rsi()  # RSI (Índice de Força Relativa)
    df['sma_30'] = ta.trend.SMAIndicator(df['preco'], window=30).sma_indicator()  # Média Móvel de 30 dias
    return df

# Função para analisar a oportunidade de compra 🛒
def analisar_compra(fib, sma_30, rsi, preco_atual):
    if preco_atual < fib['23.6%'] and rsi < 30:
        return "📢 **Oportunidade de compra detectada!** 🟢\nPreço abaixo do nível de 23.6% de Fibonacci e RSI abaixo de 30 (sobrevendido)."
    else:
        return "🔍 **Monitoramento ativo**. Nenhuma oportunidade clara de compra agora."

# Função principal de monitoramento 📍
def monitorar_bitcoin():
    print(f"⏱️ **Iniciando verificação**: {datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    
    dados = obter_dados_bitcoin()  # Obtém os dados históricos do Bitcoin
    preco_atual = dados[-1][1]  # Preço mais recente
    maximo = max(x[1] for x in dados)  # Preço máximo histórico
    minimo = min(x[1] for x in dados)  # Preço mínimo histórico

    fibonacci = calcular_fibonacci(maximo, minimo)  # Calcula Fibonacci
    df = calcular_indicadores(dados)  # Calcula RSI e SMA
    rsi = df['rsi'].iloc[-1]  # Último valor de RSI
    sma_30 = df['sma_30'].iloc[-1]  # Último valor da SMA de 30 dias

    print(f"📉 Preço atual: ${preco_atual:.2f}")
    print(f"📈 RSI: {rsi:.2f} | SMA 30 dias: ${sma_30:.2f}")
    print(f"🔢 Fibonacci 23.6%: ${fibonacci['23.6%']:.2f}")

    resultado = analisar_compra(fibonacci, sma_30, rsi, preco_atual)  # Análise de compra
    print(f"📊 **Resultado da análise**: {resultado}")

    if "Oportunidade de compra" in resultado:
        enviar_whatsapp(resultado)  # Envia alerta no WhatsApp

# Mensagem inicial de monitoramento 🚀
enviar_whatsapp("📡 **Monitoramento de preço do Bitcoin iniciado!** Você receberá alertas quando uma boa oportunidade de compra for detectada.")

# Loop contínuo para atualizações a cada 60 segundos ⏳
while True:
    monitorar_bitcoin()
    for i in range(60, 0, -1):
        print(f"⌛ **Próxima atualização em {i}s**", end='\r')
        time.sleep(1)
