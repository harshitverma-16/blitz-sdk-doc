---
hide:
  - toc
---

# **Blitz** module

Blitz SDK for Python.   
```python
pip install blitzSDK
```

## **The library**
Blitz provides a comprehensive set of APIs for building trading and investment applications. With Blitz, you can place orders instantly, monitor portfolios, access live market data through WebSockets, and leverage a variety of trading features using a simple, unified HTTP interface.

## **Getting started**

    from blitzSDK import AuthClient, MarketDataClient

    # Step 1: Authenticate
    auth_client = AuthClient(app_key="your_app_key_here", user_id="USER123")

    # Step 2: Generate Access Token
    access_token = auth_client.get_access_token()
    print("Access Token:", access_token)

    # Step 3: Initialize Market Data Client
    market_client = MarketDataClient(app_key="your_app_key_here", user_id="USER123")

    # Step 4: Fetch LTP for instruments
    ltp_data = market_client.get_ltp(["NSE:RELIANCE", "NSE:TCS"])
    print("LTP Data:", ltp_data)


**Output**
```python
Access Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
LTP Data: {'NSE:RELIANCE': 2448.25, 'NSE:TCS': 3575.10}
```

---

## **Class**

```
class AuthClient
```

## **Methods**

### **```__init__```**
```
def __init__(self, app_key: str, user_id: str)
```
??? info "Source code"
    ```python
    def __init__(self, app_key: str, user_id: str):
        self.app_key = app_key
        self.user_id = user_id
        self.auth_base_url = "http://182.70.127.254:44347"
        self.access_token = None
    ```

Authentication of the client.

**Parameters**

| Parameter | Type | Required | Description |
|------------|------|-----------|--------------|
| `app_key` | `string` | Yes | Unique key provided to the client for API authentication. |
| `user_id` | `string` | Yes | Unique identifier of the client or account used for login. |

**Example:**
```python
auth = AuthClient(app_key="abc123xyz", user_id="U12345")
```

---

### **```get_access_token```**
```
def get_access_token(self):
```
??? info "Source Code"
    ```python
    def get_access_token(self):
        """Return the access token, logging in if necessary."""
        if not self.access_token:
            self._app_login()
        return self.access_token
    ```

Return the access token of the client

---

## **Class**

```
class MarketDataClient
```

The `MarketDataClient` class provides access to **live** and **historical market data** through REST and WebSocket APIs.  
It manages authentication, connection setup, and streaming of real-time tick data.

---

## **Methods**

### **```__init__```**
```
def __init__(self, app_key: str, user_id: str)
```
??? info "Source Code"
    ```python
    def __init__(self, app_key: str, user_id: str):
        self.app_key = app_key
        self.user_id = user_id
        self.api_base_url = "http://182.70.127.254:11022/api"
        self.api_hist_base_url = "http://14.99.227.18:11025/md-api/"
        self.token = None 
        self.auth_client = AuthClient(app_key, user_id)
        self.access_token = None

        self._ensure_logged_in()
        self.ws_client = MarketDataWebSocketClient(self.access_token)
        self.ws_client.set_on_connect(self.on_connect)
        self.ws_client.set_on_close(self.on_close)
        self._on_tick = None
    ```

Initializes a new instance of the `MarketDataClient` class.  
This client is responsible for connecting to live and historical market data endpoints and managing WebSocket communication for real-time updates.

**Parameters**

| Parameter | Type | Required | Description |
|------------|------|-----------|--------------|
| `app_key` | `string` | Yes | Unique application key provided to the client during registration. Used for API authentication. |
| `user_id` | `string` | Yes | Unique user identifier assigned by the broker or platform. Used to link API calls to a specific client account. |

**Example:**
```python
auth = MarketDataClient(app_key="abc123xyz", user_id="U12345")
```

---

### **```on_connect```**
```
def on_connect(self)
```
??? info "Source Code"
    ```python
    def on_connect(self):
        """Callback when WebSocket connects."""
        self.ws_client.start()
        logging.info("WebSocket connected successfully.")
    ```

Confirmation when the websocket is connected.

---

