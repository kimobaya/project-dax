curl -X POST http://localhost:8000/publish \
  -H "Content-Type: application/json" \
  -d '{
    "id":"unique_123",
    "ts":1736300000000,
    "ticker":"BTC-USD",
    "company":"Bitcoin",
    "asset":"crypto",
    "exchange":"Coinbase",
    "currency":"USD",
    "eventType":"Network Outage",
    "headline":"BTC network fees ease as mempool clears",
    "impact":"Throughput +40% vs. last hour",
    "mechanism":"Lower fees restore arbitrage flows",
    "watch":"Halvings narrative attracts hashpower",
    "source":"Exchange Notice",
    "url":"https://example.com",
    "confidence":"high",
    "direction":"up",
    "rank":87.3,
    "latency":540,
    "price":64321.5,
    "ch1m":0.7,
    "ch5m":1.2,
    "spark":[100,101,99,102]
  }'

