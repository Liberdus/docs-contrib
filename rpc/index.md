
# RPC
Liberdus rpc provided RPC-ed interface for interacting with liberdus network for clients. The primary role of rpc is to distribute traffic accorss all the consensor. Rpc also aid in mitigating stress act upon validator for querying chain data such as transaction receipt and account balances from validator nodes by data distribution protocol's collector. Although it will prefer data from collector server, rpc will fallback towards validator nodes. Rpc also provide websocket chat room subscription for new messages.

If what you're looking for is a pure proxy server that route traffic to the one of the validator, you can use `liberdus-proxy` instead.

# Content
- [RPC API Usage](./api.md)
- [Internal Documentation](https://liberdus.github.io/liberdus-rpc)

