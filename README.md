![image](https://github.com/user-attachments/assets/7ee48940-df63-434f-aa60-d2f27eb110f1)

뉴스의 감정분석을 통한 코인 , 주식 예측 파이썬
### NLTK(Natural Language Toolkit) 설명

**NLTK (Natural Language Toolkit)**는 자연어 처리를 위한 파이썬 라이브러리입니다. 텍스트 처리 및 분석을 위한 다양한 도구와 리소스를 제공하며, 주로 텍스트 데이터에서 유용한 정보를 추출하거나 분석하는 데 사용됩니다. 주로 사용되는 기능으로는 텍스트 토큰화, 품사 태깅, 감정 분석, 문법 분석, 그리고 다양한 말뭉치(corpus)와 도구들이 포함됩니다.

### 주요 기능:
1. **토큰화(Tokenization)**: 텍스트를 단어나 문장 단위로 분리합니다.
2. **품사 태깅(Part-of-Speech tagging)**: 각 단어의 품사를 식별합니다.
3. **감정 분석(Sentiment Analysis)**: 텍스트의 감정을 분석하여 긍정적/부정적/중립적 감정을 분류합니다.
4. **어휘 분석(Lemmatization)**: 단어의 기본형(lemma)을 추출합니다.
5. **문법 분석(Parse Trees)**: 문장의 문법적 구조를 분석합니다.

감정 분석에서 사용되는 `SentimentIntensityAnalyzer`는 NLTK에서 제공하는 도구로, 텍스트에서 긍정적, 부정적, 중립적인 감정을 분석하고, 그 결과를 수치로 표현해줍니다.

### GitHub README 작성 예시

아래는 전체 코드를 설명하는 GitHub README용 문서입니다. 코드의 목적, 설치 방법, 사용법 등을 포함하여 프로젝트의 개요를 잘 전달할 수 있습니다.

---

# Market Sentiment Prediction Using News Articles and Prices

## Overview

This project uses news articles related to cryptocurrency, stock prices, and cryptocurrency prices to predict market movements. It scrapes the latest news articles from CNN's cryptocurrency section, analyzes the sentiment of these articles using the NLTK library, and then provides a market prediction based on sentiment. Additionally, it fetches the current prices of stocks and cryptocurrencies to help guide the prediction.

The program will run indefinitely, fetching the latest news and updating stock and cryptocurrency prices at regular intervals (every 10 minutes).

## Key Features
- **News Scraping**: Collects cryptocurrency-related news articles from CNN's website.
- **Sentiment Analysis**: Analyzes the sentiment of news headlines and articles using the NLTK's SentimentIntensityAnalyzer.
- **Stock and Cryptocurrency Prices**: Retrieves current stock prices (e.g., Samsung, Apple, Google) and cryptocurrency prices (e.g., Bitcoin, Ethereum, XRP) using `yfinance` and `ccxt` libraries.
- **Market Prediction**: Based on sentiment analysis, it predicts whether the market will go up, down, or remain neutral.

## Requirements

Before you begin, make sure you have the following libraries installed:

- **NLTK**: Natural Language Toolkit, for text processing and sentiment analysis.
- **yfinance**: For fetching stock market data.
- **ccxt**: For fetching cryptocurrency prices from exchanges like Binance.
- **BeautifulSoup**: For web scraping and parsing HTML content.
- **Requests**: For making HTTP requests to fetch web data.

To install the required libraries, run the following command:

```bash
pip install nltk yfinance ccxt beautifulsoup4 requests
```

## How It Works

### News Scraping
The program scrapes the cryptocurrency news from CNN’s business section. It extracts both the headline and the article body to analyze the sentiment.

### Sentiment Analysis
Sentiment analysis is performed using NLTK's `SentimentIntensityAnalyzer`. It scores the sentiment of the news article headlines and body, providing a "compound" score that ranges from -1 (negative) to 1 (positive). This sentiment score helps in predicting market movements.

### Market Prediction
Using the sentiment score:
- If the sentiment score is positive (`> 0.1`), the program predicts the market will go **up**.
- If the sentiment score is negative (`< -0.1`), the program predicts the market will go **down**.
- If the sentiment score is neutral (`between -0.1 and 0.1`), it predicts **no significant movement** in the market.

### Stock and Cryptocurrency Prices
The program fetches current stock prices using Yahoo Finance (`yfinance`) and cryptocurrency prices using Binance’s API (`ccxt`).

### Continuous Updates
The program runs in an infinite loop, fetching the latest news and prices every 10 minutes. You can adjust the interval in the code by modifying the `time.sleep(600)` call, where `600` is the number of seconds (10 minutes).

## Code Example

```python
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
```

## Running the Program
1. Clone this repository:
    ```bash
    git clone https://github.com/yourusername/market-sentiment-prediction.git
    ```
2. Navigate to the project directory:
    ```bash
    cd market-sentiment-prediction
    ```
3. Install the required dependencies:
    ```bash
    pip install -r requirements.txt
    ```
4. Run the program:
    ```bash
    python main.py
    ```

The program will start fetching news, analyzing sentiment, retrieving stock and cryptocurrency prices, and then predict market movement every 10 minutes.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

이 README는 프로젝트의 목적과 설치 방법, 사용법을 명확히 설명하며, 코드를 실행하기 위한 지침을 제공합니다.