### **```on_close```**
```
def on_close(self)
```
??? info "Source Code"
    ```python
    def on_close(self, close_status_code, close_msg):
        """Callback when WebSocket closes."""
        logging.warning(f"WebSocket closed: {close_status_code}, {close_msg}")
    ```

Callback when websocket closes

---

### **```get_ltp```**
```
def get_ltp(self, instrument_ids)
```
??? info "Source Code"
    ```python
    def get_ltp(self, instrument_ids):
        payload = {"InstrumentIds": instrument_ids}
        return self._send_request("/marketfeed/ltp", payload)
    ```

Fetches the **Last Traded Price (LTP)** for one or more instruments from the market data API.

**Parameters**

| Parameter | Type | Required | Description |
|------------|------|-----------|--------------|
| `instrument_ids` | `list` or `array` | Yes | A list of instrument IDs (symbols or tokens) for which the last traded price is to be fetched. |

**Example**

```python
# Example: Get LTP for multiple instruments
ltp_data = client.get_ltp(["NSE:INFY", "NSE:TCS", "NSE:RELIANCE"])

print(ltp_data)
# Example Response:
# {
#   "NSE:INFY": 1620.50,
#   "NSE:TCS": 3575.10,
#   "NSE:RELIANCE": 2448.25
# }
```

---

### **```get_option_chain```**
```
def get_option_chain(self, symbol, expiry_date)
```
??? info "Source Code"
    ```python
    def get_option_chain(self, symbol, expiry_date):
        payload = {"symbol": symbol, "expiryDate": expiry_date}
        return self._send_request("/marketfeed/optionChain", payload)
    ```

Retrieves the **Option Chain** data for a given underlying symbol and expiry date.  
This includes all available strike prices along with their corresponding call (CE) and put (PE) option details.

**Parameters**

| Parameter | Type | Required | Description |
|------------|------|-----------|--------------|
| `symbol` | `string` | Yes | The underlying asset symbol (e.g., `NIFTY`, `BANKNIFTY`, or an equity symbol). |
| `expiry_date` | `string` | Yes | Expiry date of the options contract in `YYYY-MM-DD` format. |


**Example**


```python
# Example: Get Option Chain for NIFTY
option_chain = client.get_option_chain(symbol="NIFTY", expiry_date="2025-10-31")

print(option_chain)
# Example Response:
# {
#   "symbol": "NIFTY",
#   "expiryDate": "2025-10-31",
#   "data": [
#       {
#           "strikePrice": 20000,
#           "CE": {
#               "ltp": 120.5,
#               "bid": 119.8,
#               "ask": 121.0,
#               "volume": 5400,
#               "openInterest": 152300
#           },
#           "PE": {
#               "ltp": 95.3,
#               "bid": 94.8,
#               "ask": 95.6,
#               "volume": 4600,
#               "openInterest": 138200
#           }
#       },
#       ...
#   ]
# }
```

---

### **```get_quote```**
```
def get_quote(self, instrument_ids)
```
Retrieves **detailed market quotes** for one or more instruments.  
This method provides complete quote information — including last traded price, volume, and other key market data fields.

**Parameters**

| Parameter | Type | Required | Description |
|------------|------|-----------|--------------|
| `instrument_ids` | `list` or `array` | Yes | A list of instrument IDs (symbols or tokens) for which the full market quote is to be fetched. |

**Example**

```python
# Example: Get full quote details for instruments
quote_data = client.get_quote(["NSE:TCS"])

print(quote_data)
# Example Response:
# {
#   "NSE:TCS": {
#       "ltp": 3575.1,
#       "bid": 3574.5,
#       "ask": 3576.2,
#       "volume": 182300,
#       "open": 3550.0,
#       "high": 3588.0,
#       "low": 3540.0,
#       "close": 3560.0
#   }
# }
```

---

### **```connect_ws```**
```
def connect_ws(self)
```
??? info "Source Code"
    ```python
    def connect_ws(self):
        """Start WebSocket if not already connected."""
        if not self._is_connected():
            self.ws_client.start()
            time.sleep(2) 
        else:
            logging.info("WebSocket already connected.")
    ```

