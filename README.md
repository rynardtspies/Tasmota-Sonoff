# Flashing a Sonoff MINI R2 (DIY) with Tasmota using OTA

## Getting the device on your Wifi network in DIY Mode

1. Wire up a 220V live and neutral connection to the MINI and a standard UK plug.
2. Plug in into a mains socket and it will power on. The Blue LED will either  flash `. .` or `. . -`. If it's flashing `. . -`, skip to step 4.
3. Hold down the button for 5 seconds. The blue LED should flash `. . -`
4. Hold down the button for another 5 seconds. The blue LED will flash `. . . . . . . . . .` If it doesn't hold it down for a further 5 seconds
5. The MINI should now be broadcasting a Wifi network called `ITEAD-#######`. Connect to this network using the password `12345678`
6. Browse to `10.10.7.1`
7. Scroll down and click `Wifi Setting`. Then configure your Wifi network
8. The device will connect to your main Wifi, so the `ITEAD-#####` network will disconnect. It will display "Wifi configuration is completed" when done.
9. Find the device IP for your local Wifi network on your Wifi router. It will be a hostname looking something like `ESP_XXXXXX`.


## Unlock the device for OTA

For this step, I have provided a Postman collection that you can import. The collection contains 3 API calls, as well as an `Postman Environment` that holds the IP address of the Sonoff Mini. **If you would like to use the collection, ensure that you have updated the IP address in the variable `sonoff-ip` to match your Sonoff Mini IP address on your Wifi network.**

1. Open Postman API client and issue the following API call to confirm that the MINI's API is listening on the network:

Request Params:
``` Text
Method: POST
Url: http://<local-wifi-sonoff-mini-ip>:8081/zeroconf/info

```
Request Data:
```json
{
    "deviceid": "",
    "data": {}
}
```

The API call should return something link this:

```json
{
    "seq": 2,
    "error": 0,
    "data": {
        "switch": "off",
        "startup": "off",
        "pulse": "off",
        "pulseWidth": 500,
        "ssid": "<YOUR NETWORK>",
        "otaUnlock": false,
        "fwVersion": "3.6.0",
        "deviceid": "1001405504",
        "bssid": "7a:ac:b9:91:51:e",
        "signalStrength": -59
    }
}
```

The `otaUnlock` attribute needs to be set to `true`, so that we can flash the MINI with Tasmota. Set it to `true` using the next API call:

```Text
Url: http://<local-wifi-sonoff-mini-ip>:8081/zeroconf/ota_unlock
Method: POST
```

Request Data:

```json
{
    "deviceid": "",
    "data": {}
}
```

The API call should return something like:

```json
{
    "seq": 2,
    "error": 0
}
```

Now when we run the INFO API call again, we should see:

```json
{
    "seq": 3,
    "error": 0,
    "data": {
        "switch": "off",
        "startup": "off",
        "pulse": "off",
        "pulseWidth": 500,
        "ssid": "<YOUR NETWORK>",
        "otaUnlock": true,
        "fwVersion": "3.6.0",
        "deviceid": "1001405504",
        "bssid": "7a:ac:b9:91:51:e",
        "signalStrength": -59
    }
}
```

## Flash with Tasmota

The API call to flash the MINI with Tasmota contains the URL and checksum for the binary to be installed. You don't need to download anything manually. Just send the API call:

```text
Url: http:/<local-wifi-sonoff-mini-ip>:8081/zeroconf/ota_flash
Method: POST
```

Request Data:

```json
{
    "deviceid": "",
    "data": {
        "downloadUrl": "http://sonoff-ota.aelius.com/tasmota-latest-lite.bin",
        "sha256sum": "5c1aecd2a19a49ae1bec0c863f69b83ef40812145c8392eebe5fd2677a6250cc"
    }
}
```

The API call should return the following immediately:

```json
{
    "seq": 4,
    "error": 0
}
```

The MINI is now flashing itself with version `9.5.0` of Tasmota. We will upgrade it to the latest in a moment.

Once flashed, the MINI will broadcast a `tasmota_xxxxx` Wifi network. Connect to this network and browse to `192.168.4.1`

Configure it on your wifi network using the Tasmota web interface.

## Upgrading Tasmota to Version 11.0.0

Once configured and on your wifi network, it should redirect to it's new live wifi IP address on your network.

For Tasmota to work with MQTT in Home Assistant, I found that it needed to be version 11.0.

Click `Firmware Upgrade`

Change the OTA URL to: `http://ota.tasmota.com/tasmota/release/tasmota-lite.bin.gz`

**NOTE THE "-lite" portion of the URL. This is important as the MINI doesn't have enough memory for the full Tasmota binary.**

When the upgrade completes, the browser should refresh and Tasmota 11.0.0 should be shown at the bottom!

## Configure Tasmota to work correctly with the MINI (Toggle Switch)

To get the physical toggle switch to work on the MINI:

1. In Tasmota, click "Configure", then "Configure Other"
2. In the `Template` field, paste the following line:

```json
{"NAME":"Sonoff MINIR2","GPIO":[17,0,0,0,9,0,0,0,21,157,0,0,0],"FLAG":0,"BASE":1}
```
3. Check "Activate"
4. Click Save.

You're done!
