# Python-stock-fundamental-analyst

import os
import requests
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
from dotenv import find_dotenv, load_dotenv
from newsapi import NewsApiClient
from rich.console import Console
from rich.markdown import Markdown
import yfinance as yf  # for historical stock prices and technical analysis

# Add your API keys here
NEWS_API = "YOUR NEWS API KEY"
AV_API = "YOUR ALPHAVANTAGE KEY"

TICKER = 'AAPL'  # Change the symbol for your analysis

# Fetch stock data using yfinance
stock_data = yf.download(TICKER, start='2022-01-01', end='2023-01-01')

# Technical Analysis
stock_data['MA_50'] = stock_data['Close'].rolling(window=50).mean()
stock_data['MA_200'] = stock_data['Close'].rolling(window=200).mean()

# Financials
companyOverview = requests.get(f'https://www.alphavantage.co/query?function=OVERVIEW&symbol={TICKER}&apikey={AV_API}').json()
incomeStmt = requests.get(f'https://www.alphavantage.co/query?function=INCOME_STATEMENT&symbol={TICKER}&apikey={AV_API}').json()
balanceSheet = requests.get(f'https://www.alphavantage.co/query?function=BALANCE_SHEET&symbol={TICKER}&apikey={AV_API}').json()
cashflow = requests.get(f'https://www.alphavantage.co/query?function=CASH_FLOW&symbol={TICKER}&apikey={AV_API}').json()

# News and Sentiment
newsapi = NewsApiClient(api_key=NEWS_API)
news_object = newsapi.get_everything(q=TICKER, from_param='2023-09-15', language='en', sort_by='relevancy')
all_articles = pd.DataFrame.from_records(news_object['articles'])

newsAndSentiment = requests.get(f'https://www.alphavantage.co/query?function=NEWS_SENTIMENT&symbol={TICKER}&apikey={AV_API}').json()
newsdf = pd.DataFrame.from_records(newsAndSentiment['feed'])

# Display overall sentiment score
overall_sentiment_score = np.mean(newsdf['overall_sentiment_score'])
console = Console()

# Display Technical Analysis
fig, ax = plt.subplots(figsize=(15, 5))
ax.plot(stock_data.index, stock_data['Close'], label='Closing Price')
ax.plot(stock_data.index, stock_data['MA_50'], label='50-day MA')
ax.plot(stock_data.index, stock_data['MA_200'], label='200-day MA')
ax.set_title("Stock Price and Moving Averages")
ax.set_xlabel("Date")
ax.set_ylabel("Price (USD)")
ax.legend()
plt.show()

# Display Financials
fig, axes = plt.subplots(nrows=2, ncols=2, figsize=(15, 10))

# Plot Income Statement
axes[0, 0].plot(annual_incomes.index, annual_incomes.operatingIncome, label='Annual Operating Income', c='g')
axes[0, 0].set_title("Annual Operating Income")
axes[0, 0].set_ylabel("USD")
axes[0, 0].legend()

# Plot Balance Sheet
axes[0, 1].bar(date_list, annual_balances['totalAssets'], width=width, label='Total Assets', color='b')
axes[0, 1].bar(date_list, annual_balances['totalLiabilities'], width=width, label='Total Liabilities', color='r', alpha=0.7)
axes[0, 1].set_title("Total Assets vs Total Liabilities")
axes[0, 1].set_ylabel("USD")
axes[0, 1].legend()

# Plot Cash Flow
axes[1, 0].bar(annual_cashflow.index, annual_cashflow['profitLoss'], label='Profit/Loss')
axes[1, 0].set_title("Profit/Loss in Cash Flow")
axes[1, 0].set_ylabel("USD")
axes[1, 0].legend()

# Display News Sentiment
axes[1, 1].bar(['Overall Sentiment'], [overall_sentiment_score], color='purple')
axes[1, 1].set_title("Overall News Sentiment")
axes[1, 1].set_ylabel("Sentiment Score")
plt.tight_layout()
plt.show()

# Display Buy/Sell Signals
MARKDOWN = f"""
P/E Ratio: {pe_ratio} \t PEG Ratio: {peg_ratio} \t Beta: {beta}
# Signal 1: {'BUY (50 day Moving Average is above 200 day)' if MA_50 > MA_200 else 'SELL (50 day Moving Average is below 200 day)'}
# Signal 2: {'BUY' if signals > 0.5 else 'SELL'}
# Signal 3: {'BUY' if overall_sentiment_score > 0.15 else 'HOLD' if -0.15 < overall_sentiment_score < 0.15 else 'SELL'}
"""

md = Markdown(MARKDOWN)
console.print(md)

