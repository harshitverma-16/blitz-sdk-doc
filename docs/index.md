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
    ltp_data = market_client.get_ltp(["NSE|RELIANCE", "NSE|TCS"])
    print("LTP Data:", ltp_data)
 
 
**Output**
```python
Access Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Downloading instruments...
Loaded 85313 instruments.
INFO:root:App login successful.
Matched → Symbol: NSECM|RELIANCE | Instrument ID: 1010010000002885
Matched → Symbol: NSECM|TCS | Instrument ID: 1010010000011536
LTP Response: {'data': {
    '1010010000002885': {'InstrumentID': 1010010000002885, 'InstrumentName': 'RELIANCE', 'LTP': 1422.7}, '1010010000011536': {'InstrumentID': 1010010000011536, 'InstrumentName': 'TCS', 'LTP': 3507.8}},
    'status': 'success'
    }
```
 
---
 
## **Class: AuthClient**
 
```
class AuthClient
```
 
### **Methods**
 
#### **```__init__```**
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
 
#### **```get_access_token```**
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
 
## **Class: MarketDataClient**
 
```
class MarketDataClient
```
 
The `MarketDataClient` class provides access to **live** and **historical market data** through REST and WebSocket APIs.  
It manages authentication, connection setup, and streaming of real-time tick data.
 
---
 
### **Methods**
 
#### **```__init__```**
```
def __init__(self, app_key: str, user_id: str)
```
??? info "Source Code"
    ```python
    def __init__(self, app_key: str, user_id: str):
        self.app_key = app_key
        self.user_id = user_id
        self.api_base_url = API_BASE_URL
        self.api_hist_base_url = API_BASE_URL
        self.auth_client = AuthClient(app_key, user_id)
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
 
#### **```get_ltp```**
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
ltp_data = client.get_ltp(["NSE|TCS", "NSE|RELIANCE", 1010010002000001])
 
print(ltp_data)
# Example Response:
# LTP Response: {'data': {
#   '1010010000002885': {'InstrumentID': 1010010000002885, 'InstrumentName': 'RELIANCE', 'LTP': 1420.2},       
#   '1010010000011536': {'InstrumentID': 1010010000011536, 'InstrumentName': 'TCS', 'LTP': 3500.3},             
#   '1010010002000001': {'InstrumentID': 1010010002000001, 'InstrumentName': 'NIFTY 50', 'LTP': 24834.3}},  
#   'status':'success'}
```
 
---
 
#### **```get_option_chain```**
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
option_chain = client.get_option_chain(symbol="NIFTY", expiry_date="2025-05-29")
 
print(option_chain)
# Example Response:
# {
#     "status": "success",
#     "data": {
#         "spotPrice": 24813.5,
#         "expiryDate": "2025-05-29",
#         "atm": 24800,
#         "chains": [
#             {
#                 "strikePrice": 23800,
#                 "callOption": {
#                     "gamma": 0,
#                     "vega": 0,
#                     "theta": 0,
#                     "delta": 1,
#                     "oiPercentage": 0,
#                     "oi": 0,
#                     "ltp": 1042.8,
#                     "iv": 0.2,
#                     "price": 1013.5,
#                     "rho": 0
#                 },
#                 "putOption": {
#                     "gamma": 0,
#                     "vega": 0,
#                     "theta": 0,
#                     "delta": 0,
#                     "oiPercentage": 0,
#                     "oi": 0,
#                     "ltp": 8.85,
#                     "iv": 0.2,
#                     "price": 0,
#                     "rho": 0
#                 }
#             },
#             {
#                 "strikePrice": 23850,
#                 "callOption": {
#                     "gamma": 0,
#                     "vega": 0,
#                     "theta": 0,
#                     "delta": 1,
#                     "oiPercentage": 0,
#                     "oi": 0,
#                     "ltp": 990.5,
#                     "iv": 0.2,
#                     "price": 963.5,
#                     "rho": 0
#                 },
#                 "putOption": {
#                     "gamma": 0,
#                     "vega": 0,
#                     "theta": 0,
#                     "delta": 0,
#                     "oiPercentage": 0,
#                     "oi": 0,
#                     "ltp": 10.25,
#                     "iv": 0.2,
#                     "price": 0,
#                     "rho": 0
#                 }
#             },
#             ...
#         ]
#     }
# }
 
