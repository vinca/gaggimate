# Enable CONFIG_ESP_WIFI_GCMP_SUPPORT in default Arduino-ESP32 build

## Description

The Arduino-ESP32 framework is currently compiled without `CONFIG_ESP_WIFI_GCMP_SUPPORT` enabled, which prevents ESP32 devices from connecting to WiFi networks that advertise GCMP-256 ciphers.

## Problem

When a router/access point is configured with WPA2/WPA3 mixed mode and has GCMP-256 (00-0f-ac:9) cipher enabled alongside other ciphers (CCMP, GCMP-128, CCMP-256), ESP32 devices using the Arduino framework fail to connect.

**Expected behavior:** The ESP32 should negotiate a connection using one of the supported ciphers (e.g., CCMP) and ignore the unsupported GCMP-256.

**Actual behavior:** The connection fails entirely. The device goes through the authentication phase but disconnects without associating.

**Workaround:** Disabling GCMP-256 on the router allows the ESP32 to connect successfully.

## Technical Details

- **Board:** ESP32-S3 (Seeed XIAO ESP32S3)
- **Framework:** Arduino (via PlatformIO, `espressif32@6.12.0`, `framework-arduinoespressif32@3.20017.241212`)
- **ESP-IDF version:** 4.4.x based

I verified that the current sdkconfig.h does **not** define `CONFIG_ESP_WIFI_GCMP_SUPPORT`:

Present: `CONFIG_ESP32_WIFI_ENABLE_WPA3_SAE=1`
Missing: `CONFIG_ESP_WIFI_GCMP_SUPPORT` (not defined)

According to ESP-IDF documentation, this option enables GCMP support (GCMP-128 and GCMP-256) and is described as:
> "Select this option to enable GCMP support. GCMP support is compulsory for WiFi Suite-B support."

## Code Attempted

I attempted to configure WPA3 with PMF at runtime, but this doesn't help since GCMP support requires compile-time SDK configuration:

```cpp
wifi_config_t wifi_config = {};
wifi_config.sta.threshold.authmode = WIFI_AUTH_WPA2_WPA3_PSK;
wifi_config.sta.pmf_cfg.capable = true;
wifi_config.sta.pmf_cfg.required = false;
esp_wifi_set_config(WIFI_IF_STA, &wifi_config);
esp_wifi_connect();
```

## Feature Request

Please enable `CONFIG_ESP_WIFI_GCMP_SUPPORT=y` in the default Arduino-ESP32 sdkconfig. This would allow ESP32 devices to:

1. Properly connect to networks advertising GCMP-256 ciphers
2. Support WiFi Suite-B environments
3. Be compatible with modern router configurations using WPA3 with high-security ciphers

## Related Information

- Similar issues reported in other ESP32 projects (e.g., [Tasmota discussion #20121](https://github.com/arendst/Tasmota/discussions/20121))
- ESP-IDF Kconfig reference for `CONFIG_ESP_WIFI_GCMP_SUPPORT`: [ESP-IDF docs](https://docs.espressif.com/projects/esp-idf/en/v5.1-beta1/esp32/api-reference/kconfig.html)

Thank you for considering this enhancement!

