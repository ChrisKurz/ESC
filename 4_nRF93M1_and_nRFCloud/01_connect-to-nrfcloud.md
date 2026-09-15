# nRF93M1-DK Connect to nRF Cloud

Connect the nRF93M1-DK to your computer using the USB1. USB1 is connected to the nRF5340 with the Segger J-Link-OB application. It expose two virtual com ports. 

| Virtual com | Device   | Description                                |
|-------------|----------|--------------------------------------------|
|VCOM0        | nRF54L15 | UART0 - Zephyr shell (modem bypass sample) |
|VCOM1        | nRF93M1  | UART1 - AT Command host interface          |

We want to communicate directly to the AT Command host interface. On the command line use the command `nrfutil device list` to detect which virtual ports are in use.


```
>nrfutil device list
1052475354
Product         J-Link
Ports           COM110, vcom: 0
                COM111, vcom: 1
Traits          broken, jlink, seggerUsb, serialPorts, usb

Supported devices found: 1
```

Connect the `Serial Terminal` from **nRF Connect For Desktop** application, or use any other serial terminal. Connect to the `VCOM1` port at 115200 baud. 

Us `AT` to onfirm the nRF93M1 is responding correctly. 

```
> AT

OK
```



Report modem firmware: `AT+CGMR`

```
>AT
OK
>AT+CGMR
mfw_nrf93m1_1.4.1

OK
```

Report hardware version: `AT+CGMM`

```
> at+cgmm

nRF93M1-LABA-A0A (R3)

OK
```

List the variants supported bands: `AT%BAND`
```
>AT%BAND=?
%BAND: (1,3,5,7,8,20,28,38,40,41)

OK
```

List all commands available in this modem version: `AT+CLAC`

```
>AT+CLAC
<list of all supported AT commands>
OK
```

## Setup a connection

### Setup Unsolicited Response
An unsolicited notification (formally an Unsolicited Result Code, or URC) breaks the normal AT command pattern. Normally the exchange is strictly request-response: the host sends a command, reads back the result, and the transaction is finished.

A URC is a line that arrives on the same link without any command having been sent. It is pushed asynchronously whenever some event occurs on the other side — a state change, an incoming message, a completed background operation — so its timing is not under the host's control.

We setup some unsolicited responses, use those commands before starting the connection. This will help you to view state of the connection. 

#### EPS network registration status `+CEREG`:
Subscribes to unsolicited EPS network registration status notifications. 

Use `AT+CEREG=5` where 5 - URC subscription level, 5 is maximum parameters
```
>AT+CEREG=5

OK
```

This command enable `+CEREG:` unsolicited notifications. Typically you will see first `+CEREG: 2` what means he found a network and is trying to attach. If attaching fails you could see another `+CEREG: 2` with a different cell. 
When succesfull connected you see either `+CEREG: 1` registered to home network or `+CEREG: 5` registered to network and roaming.

```
+CEREG: 2

+CEREG: 5,"003B","005BC435",7
```

#### Packet domain event reporting `+CGEREP`: 
Enable unsolicited notifications to indicate EPS Packet Data Network (PDN) connection and bearer resources operations status. Notifications are sent when a packet data context is activated, deactivated, or modified. This enable unsolicted message (URC) from `+CGEV`.

Use `AT+CGEREP=1,0` where 1 - Discard URCs when the MT-TE link is reserved, for example, in on-line data mode.

```
> AT+CGEREP=1,0
AT+CGEREP=1,0
OK
```

#### Signaling connection status notification `+CSCON`:
Enables unsolicited notifications (+CSCON: `<state>`) that report changes in the RRC (Radio Resource Control) signaling connection state. It tells the modem to notify the host whenever it transitions between idle and connected mode. In idle mode the modem can be in the following states : DRX, eDRX or PSM.

```
> AT+CSCON=1

OK
```

#### Mobile termination error notification `+CMEE`:
Enable the use of the final result code +CME ERROR [err_nr]. The error codes are listed in command description. For reference of the full table see 3GPP 27.007 Ch. 9.1.

Use `AT+CMEE=2` where 2 - Enable verbose error message instead of error numbers.

```
>AT+CMEE=2

OK
```

#### Functional mode `+CFUN`:
The set command sets the functional mode to Minimum functionality, Normal, or Offline (Flight) mode. Setting the module to Minimum functionality or Offline mode might take some time if signaling with the network is needed. 

Use `AT+CFUN=1` where 1 - Sets the module to full functionality

```
>AT+CFUN=1

OK
```

No wait until you see `+CEREG: 5` this indicates that you are succesfully attaced to the cellular network. 

