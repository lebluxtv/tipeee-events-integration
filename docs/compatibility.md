# Compatibility Notes

## Socket.IO / Engine.IO
- Engine.IO version: **v3**
- Required client: **socket.io-client v2.x**
- Socket.IO v3+ clients are **not compatible**

## Transports
- Recommended: `websocket`
- Fallback: `polling`

## Known Constraints
- Subscription must occur after connection
- A short delay after `connect` improves reliability
- Silent disconnects may occur

## Recommendations
- Implement automatic reconnection
- Use exponential backoff
- Preserve raw payloads for debugging
