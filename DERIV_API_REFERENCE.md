# Deriv API Reference Guide

## Overview
Deriv APIs are built on WebSocket technology, enabling real-time trading systems and account management. The platform uses OAuth 2.0 for secure user authentication.

---

## 1. App ID Configuration

### Current Implementation
- **All Environments**: 113831 (Staging)

### App ID Usage
App IDs are used to identify your application when connecting to Deriv's WebSocket API. Each registered application has a unique app_id found in the Dashboard → Applications → Application manager.

---

## 2. Authentication Flow

### OAuth 2.0 Process

#### Step 1: Initialize OAuth Login
Direct users to:
```
https://oauth.deriv.com/oauth2/authorize?app_id=YOUR_APP_ID
```

Optional parameters for partners:
```
https://oauth.deriv.com/oauth2/authorize?app_id=YOUR_APP_ID&affiliate_token=YOUR_TOKEN&utm_campaign=YOUR_CAMPAIGN
```

#### Step 2: Handle Redirect
After login, users are redirected to your configured Redirect URL with session tokens:
```
https://[YOUR_WEBSITE_URL]/redirect/?acct1=cr799393&token1=a1-f7pnteezo4jzhpxclctizt27hyeot&cur1=usd&acct2=vrtc1859315&token2=a1clwe3vfuuus5kraceykdsoqm4snfq&cur2=usd
```

#### Step 3: Parse Query Parameters
Convert to account array:
```javascript
const user_accounts = [
  {
    currency: 'usd',
    token: 'a1-Yxh5gJS8m406Jopon5JlvKNRsxLMC',
    account: 'CRW1157'
  },
  {
    account: 'VRW1160',
    token: 'a1-yUqdjiIN0t6ICRc4eIMHDr1i6uKSV',
    currency: 'usd',
  }
];
```

#### Step 4: Authorize Using WebSocket
Call the `authorize` API with the selected token:
```json
{
  "authorize": "a1-Yxh5gJS8m406Jopon5JlvKNRsxLMC"
}
```

#### Authorization Response
```json
{
  "account_list": [
    {
      "account_category": "wallet",
      "account_type": "doughflow",
      "broker": "CRW",
      "created_at": 1753955451,
      "currency": "USD",
      "is_virtual": 0,
      "loginid": "CRW1157"
    }
  ],
  "balance": 0,
  "country": "aq",
  "currency": "USD",
  "email": "user@deriv.com",
  "fullname": "User Name",
  "is_virtual": 0,
  "loginid": "CRW1157",
  "scopes": ["admin", "payments", "read", "trade", "trading_information"],
  "user_id": 464
}
```

---

## 3. WebSocket Connection

### Base Endpoint
```
wss://ws.derivws.com/websockets/v3?app_id={app_id}
```

### Connection Example (JavaScript)
```javascript
const app_id = '113831'; // Staging API
const socket = new WebSocket(`wss://ws.derivws.com/websockets/v3?app_id=${app_id}`);

socket.onopen = function (e) {
  console.log('[open] Connection established');
  const sendMessage = JSON.stringify({ ping: 1 });
  socket.send(sendMessage);
};

socket.onmessage = function (event) {
  console.log(`[message] Data received: ${event.data}`);
};

socket.onclose = function (event) {
  if (event.wasClean) {
    console.log(`[close] Code=${event.code} reason=${event.reason}`);
  } else {
    console.log('[close] Connection died');
  }
};

