import pandas as pd
import pandas_ta as ta
import time
from webull import webull
from ta.momentum import RSIIndicator
from ta.trend import MACD   # Moving Average Convergence Divergence

# Webull login information
email = 'your_email@example.com'
password = 'your_password'
webull_app = webull()

def login_webull(email, password):
    try:
        webull_app.login(email, password)
    except Exception as e:
        print('Error logging in:', e)
        return False
    return True

def get_historical_data(stock, interval, count):
    try:
        data = webull_app.get_bars(stock, interval, count)
        df = pd.DataFrame(data)
        df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
    except Exception as e:
        print('Error fetching historical data:', e)
        return None
    return df

def calculate_rsi(df, period=14):
    rsi_indicator = RSIIndicator(df['close'], period)
    return rsi_indicator.rsi()

def calculate_macd(df, short_period=12, long_period=26, signal_period=9):
    macd_indicator = MACD(df['close'], short_period, long_period, signal_period)
    return macd_indicator.macd(), macd_indicator.macd_signal()

def check_buy_signal(df):
    latest_data = df.iloc[-1]
    if latest_data['rsi'] < 30 and latest_data['macd'] > latest_data['macd_signal']:
        return True
    return False

def place_buy_order(stock, quantity, limit_price):
    try:
        # Get stock details
        stock_details = webull_app.get_stock(stock)
        stock_id = stock_details['tickerId']

        # Prepare order details
        order_details = {
            'action': 'BUY',
            'orderType': 'LMT',
            'outsideRegularTradingHour': True,  # Set to True for trading during extended hours
            'price': limit_price,
            'quantity': quantity,
            'tickerId': stock_id,
            'timeInForce': 'GTC'  # Good-Till-Cancelled order
        }

        # Place the order
        order_response = webull_app.place_order(stock=stock, order=order_details)

    except Exception as e:
        print('Error placing buy order:', e)
        return None

    return order_response

def main():
    if not login_webull(email, password):
        print('Failed to log in. Exiting.')
        return

    stock = 'AAPL'  # Specify the stock you want to trade
    interval = 'm1'  # Set the interval for the data (m1 = 1-minute data)
    count = 390 * 5  # Number of historical data points to fetch (390 minutes per trading day, 5 days)

    while True:
        df = get_historical_data(stock, interval, count)
        if df is None:
            print('Failed to fetch historical data. Retrying in 1 hour.')
            time.sleep(60 * 60)
            continue

        df['rsi'] = calculate_rsi(df)
        df['macd'], df['macd_signal'] = calculate_macd(df)

        if check_buy_signal(df):
            print('Buy signal detected for', stock)

            # Specify order details
            quantity = 1  # Number of shares to buy
            limit_price = df.iloc[-1]['close']  # Use the latest close price as the limit price

            # Place the buy order
            order_response = place_buy_order(stock, quantity, limit_price)
            print('Buy order placed:', order_response)

        time.sleep(60*60*24)  # Check for buy signal once a day

if __name__ == '__main__':
    main()