```
 
---
 
#### **```get_quote```**
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
quote_data = client.get_quote(["NSECM|RELIANCE", "NSECM|TCS", 1010010002000001])
 
print("Quote Response:", quote_data)
# Example Response:
# Quote Response:
# {
#     'data': {
#         '1010010000002885': {
#             'InstrumentID': 1010010000002885,
#             'ExchangeSegment': 1,
#             'ExchangeInstrumentID': 2885,
#             'InstrumentName': 'RELIANCE',
#             'Timestamp': 1748339153,
#             'LTP': 1419.4,
#             'LTQ': 74,
#             'LTT': 1748339153,
#             'ATP': 1422.41,
#             'VTT': 1651139,
#             'TBQ': 0,
#             'TSQ': 0,
#             'OI': 0,
#             'Open': 1426.1000000000001,
#             'High': 1429.7,
#             'Low': 1418.1000000000001,
#             'Close': 1434.8,
#             'BidLevel': [
#                 {'Qty': 1, 'Price': 1419.3, 'Orders': 1},
#                 {'Qty': 7, 'Price': 1419.2, 'Orders': 7},
#                 {'Qty': 24, 'Price': 1419.1000000000001, 'Orders': 24},
#                 {'Qty': 148, 'Price': 1419, 'Orders': 148},
#                 {'Qty': 11, 'Price': 1418.9, 'Orders': 11}
#             ],
#             'AskLevel': [
#                 {'Qty': 1, 'Price': 1419.4, 'Orders': 1},
#                 {'Qty': 4, 'Price': 1419.5, 'Orders': 4},
#                 {'Qty': 4, 'Price': 1419.6000000000001, 'Orders': 4},
#                 {'Qty': 3, 'Price': 1419.7, 'Orders': 3},
#                 {'Qty': 8, 'Price': 1419.8, 'Orders': 8}
#             ]
#         },
#         ...
#     },
#     'status': 'success'
# }
 
```
 
---
 
#### **```get_historical_data```**
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
    instrument="NSECM|RELIANCE",
    from_date="2025-09-01",
    to_date="2025-09-15"
)
 
print(historical_data)
# Example Response:
# {
#   "instrument": "NSE|RELIANCE",
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
 
---
 
#### **```subscribe_market_data```**
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
client.subscribe_market_data(["NSECM|RELIANCE", "NSECM|TCS", 1010010002000001])
 
# Response:
 
# New tick data received: MessageCode: 1502
# ID: 1010010000002885
# MarketDepthMessage {
#   InstrumentID: 1010010000002885
#   ExchangeSegment: NSECM
#   ExchangeInstrumentID: 2885
#   InstrumentName: "RELIANCE"
#   TimeStamp: 1748339705
#   LTP: 1418.7
#   LTQ: 7
#   LTT: 1748339705
#   ATP: 1421.8700000000001
#   VTT: 1948737
#   Open: 1426.1000000000001
#   High: 1429.7
#   Low: 1417.5
#   Close: 1434.8
#   BestBidLevel {
#     Qty: 270
#     Price: 1418.7
#     Orders: 1
#   }
#   BestBidLevel {
#     Qty: 322
#     Price: 1418.4
#     Orders: 5
#   }
#   BestBidLevel {
#     Qty: 385
#     Price: 1418.3
#     Orders: 7
#   }
#   BestBidLevel {
#     Qty: 1227
#     Price: 1418.2
#     Orders: 10
#   }
#   BestBidLevel {
#     Qty: 1978
#     Price: 1418.1000000000001
#     Orders: 17
#   }
#   BestAskLevel {
#     Qty: 363
#     Price: 1418.8
#     Orders: 2
#   }
#   BestAskLevel {
#     Qty: 379
#     Price: 1418.9
#     Orders: 1
#   }
#   BestAskLevel {
#     Qty: 730
#     Price: 1419
#     Orders: 3
#   }
#   BestAskLevel {
#     Qty: 193
#     Price: 1419.1000000000001
#     Orders: 3
#   }
#   BestAskLevel {
#     Qty: 415
#     Price: 1419.2
#     Orders: 10
#   }
# }
# ...
#
# New tick data received: MessageCode: 1502
# ID: 1010010000011536
# MarketDepthMessage {
#   InstrumentID: 1010010000011536
#   ExchangeSegment: NSECM
#   ExchangeInstrumentID: 11536
#   InstrumentName: "TCS"
 
# ...
#
# New tick data received: MessageCode: 1505
# IndexDataListMessage {
#   IndexDataList {
#     InstrumentID: 1010010002000001
#     ExchangeSegment: NSECM
#     ExchangeInstrumentID: 2000001
#     IndexName: "NIFTY 50"
#     TimeStamp: 1748339706
#     Last: 24787.55
#     Open: 24956.65
#     High: 24976.100000000002
#     Low: 24765.75
#     Close: 25001.15
#     YearlyHigh: 26277.350000000002
#     YearlyLow: 21281.45
#   }
# }
# ...
 
```
 
---
 
#### **```unsubscribe_market_data```**
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
client.unsubscribe_market_data(["NSE|RELIANCE"])
 
# Response:
# Unsubscribed from: NSE|RELIANCE
```
 
---
 
#### **```on_connect```**
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
 
#### **```on_close```**
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
 
#### **```connect_ws```**
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
 
