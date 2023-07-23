import pandas as pd
import yfinance as yf
from datetime import datetime
import plotly.express as px

# Setting start date and end date for the stocks that we want to analyze
start_date = datetime.now() - pd.DateOffset(months=3)
end_date = datetime.now()

tickers = ['AAPL', 'MSFT', 'NFLX', 'GOOG']

df_list = []
for ticker in tickers:
    data = yf.download(ticker, start=start_date, end=end_date)
    df_list.append(data)

df = pd.concat(df_list, keys=tickers, names=['Ticker', 'Date'])

print(df.head())
print(df.tail())

df = df.reset_index()
print(df.head())
print(df.tail())

img = px.line(df,x='Date',
              y='Close',
              color = 'Ticker',
              title = 'Stocks performance')
img.show()


img_1 = px.area(df,x='Date',y='Close',
                color = 'Ticker',
                facet_col = 'Ticker',
                labels = {'Date':'Date','Close':'Closing value',"Ticker":'Company'},
                title = 'Stock Prices')

img_1.show()
img_1.show()

df['MAverage10'] = df.groupby("Ticker")['Close'].rolling(window=10).mean().reset_index(0,drop = True)
df['MAverage20'] = df.groupby("Ticker")['Close'].rolling(window=20).mean().reset_index(0,drop = True)

for ticker , group in df.groupby('Ticker'):
  print(f'Moving Average for {ticker}:')
  print(group[['MAverage10','MAverage20']])

for ticker, groupin in df.groupby("Ticker"):
  img_3 = px.line(group, x = 'Date',y=['Close','MAverage10','MAverage20'],title = f'Moving average of all the stocks {ticker}')
  img_3.show()

df['Volatility'] = df.groupby("Ticker")['Close'].pct_change().rolling(windows = 10).std().reset_index(0,drop = True)
img_4 = px.line(df , x='Date',y = 'Volatility',
                color = "Ticker",
                title = 'Stock of Volatity')
img_4.show()

apple = df.loc[df['Ticker'] == 'AAPL',['Date','Close']].rename(columns = {'Close':'AAPL'})
microsoft = df.loc[df['Ticker'] == 'MSFT',['Date','Close']].rename(columns = {'Close':'MSFT'})

df_correlation = pd.merge(apple,microsoft)

img_5 = px.scatter(df_correlation, x= 'AAPL', y = 'MSFT',
                   trendline = 'ewm',
                   title = 'Correlation between aplle and microsft')
