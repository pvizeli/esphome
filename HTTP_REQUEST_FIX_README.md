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

This fix is applied to:
- ✅ **http_request actions with `capture_response: true`**
- ✅ **OTA/update component** (firmware downloads and MD5 verification)
- ✅ **http_request.ota component** (flash operations)

## How to Use as External Component

**IMPORTANT**: Always use the branch name, not commit hashes. ESPHome's external component system may not be able to fetch specific commits from forked repositories.

### Recommended Configuration

Add this to your ESPHome YAML configuration:

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/pvizeli/esphome
      ref: fix-http-request-2025.11  # Use branch name, not commit hash
    components: [ http_request ]
    refresh: 0s  # Force refresh to get latest code

# Your http_request configuration
http_request:
  useragent: esphome/device
  timeout: 10s

# The fix works for:
# 1. http_request actions with capture_response: true
# 2. update component (OTA firmware downloads)
# 3. http_request.ota component

# Example 1: Using with update component (most common use case)
update:
  - platform: http_request
    name: "Firmware Update"
    source: "http://your-server.com/firmware.bin"

# Example 2: Using with capture_response
script:
  - id: make_request
    then:
      - http_request.get:
          url: "http://example.com/api"
          capture_response: true
          on_response:
            then:
              - logger.log:
                  format: "Response: %s"
                  args: [ 'body.c_str()' ]
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

**Files Changed**:
1. `esphome/components/http_request/http_request.h` (lines 258-260)
   - Response capture loop
2. `esphome/components/http_request/ota/ota_http_request.cpp` (lines 136, 251-253)
   - OTA firmware download loop
   - MD5 verification download loop

**Change**: Added `if (read <= 0) { break; }` or changed `if (bufsize < 0)` to `if (bufsize <= 0)` in all read loops

## License

Same as ESPHome (GPL-3.0)