- Start the websocket if not started.

---

### **```Stop_websocket```**
```
def stop_websocket(self)
```
??? info "Source Code"
    ```python
    def stop_websocket(self):
        """Stop the WebSocket connection."""
        if self.ws_client:
            self.ws_client.stop()
            logging.info("WebSocket stopped.")
    ```

- Stop the websocket connection

---

### **```subscribe_market_data```**
```
def subscribe_market_data(self, instrument_ids)
```
??? info "Source Code"
    ```python
    def subscribe_market_data(self, instrument_ids):
        """Subscribe to live market data."""
        pass
    ```


Subscribes to **live market data** for one or more instruments through a WebSocket connection.  
Once subscribed, the client will start receiving **real-time tick updates** for the specified instruments.

**Parameters**

| Parameter | Type | Required | Description |
|------------|------|-----------|--------------|
| `instrument_ids` | `list` or `array` | Yes | A list of instrument IDs (symbols or tokens) to subscribe to for live market data updates. |

**Example**

```python
# Example: Subscribe to live market data
client.subscribe_market_data(["NSE:RELIANCE", "NSE:TCS", "NSE:INFY"])
```

---

### **```unsubscribe_market_data```**
```
def unsubscribe_market_data(self, instrument_ids)
```
??? info "Source Code"
    ```python
    def unsubscribe_market_data(self, instrument_ids):
        """Unsubscribe from market data through WebSocket."""
        if self._is_connected():
            self.ws_client.unsubscribe(instrument_ids)
            logging.info(f"Unsubscribed from instrument IDs: {instrument_ids}")
        else:
            logging.error("WebSocket is not connected. Cannot unsubscribe.")
    ```

Unsubscribes from **live market data** for one or more instruments through the active WebSocket connection.  
Once unsubscribed, the client will **stop receiving tick updates** for the specified instruments.

**Parameters**

| Parameter | Type | Required | Description |
|------------|------|-----------|--------------|
| `instrument_ids` | `list` or `array` | Yes | A list of instrument IDs (symbols or tokens) to unsubscribe to for live market data updates. |

**Example**

```python
# Example: Unubscribe to live market data
client.unsubscribe_market_data(["NSE:RELIANCE", "NSE:TCS", "NSE:INFY"])
```

---

### **```get_historical_data```**
```
def get_historical_data(self, instrument: str, from_date: str, to_date: str)
```
??? info "Source Code"
    ```python
    def get_historical_data(self, instrument, from_date, to_date):
        """Fetch historical market data for a specific instrument and date range."""
        payload = {
            "instrument": instrument,
            "from": from_date,
            "to": to_date
        }
        return self._send_request("/marketfeed/historicalData", payload)
    ```

Fetches **historical market data** (OHLC – Open, High, Low, Close) for a specific instrument over a given date range.  
This method is useful for backtesting, analytics, and visualizing past market trends.

**Parameters**

| Parameter | Type | Required | Description |
|------------|------|-----------|--------------|
| `instrument` | `string` | Yes | The instrument ID or symbol for which historical data is to be fetched. |
| `from_date` | `string` | Yes | Start date of the historical data range in `YYYY-MM-DD` format. |
| `to_date` | `string` | Yes | End date of the historical data range in `YYYY-MM-DD` format. |

**Example**

```python
# Example: Fetch historical data for RELIANCE
historical_data = client.get_historical_data(
    instrument="NSE:RELIANCE",
    from_date="2025-09-01",
    to_date="2025-09-15"
)

print(historical_data)
# Example Response:
# {
#   "instrument": "NSE:RELIANCE",
#   "data": [
#       {
#           "date": "2025-09-01",
#           "open": 2450.0,
#           "high": 2470.0,
#           "low": 2435.0,
#           "close": 2462.5,
#           "volume": 1250000
#       },
#       {
#           "date": "2025-09-02",
#           "open": 2463.0,
#           "high": 2488.0,
#           "low": 2450.5,
#           "close": 2475.8,
#           "volume": 1178000
#       },
#       ...
#   ]
# }
```