```
>AT+CFUN=1

OK

+CEREG: 2

+CEREG: 5,"003B","005BC435",7
```

Let's find out to what network we are connected with `AT+COPS?`:
```
> AT+COPS?

+COPS: 0,2,"20416",7

OK
```

#### Mobile network (PLMN) selection `+COPS`:
The third parameter "20416" is the numeric valua of the country code (MCC) and operator (MNC).
In this case MCC=204 which the Netherlands and MNC=16 which is "T-Mobile/Odido". 

There are many sources online to resolve the MCC/MNC to the actual value. Check to which network you are attached using [mcc-mnc.org](https://mcc-mnc.org/)


#### PDP Context `+CGDCONT`:
A PDP context (Packet Data Protocol context) is the data session configuration that lets a device exchange IP packets over a cellular network.

It is essentially a set of parameters — APN, IP type (IPv4/IPv6/both), and optionally authentication — that you define first, then activate. On activation, the network assigns an IP address and sets up the tunnel between the device and the gateway. Until a context is active, you have signal and network registration but no IP connectivity.

There is normally one default context that comes up automatically on attach, plus any additional ones you create and activate yourself.

Let's check to which APN we are connected, our IP address and support IP type (IPv4/IPv6) with `AT+CGDCONT?`. 

```
> AT+CGDCONT?

+CGDCONT: 1,"IPV4V6","pepper","240.2.221.92",0,0

OK
```

# Provision the nRF93M1 in nRF Cloud

Register for an [nRF Cloud](https://nrfcloud.nordicsemi.com/) account or log in to your existing account. If you already had an account, it is possible that it opens the nRF Cloud legacy dashboard; select `NEW EXPERIENCE` on the left navigation sidebar.

To provision the device, we need to generate a JSON Web Token (JWT) on the nRF93M1 using our `team-id` from nRF Cloud.

Navigate to `Fleet`, select `Devices`, and click `Add devices`. This is the manual way to add a single device. Use `Bulk edit devices` to provision a batch of devices.

![Add devices](images/nRFCloud_prov_01.png)

Choose the nRF93M1 from the device chooser:

![Select nRF93M1](images/nRFCloud_prov_02.png)


First we need to check that the nRF93M1 has a valid time. If you have been connected to a network after power on, the time should be set (some networks do not provide network time). Let's first check whether the time has been set:

```
>AT+CCLK?
AT+CCLK?
+CCLK: "00/01/01,00:00:21+00"

OK
```

This indicates that the network time is not set. You could connect to a network, but probably, during provisioning, it is better to set the time manually with `AT+CCLK="yyyy/MM/dd,hh:mm:ss+zz"`.
Make sure the network time is valid before continuing to the next step.

```
>AT+CCLK="2026/09/04,09:56:00+00"
AT+CCLK="2026/09/04,09:56:00+00"
OK
```

We now generate the JWT token on the nRF93M1 with the command `AT%REGJWT=<team-id>`. In the first step, you can copy the needed command with the `team-id` already added to the command. This JWT token is only valid for one hour. That is also the reason why we need the correct time set in the nRF93M1.
We now generate the JWT token on the nRF93M1 with the command `AT%REGJWT=<team-id>`. In the first step, you can copy the needed command with the `team-id` already added to the command. This JWT token is only valid for one hour. That is also the reason why we need the correct time set in the nRF93M1.

```
AT%REGJWT=3634e806-25e1-43ba-8bfa-2845a9aa243b
%REGJWT:
eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxNGE2ZmVjYS1jNGYwLTQ0ZTEtYTZkZC1hZjBhYzkzNzU5MjMiLCJpZCI6IjM2MzRlODA2LTI1ZTEtNDNiYS04YmZhLTI4NDVhOWFhMjQzYiJ9.signature
```

We also need the device-specific UUID. Use the command below to read this.

```
AT%DEVICEUUID
%DEVICEUUID: 14a6feca-c4f0-44e1-a6dd-af0ac9375923
```

Run those commands in the terminal on the nRF93M1 and keep the result; it is needed for the next step.

![alt text](images/nRFCloud_prov_03.png)

After retrieveing the information press `Continue` and copy your JWT Token and continue to the next step.

![alt text](images/nRFCloud_prov_04.png)

Confirm that the device UUID is matching you device (`AT%DEVICEUUID`)

![alt text](images/nRFCloud_prov_05.png)

Your device is now registered in nRF Cloud. 

![alt text](images/nRFCloud_prov_06.png)

### Next Step update the modem firmware of the nRF93M1 (02-update-modem-firmware.md)
