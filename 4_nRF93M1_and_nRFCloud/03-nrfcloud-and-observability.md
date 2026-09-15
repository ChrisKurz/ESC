# nRF Cloud commands
The nRF93M1 runs the nRF Cloud client inside the module firmware, so cloud services are reached with AT commands rather than a cloud library on the host. The transport is CoAP over DTLS 1.2, which keeps payload overhead low and coexists with cellular power saving. Every cloud transaction emits a `%COAP:` URC carrying the transport-layer status for that request, separately from the command's own response.

In the first step, we onboarded our device to nRF Cloud. This is a mandatory procedure to follow before starting this chapter. 

Some commands:
|Command|Service|
|-----------------|----------------------------------------------------------|
|%NRFCLOUDLOCATION|	Cell and Wi-Fi based positioning resolved by the cloud   |
|%NRFCLOUDMESSAGE |	Publish a JSON message to the cloud                      |
|%NRFCLOUDSHADOW  | Read desired shadow values, write reported shadow values |
|%NRFCLOUDFOTA	  |Modem firmware update and host application OTA            |
|%NRFCLOUDOBSUPLOAD/<br>%NRFCLOUDOBSHEARTBEAT/<br>%NRFCLOUDOBSFORWARD | Memfault firmware observability |

# Wi-Fi Scan
AT%WIFISCAN uses the module's 2.4 GHz Wi-Fi beacon receiver to detect access points for location purposes only — it is a passive scan, not a Wi-Fi connection feature. 

```
> AT%WIFISCAN=12000,1,5,3,0
%WIFISCAN: (-,,-28,"00:BE:D5:20:EA:54",11)
%WIFISCAN: (-,,-38,"00:BE:D5:20:EA:27",3)
OK
```
Each result line is `%WIFISCAN: (<encryption>,<SSID>,<RSSI dBm>,<BSSID>,<channel>)`. Duplicate SSIDs are removed and results are sorted by signal strength, strongest first.

```
%WIFISCAN=[<time>][,<round>][,<maxbssidnum>][,<scantimeout>][,<priority>]
[,<channelRecLen>][,<channelCount>][,<channelId1>][,<channelId2>]...[,<channelId14>]
[,<specific MAC address>]
```

Start the Wi-Fi scan with `AT%WIFISCAN=12000,1,5,3,0`

Main parameters: 
| Value| Description                                           |
|------|-------------------------------------------------------|
|12000 | Max response time in ms (5000–255000, default 12000)  |
|1     | Scan rounds (1–3, default 1)                          |
|5     | Required SSID count (4–40, default 5)                 |
|3     | Search time per round in seconds (1–255, default 5)   | 
|0     | Priority -  0 = Data preferred / 1 = Wi-Fi preferred     |


```
> AT%WIFISCAN=12000,1,5,3,0

%WIFISCAN:(-,"",-83,"0A:CF:0E:18:92:33",1)
%WIFISCAN:(-,"",-83,"2E:CF:0E:21:E1:1C",1)
%WIFISCAN:(-,"petronel 9 ",-83,"5C:A6:E6:86:E7:72",2)
%WIFISCAN:(-,"",-84,"62:A6:E6:86:E7:72",2)
%WIFISCAN:(-,"",-86,"92:5C:54:CA:DF:90",6)

OK

```

This data can be forwarded to your back office or used to obtain the latitude and longitude of your device. 


# nRF Cloud Location services
The `%NRFCLOUDLOCATION` command triggers a cloud-based location flow. 

`AT%NRFCLOUDLOCATION=<method>,<fetch_result>` collects radio measurements locally and lets nRF Cloud resolve them into a position. There are 7 different collection methods.

|Value | Method |
|------|----------------------------------------------|
|1     | Single-cell                                  |
|2     | Multicell                                    |
|3     | Single-cell and multicell                    |
|4     | Wi-Fi only                                   |
|5     | Single-cell and Wi-Fi                        |
|6     | Multicell and Wi-Fi                          |
|7     | All                                          |

The `<fetch_result>` parameter specifies whether the modem reports a location result after sending the collected data to nRF Cloud.
- 0 = Report an acknowledgment without returning a location result. 
- 1 = Report the location result back to the device.