#### **```stop_websocket```**
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
 
## **Web Socket Connection**
 
### **MarketDataWebSocketClient**
 
### Methods
 
#### **```__init_```**
```
def __init__(self, app_key: str, user_id: str)
```
??? info "Source code"
    ```python
    def __init__(self, access_token: str):
        self.access_token = access_token
        self.web_base_url = f"{Web_Base_URL}{self.access_token}"
 
        self.ws = None
        self.thread = None
 
        self.reconnect = True
        self.ping_interval = 30
        self.last_ping_time = time.time()
        self._heartbeat_thred = None
 
        self.on_massage_callback = None
        self.on_connect_callback = None
        self.on_close_callback = None
    ```
Initializes the WebSocket client with authentication and default configurations.
 
---
 
#### **```set_on_message```**
```
def set_on_message(self, callback)
```
 
Registers a custom function to handle incoming WebSocket messages.<br>
 
- Parameter: ```callback``` — a function that processes each received message.
 
---
 
#### **```set_on_connect(callback)```**
```
def set_on_connect(self, callback)
```
Registers a custom function to execute when the WebSocket connection is successfully established.<br>
 
- Parameter: ```callback``` — a function called once the connection opens.
 
---
 
#### **```set_on_close```**
```
def set_on_close(callback)
```
Registers a custom function to execute when the WebSocket connection closes.<br>
 
- Parameter: ```callback``` — a function called when the connection is terminated.
 
---
 
#### **```subscribe```**
```
def subscribe(instrument_ids)
```
Subscribes to live market data updates for the specified instruments.<br>
 
Parameters : ```instrument_ids``` (list | str): One or more instrument IDs to subscribe to.
 
---
 
#### **```unsubscribe```**
```
def unsubscribe(instrument_ids)
```
Unubscribes to live market data updates for the specified instruments.<br>
 
Parameters : ```instrument_ids``` (list | str): One or more instrument IDs to unsubscribe to.
 
---
 
#### **```on_message```**
```
def on_message(self, ws, message)
```
Handles incoming WebSocket messages from the market data stream.
 
**Parameters:**
 
| Name | Type | Description |
|------|------|--------------|
| `ws` | `WebSocket` | The active WebSocket connection instance. |
| `message` | `str` | The base64-encoded market data message received from the server. |
 
**Description:**  
Decodes the incoming message from Base64, parses it into a `MarketDataMessageBase` Protobuf object, and invokes the user-defined `on_message_callback` (if set).  
If message parsing fails, an error is logged for debugging purposes.
 
??? info "Source Code"
    ```python
    def on_message(self, ws, message):
        try:
            logging.debug(f"Raw message received: {message}")  
            decoded = base64.b64decode(message)  
            md_message = marketdata_pb2.MarketDataMessageBase()
            md_message.ParseFromString(decoded)
 
            if self.on_message_callback:
                self.on_message_callback(md_message)
 
 
        except Exception as e:
            logging.error(f"Failed to parse WebSocket message: {e}")
    ```
 
---
 
#### `on_close`
```
def on_close(ws, close_status_code, close_msg)
```
 
Handles WebSocket connection closure events and manages reconnection logic.
 
**Parameters:**
 
| Name | Type | Description |
|------|------|--------------|
| `ws` | `WebSocket` | The active WebSocket connection instance. |
| `close_status_code` | `int` | The status code indicating why the WebSocket was closed. |
| `close_msg` | `str` | The close message returned by the server, if any. |
 
**Description:**  
Logs the WebSocket disconnection details and triggers the user-defined `on_close_callback` (if set).  
If auto-reconnect is enabled (`self.reconnect` is `True`), waits for 5 seconds before attempting to reconnect automatically.
 
??? info "Source Code"
    ```python
    def on_close(self, ws, close_status_code, close_msg):
        logging.warning(f"WebSocket closed: {close_status_code}, {close_msg}")
        
        if self.on_close_callback:
            self.on_close_callback(close_status_code, close_msg)
 
        if self.reconnect:
            logging.info("Attempting to reconnect in 5 seconds...")
            time.sleep(5)
            self.start()
    ```
 
---
 
#### `start`
```
def start(self)
```
 
Initializes and starts the WebSocket connection in a background thread.
 
??? info "Source Code"
    ```python
        def start(self):
        self.ws = websocket.WebSocketApp(
            self.web_base_url,
            on_message=self.on_message,
            on_error=self.on_error,
            on_close=self.on_close
        )
        self.thread = threading.Thread(target=self.ws.run_forever)
        self.thread.daemon = True
        self.thread.start()
    ```
 
---
 
#### `stop`
 
` def stop(self)`
 
 
Stops the active WebSocket connection and disables automatic reconnection.
 
??? info "Source Code"
    ```python
        def stop(self):
        if self.ws:
            self.reconnect = False
            self.ws.close()
    ```