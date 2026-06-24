# Candle Aggregation API Documentation

## Base URL

http://localhost:8080/api/v1

---

## Load CSV Data

### Endpoint

GET /load

### Description

Loads stock market data from CSV into Cassandra.

### Request

GET http://localhost:8080/api/v1/load

### Response

```text
Data Loaded
```

---

## Get Aggregated Candles

### Endpoint

GET /candles

### Description

Returns aggregated OHLCV candle data for a given symbol and timeframe.

### Query Parameters

| Parameter | Type    | Required | Description                   |
| --------- | ------- | -------- | ----------------------------- |
| symbol    | String  | Yes      | Stock symbol (e.g. TCS)       |
| timeFrame | String  | Yes      | Aggregation timeframe         |
| startDate | String  | Yes      | Start date-time               |
| endDate   | String  | Yes      | End date-time                 |
| page      | Integer | No       | Page number (default 0)       |
| size      | Integer | No       | Records per page (default 10) |

### Supported Timeframes

* 1m
* 5m
* 15m
* 30m
* 60m
* 1d

### Sample Request

```http
GET /api/v1/candles?symbol=TCS&timeFrame=5m&startDate=2026-01-01T12:00:00&endDate=2026-01-01T12:59:00&page=0&size=5
```

### Sample Success Response

```json
{
  "symbol": "TCS",
  "timeframe": "5m",
  "candles": [
    {
      "datetime": "2026-01-01T12:00:00Z",
      "open": 3225.9,
      "high": 3226.7,
      "low": 3224.6,
      "close": 3225.9,
      "volume": 11282
    }
  ],
  "count": 12
}
```

---

## Error Responses

### Invalid Timeframe

```json
{
  "status": 400,
  "error": "Invalid timeframe: 10m"
}
```

### Symbol Not Found

```json
{
  "status": 400,
  "error": "No data found for symbol: ABC"
}
```

### Invalid Date Range

```json
{
  "status": 400,
  "error": "Start date must be before end date"
}
```

### Missing Parameter

```json
{
  "status": 400,
  "error": "symbol is required"
}
```

---

## Caching

The API uses Redis caching.

Cache Key Format:

```text
symbol-timeFrame-startDate-endDate
```

Example:

```text
TCS-5m-2026-01-01T12:00:00Z-2026-01-01T12:59:00Z
```

Repeated requests are served from Redis without querying Cassandra.

---

## Aggregation Logic

For every aggregation window:

* Open = First open value
* High = Maximum high value
* Low = Minimum low value
* Close = Last close value
* Volume = Sum of volumes