socket.onerror = function (error) {
  console.log(`[error] ${error.message}`);
};
```

### Session Validity
- **Timeout**: 2 minutes of inactivity
- **Keep-Alive**: Send ping/time requests periodically to maintain connection

### WebSocket Events
1. **onopen**: Connection established
2. **onmessage**: Data received from server
3. **onclose**: Connection closed
4. **onerror**: Error occurred

### WebSocket Methods
1. **send()**: Transmit data to server
2. **close()**: Terminate connection (cannot be reused)

---

## 4. API Call Functions

### Three Main Functions

#### 1. Send
- Server responds with requested data **once**
- Used for one-time queries
- All APIs support this function

#### 2. Subscribe
- Initiates continuous data stream
- Perfect for real-time updates
- Requires `subscribe: 1` in request

#### 3. Forget
- Stops data stream from subscribe
- Requires the subscription ID from subscribe response
- Cleans up server-side resources

### Example: Subscribe to Ticks
```javascript
{
  "ticks": "R_100",
  "subscribe": 1
}
```

### Example: Forget Subscription
```javascript
{
  "forget": "subscription_id"
}
```

---

## 5. API Categories

### Available API Groups

1. **MT5** - MetaTrader 5 integration
2. **P2P** - Peer-to-peer trading
3. **Application** - App management and settings
4. **Account** - User account operations
5. **Market Data** - Real-time market information
6. **Cashier** - Deposits and withdrawals
7. **Reports** - Trading history and statements
8. **Trading** - Buy/sell contracts, proposals
9. **Utilities** - Helper functions like time, status

---

## 6. Rate Limits

- Check current limits via `website_status` API call
- Limits specified in `api_call_limits` field
- Can change over time

---

## 7. Connection Security

### Protocol
- **wss://** - Secure WebSocket (encrypted, protected) ✅ **RECOMMENDED**
- **ws://** - Unsecured WebSocket (not encrypted)

### Authentication
- Session tokens obtained via OAuth
- Tokens must be passed to `authorize` API
- Multiple accounts supported per token

---

## 8. Error Handling

### Connection Errors
- Handle with `socket.onerror` event
- Implement reconnection logic
- Log error details for debugging

### API Errors
- Server returns error objects in responses
- Include error code and message
- Validate before processing responses

---

## 9. Best Practices

1. **Always use wss://** for secure connections
2. **Implement keep-alive logic** - Send ping every 60-90 seconds
3. **Handle disconnections gracefully** - Implement automatic reconnect
4. **Parse OAuth tokens carefully** - Extract all account information
5. **Use subscribe for real-time data** - More efficient than repeated sends
6. **Clean up subscriptions** - Use forget to stop unused streams
7. **Implement error handling** - Anticipate network failures
8. **Rate limiting** - Monitor and respect API limits

---

## 10. Current Implementation Notes

### Project Structure
- API configuration: `/src/external/bot-skeleton/services/api/`
- App ID management: `appId.js`
- Base API configuration: `api-base.ts`
- Global config: `/src/components/shared/utils/config/config.ts`

### WebSocket Integration
The project uses `@deriv/deriv-api` package for WebSocket communication. All bot instances connect to `wss://ws.derivws.com/websockets/v3` with their respective app IDs.

### Token Management
- OAuth tokens extracted from redirect URL query parameters
- Tokens stored and managed by trading store
- Authorize call made after OAuth redirect
- User accounts list maintained for multi-account support

---

## 11. Useful API Calls

### Authentication
- `authorize` - Authenticate user with token
- `website_status` - Check API status and limits
- `logout` - End user session

### Market Data
- `ticks` - Subscribe to real-time price ticks
- `candles` - OHLC candlestick data
- `active_symbols` - Available trading instruments

### Trading
- `proposal` - Get contract proposal
- `buy` - Purchase a contract
- `sell` - Sell/close a contract

### Account
- `balance` - Get account balance
- `statement` - Trading history
- `portfolio` - Open positions

---

## 12. Testing the Connection

### Simple Ping Test
```javascript
socket.onopen = () => {
  socket.send(JSON.stringify({ ping: 1 }));
};

socket.onmessage = (event) => {
  console.log('Pong received:', event.data);
};
```

### Get Server Time
```javascript
{
  "time": 1
}
```

### Check API Status
```javascript
{
  "website_status": 1
}
```

---

## References

- **Getting Started**: https://legacy-docs.deriv.com/docs/getting-started
- **Authentication**: https://legacy-docs.deriv.com/docs/authentication
- **Understanding APIs**: https://legacy-docs.deriv.com/docs/understanding-apis
- **WebSockets**: https://legacy-docs.deriv.com/docs/websockets
- **OAuth**: https://legacy-docs.deriv.com/docs/oauth
- **API Explorer**: https://legacy-api.deriv.com/api-explorer
- **Application Manager**: https://legacy-api.deriv.com/dashboard/