Regardless of the selected `<fetch_result>`, the location is stored on the device in nRF Cloud. You can visualize this in nRF Cloud or use the result directly in Google Maps.
[https://maps.google.com/?q=`<lat>`,`<lon>`](https://maps.google.com/?q=<lat>,<lon>).


### Method 1 - Single Cell: `AT%NRFCLOUDLOCATION=1,1`
Uses only the serving cell (EARFCN, PCI, cell ID, TAC, RSRP/RSRQ). Fastest and cheapest, works anywhere the module is registered, and works in RRC Connected mode. Accuracy is the size of the cell, so typically hundreds of meters to several kilometers.

```
> AT%NRFCLOUDLOCATION=1,1
%COAP: RESPONSE,2.05
%NRFCLOUDLOCATION: 61.494029,23.775517,100,1
OK
```

[https://maps.google.com/?q=61.494029,23.775517](https://maps.google.com/?q=61.494029,23.775517)

### Method 4 - Wi-Fi only: `AT%NRFCLOUDLOCATION=4,1`
Runs a 2.4 GHz beacon scan and sends the access point BSSIDs and RSSI values. It provides the best accuracy of the three methods, typically tens of meters, and works indoors where cellular positioning is weakest. However, it depends entirely on access point density — a rural site may return nothing usable — and it requires RRC Idle.

Before we can send this command we first need to do a Wi-Fi scan.

```
> AT%WIFISCAN=12000,1,5,3,0

%WIFISCAN:(-,"petronel 9 ",-81,"5C:A6:E6:86:E7:72",2)
%WIFISCAN:(-,"petronel 9 ",-82,"5C:A6:E6:86:EA:F6",2)
%WIFISCAN:(-,"",-82,"62:A6:E6:86:E7:72",2)
%WIFISCAN:(-,"",-83,"62:A6:E6:86:EA:F6",2)
%WIFISCAN:(-,"Ziggo3757183",-84,"0A:CF:0E:18:92:34",1)

OK
> AT%NRFCLOUDLOCATION=4,1

%COAP: RESPONSE,2.05
%COAP: DATA,HEX,bf01fb4049ea79a430cc5102fb40116c223fc5e8ad03fb405739999999999a046457494649ff
%NRFCLOUDLOCATION: 51.831837,4.355599,92,4

OK
```

### Method 5 - Single-cell + Wi-Fi: `AT%NRFCLOUDLOCATION=5,1`
Collects both and lets the cloud choose. This is the robust option for a deployed device: Wi-Fi accuracy where APs exist, automatic fallback to single-cell where they do not, instead of a failed request.

```
> AT%NRFCLOUDLOCATION=5,1

%COAP: RESPONSE,2.05
%COAP: DATA,HEX,bf01fb4049ea787c12a01602fb40116c17f2650cca03fb405899999999999a046457494649ff
%NRFCLOUDLOCATION: 51.831801,4.355560,98,4

OK
```

# Viewing the position in nRF Cloud

In nRF Cloud, we can now visualize our reported location. Go to the `Legacy App` in the left sidebar.

![alt text](images/nRFCloud_location_00.png)

Select `DEVICE MANAGEMENT` -> `DEVICES`

![alt text](images/nRFCloud_location_01.png)

Click on your device to open a dashboard with device information.

![alt text](images/nRFCloud_location_02.png)

Scroll to the `Location` window. Here you can see the live location data and view the historical data. The window shows the lookups from that device. The blue circle represents the single-cell location, and the orange circles represent locations using Wi-Fi access points.

![alt text](images/nRFCloud_location_03.png)

# nRF Cloud Messaging
`%NRFCLOUDMESSAGE` is a one-way push of telemetry: `AT%NRFCLOUDMESSAGE={"appId":"BUTTON","data":"1"}` sends a JSON object or array (max 511 characters) to nRF Cloud over CoAP and replies `%NRFCLOUDMESSAGE: SENT`. Use it for events and sensor readings — fire and forget, nothing is stored on the device side.

Send some data to nRF Cloud: `AT%NRFCLOUDMESSAGE={"appId":"BUTTON","data":"1"}`

```
> AT%NRFCLOUDMESSAGE={"appId":"BUTTON","data":"1"}

%COAP: RESPONSE,2.01
%NRFCLOUDMESSAGE: SENT

OK
```

You can view this data at the same location as the position data in nRF Cloud.

![alt text](images/nRFCloud_location_04.png)

You can also extract this data directly from the cloud using the REST API.
The endpoint is `ListMessages`: `GET https://api.nrfcloud.com/v1/messages`, authenticated with your team's API key as a simple bearer token (found in the nRF Cloud portal under your user account page — note it is per user and per team).

For the nRF93M1, `<YOUR_DEVICE_UUID>` is the value returned by `AT%DEVICEUUID`.

```bash
curl -s -X GET \
  "https://api.nrfcloud.com/v1/messages?deviceId=<YOUR_DEVICE_UUID>&pageLimit=50" \
  -H "Authorization: Bearer <YOUR_API_KEY>" \
  -H "Accept: application/json"
```

# nRF Cloud Observability
After successfully connecting to the network, we will create the current metrics snapshot (chunk) for later observation in nRF Cloud. Before doing this, we need to run the following commands. The results of these commands will be stored in the metrics snapshot. 

Commands that are stored in the heartbeat metrics:
- AT+CGMR &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// Sets modem FW version  
- AT+CEREG=5 &nbsp;&nbsp;// Enable full CEREG URCs (PSM, AcT, etc.)  
- AT+CFUN=0 &nbsp;&nbsp;&nbsp;// Detach (triggers disconnect + connection_loss_count)  
- AT+CFUN=1 &nbsp;&nbsp;&nbsp;// Reattach (triggers time_to_connect_ms, on_time_ms)  
- AT+COPS? &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;// Get operator name  
- AT%BCINFO &nbsp;&nbsp;&nbsp;&nbsp;// Gets cell information (RSRP + band)

In the previous steps, we already activated the URC for `AT+CEREG=5` and connected with `AT+CFUN=1`. 
Run the following commands to update the metrics before sending them to nRF Cloud:
- AT+CGMR
- AT+COPS?
- AT%BCINFO

```
> AT+CGMR

mfw_nrf93m1_1.5.0

OK
> AT+COPS?

+COPS: 0,2,"20416",7

OK
> AT%BCINFO

%BCINFOSC: 3300,52,-97,-9,"204","16","005BC435","003B"
%BCINFONC: 6200,52,-80,-8
%BCINFONC: 3700,52,-81,-9
%BCINFONC: 1656,52,-90,-8
%BCINFONC: 1800,52,-93,-12
%BCINFONC: 3700,163,-96,-24

OK
```

We will now use the command `AT%NRFCLOUDOBSHEARTBEAT`. This command immediately triggers a cloud observability metrics heartbeat event and serializes all currently accumulated metric values into the chunk queue. This is not exported to the cloud; it is stored in the internal memory of the nRF93M1. This data can be uploaded to nRF Cloud using the cellular connection with an AT command or loaded into nRF Cloud using any other transport method. 

```
> AT%NRFCLOUDOBSHEARTBEAT
AT%NRFCLOUDOBSHEARTBEAT
OK
```
We will now upload the queued metrics to nRF Cloud using the command `AT%NRFCLOUDOBSUPLOAD`. Optionally, you can specify a `Project Key`. This is only needed when uploading to another project.

```
> AT%NRFCLOUDOBSUPLOAD

%COAP: RESPONSE,2.01
%COAP: DATA,8,Accepted
%COAP: RESPONSE,2.01
%COAP: DATA,8,Accepted
%COAP: RESPONSE,2.01
%COAP: DATA,8,Accepted
%COAP: RESPONSE,2.01
%COAP: DATA,8,Accepted
%COAP: RESPONSE,2.01
%COAP: DATA,8,Accepted
%COAP: RESPONSE,2.01
%COAP: DATA,8,Accepted
%COAP: RESPONSE,2.01
%COAP: DATA,8,Accepted
%NRFCLOUDOBSUPLOAD: UPLOADED,7

OK
```
