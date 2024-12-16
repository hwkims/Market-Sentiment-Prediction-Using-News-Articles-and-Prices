import requests
from bs4 import BeautifulSoup
import nltk
from nltk.sentiment import SentimentIntensityAnalyzer
import yfinance as yf
import ccxt
import time

# News Scraping Function
def get_news():
    url = 'https://edition.cnn.com/business/cryptocurrency'
    response = requests.get(url)
    soup = BeautifulSoup(response.text, 'html.parser')

    articles = []
    for article in soup.find_all('h3', class_='cnn-search__result-headline'):
        title = article.get_text()
        link = article.find('a')['href']
        
        # Fetch article content
        full_url = 'https://edition.cnn.com' + link
        article_response = requests.get(full_url)
        article_soup = BeautifulSoup(article_response.text, 'html.parser')
        
        article_body = ' '.join([p.get_text() for p in article_soup.find_all('div', class_='l-container')])
        
        articles.append((title, article_body))

    return articles

# Sentiment Analysis Function
def analyze_sentiment(news_articles):
    nltk.download('vader_lexicon')
    sia = SentimentIntensityAnalyzer()

    sentiment_score = 0
    for title, body in news_articles:
        sentiment_score += sia.polarity_scores(title + " " + body)['compound']

    return sentiment_score / len(news_articles) if len(news_articles) > 0 else 0

# Function to Get Stock Prices
def get_stock_prices(symbols):
    stock_prices = {}
    for symbol in symbols:
        stock = yf.Ticker(symbol)
        data = stock.history(period="1d", interval="5m")
        stock_prices[symbol] = data['Close'].iloc[-1]
    return stock_prices

# Function to Get Cryptocurrency Prices
def get_crypto_prices(symbols):
    exchange = ccxt.binance()
    crypto_prices = {}
    for symbol in symbols:
        ticker = exchange.fetch_ticker(symbol)
        crypto_prices[symbol] = ticker['last']
    return crypto_prices

# Market Prediction Function
def predict_market_impact(stock_symbols, crypto_symbols):
    print("Fetching cryptocurrency news...")
    news_articles = get_news()
    sentiment = analyze_sentiment(news_articles)

    print(f"\nSentiment score for recent cryptocurrency news: {sentiment:.2f}")

    print("\nFetching stock prices...")
    stock_prices = get_stock_prices(stock_symbols)
    for symbol, price in stock_prices.items():
        print(f"Current stock price for {symbol}: {price:.2f} USD")

    print("\nFetching cryptocurrency prices...")
    crypto_prices = get_crypto_prices(crypto_symbols)
    for symbol, price in crypto_prices.items():
        print(f"Current cryptocurrency price for {symbol}: {price:.2f} USD")

    if sentiment > 0.1:
        print("\nPrediction: Expecting markets to go up.")
    elif sentiment < -0.1:
        print("\nPrediction: Expecting markets to go down.")
    else:
        print("\nPrediction: Market sentiment is neutral. No significant movement expected.")

# Infinite Loop for Continuous Updates
if __name__ == "__main__":
    stock_symbols = ['005930.KS', 'AAPL', 'GOOG']  # Samsung, Apple, Google
    crypto_symbols = ['BTC/USDT', 'ETH/USDT', 'XRP/USDT']  # Bitcoin, Ethereum, XRP

    while True:
        predict_market_impact(stock_symbols, crypto_symbols)
        print("\nWaiting for next update...\n")
        time.sleep(600)  # Wait for 10 minutes before the next iteration
