# http_request Component Fix for ESPHome 2025.11.0

This branch contains a backport of the infinite loop fix for the `http_request` component, compatible with ESPHome 2025.11.0.

## The Problem

The `http_request` component enters an infinite loop when HTTP servers don't send a `Content-Length` header or close connections prematurely, causing:
- "Stream pointer vanished!" errors (Arduino platforms)
- CPU spinning on zero-byte reads
- Watchdog timeouts and device reboots

Issue: https://github.com/esphome/issues/issues/6682

## The Fix

Adds error handling to break out of the read loop when:
- `read()` returns -1 (connection closed/error)
- `read()` returns 0 (no more data available)

## How to Use as External Component

### Option 1: Using the Fixed Component Only

Add this to your ESPHome YAML configuration:

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/pvizeli/esphome
      ref: fix-http-request-2025.11
    components: [ http_request ]
    refresh: 1d

# Your http_request configuration
http_request:
  useragent: esphome/device
  timeout: 10s
```

### Option 2: Using a Specific Commit (Recommended for Production)

For better stability, pin to a specific commit:

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/pvizeli/esphome
      ref: 59f09cca8  # Commit hash
    components: [ http_request ]

http_request:
  useragent: esphome/device
  timeout: 10s
```

## Verification

The fix has been tested and verified on:
- ✅ ESP32 (Arduino & ESP-IDF)
- ✅ ESP8266 (Arduino)
- ✅ RP2040 (Arduino)
- ✅ Host platform

## When to Remove

Once the upstream ESPHome project merges the fix and releases a new version, you can remove the `external_components` section and use the official release.

## Technical Details

**Changed File**: `esphome/components/http_request/http_request.h`
**Lines Modified**: 258-260
**Change**: Added `if (read <= 0) { break; }` check in the response reading loop

## License

Same as ESPHome (GPL-3.0)
