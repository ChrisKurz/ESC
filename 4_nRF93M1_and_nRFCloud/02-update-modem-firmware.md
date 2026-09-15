# Update the modem firmware of the nRF93M1

Modem firmware for an nRF93M1 module is built, signed, and released by Nordic Semiconductor, so you never compile it yourself. Your task is to deliver it, either over the air or locally through the host serial link.

Modem firmware updates are separate from host application updates.

### Checking the current version:

Read the modem firmware version with a standard 3GPP command `AT+CGMR`:

```
>AT
OK
>AT+CGMR
mfw_nrf93m1_1.4.1

OK
```

The latest version is v1.5.0. However, most of the kits used today are still running modem version V1.4.1. Since this was a pre-release version, it is mandatory to upgrade the modem firmware to version 1.5.0 to complete the hands-on.

There are two ways to perform the upgrade. The easiest is to use a command that downloads the delta image from a server. We will do this using the nRF Cloud platform, where we have far more control over the upgrade process. This method can be used to upgrade both the modem firmware and the application firmware. Since we are using the serial terminal and issuing commands manually, we cannot use this method to upgrade the application core of the nRF54L15. That is possible from an application running on the nRF54L15.

### Update mechanism

Updates are delivered as differential (delta) images. This image only contains the differences between the current firmware and the new firmware. This helps to keep the data traffic low. The current image is about 215Kbyte.

Delta images are signed by Nordic. The module verifies the signature before applying an update. Secure boot prevents unsigned or unauthorized firmware from executing. 

### Updating over the air

The nRF Cloud client in the module firmware handles updates over the air. The device must be provisioned and connected as we did in the previous chapter.

Updates over the air use the command `AT%NRFCLOUDFOTA`, which serves both the module and the host application through different modes.

To check for and apply a modem firmware update:

```
AT%NRFCLOUDFOTA=3,"<projectKey>"
```

Unsolicited `%COAP` and `%FOTA` result codes can be emitted during the cloud transaction.

Plan for the following in your host application:

- The module is unavailable for normal traffic while it applies the update and restarts. Your host must tolerate this rather than treating it as a fault.
- Power must remain stable through the update. Do not start an update on a device that is low on battery.

### Prepare the delta firmware image in nRF Cloud
We need to do the following steps:
1. Retrieve the `project-key` from nRF Cloud.
2. Read out the hardware version of the nRF93M1.
3. Create a FOTA release in nRF Cloud.
    - Create a new FOTA release.
    - Set up the profile (from firmware, to firmware).
    - Upload the delta FOTA release (hardware version, define software group, image).
    - Activate the FOTA release.
4. Connect the nRF93M1 to the cellular network.
5. Check on the nRF93M1 whether a firmware update is available for this hardware.
6. Download and apply the firmware update.

### Step 1 - Retrieve the project-key

Navigate to `FLEET` in the left sidebar and select `Project Settings`.
In the `General` section, you will see your `Project Key`. Copy this key, you will need it later.

![Retrieve Project Key](images/nRFCloud_fota_01.png)

### Step 2 - Read out the hardware version

Connect to the nRF93M1 using the serial terminal and read out the hardware version using `AT+CGMM`.

```
> AT+CGMM
AT+CGMM
nRF93M1-LABA-A0A (20250428)

OK
```
Remember the hardware version (`nRF93M1-LABA-A0A`), we will need it later.


### Step 3 - Create FOTA Release in nRF Cloud

Since we are going to upgrade the modem firmware, we will create a `Delta Release`.
Navigate in the sidebar to `FLEET` > `OTA Releases`. Select the `Delta Releases` tab and click `Create Release`.

![Create Delta Release](images/nRFCloud_fota_02.png)

In the `Create Delta Release` window, we need to provide the base firmware from which we want to upgrade, followed by the target version of the firmware.

Use the following versions:
| Field        | Firmware version  |
|--------------|-------------------|
|`From Version`| mfw_nrf93m1_1.4.1 |
|`To Version`  | mfw_nrf93m1_1.5.0 |

It is important to use the above naming convention as the modem firmware is expecting this for checking of new modem firmware versions.

![Enter firmware versions](images/nRFCloud_fota_03.png)

The next step is to upload the delta image for a specific hardware version.
In the OTA Release overview, click `Add OTA Payload to Delta Release`.

![Add OTA Payload to Delta Release](images/nRFCloud_fota_04.png)

In the ***Hardware Version*** field, enter the value we read earlier from the serial port: `nRF93M1-LABA-A0A`. Then enter the ***Software Type*** group to identify the type of image this is. It is **MANDATORY** to use `mfw` for the ***Software Type*** for modem firmware images. This ***Software Type*** will be linked to this ***Hardware Version***. The first hardware version a device reports becomes its permanent hardware version.

When adding another device with the same ***Hardware Version***, it is automatically assigned to the first used ***Software Type***. It is possible to change the linked ***Software Type*** for the ***Hardware Version***. Do this in `Project Settings > Hardware` by pressing the edit button on the hardware version you want to change.

If we use something different than `mfw`, it will be accepted at this stage, but it will not be recognized on the device side as modem firmware.

Now drag or browse the delta image `mfw_nrf93m1_update_from_1.4.1_to_1.5.0.bin` to the `Upload` field. This file is located in the `modem_firmware` folder in this project.
Press `Add` to create the FOTA Release.

![Add OTA Payload](images/nRFCloud_fota_05a.png)

The final step is to activate the FOTA release so it can be used. Do this by pressing `Activate` within the newly created FOTA Release package.

![Activate FOTA Release](images/nRFCloud_fota_06.png)

In the next step, we need to define the `Cohort` to which this FOTA release is assigned. Since we have not created any `Cohort`, select the `Default` cohort.

![Activate Release](images/nRFCloud_fota_07.png)

The FOTA Release is now ready to be used.

### Step 4 - Connect the nRF93M1 to the cellular network

Before we can check whether a modem firmware update is available, we should first connect to the cellular network. It is enough to start the connection with `AT+CFUN=1`, but the best practice is to enable the following unsolicited responses:

|AT Command    | Description                                                         |
|--------------|---------------------------------------------------------------------|
|AT+CMEE=2     | Enables the use of the final result code +CME ERROR [err_nr]        |
|AT+CEREG=5    | subscribes unsolicited EPS network registration status notifications|
|AT+CGEREP=1,0 | Enables the sending of unsolicited +CGEV result codes               |
|AT+CSCON=1    | Enable signaling connection status notification                     |
|AT+CFUN=1     | Sets the module's functional mode to full functionality             |


```
> AT+CMEE=2
AT+CMEE=2
OK
> AT+CEREG=5
AT+CEREG=5
OK
> AT+CGEREP=1,0
AT+CGEREP=1,0
OK
> AT+CSCON=1
OK
> AT+CFUN=1
AT+CFUN=1
OK
```

### Step 5 - Check if Modem Firmware is available

With the `AT%NRFCLOUDFOTA` command we can check or apply a modem update. The same command can also be used to check and retrieve firmware updates for the host MCU.

Updates over the air use `AT%NRFCLOUDFOTA`, which serves both the module and the host application through five modes.
|`<mode>`| Action                                                            |
|--------|-------------------------------------------------------------------|
| 0      | Check for a host application OTA update and cache the download URL|
| 1      | Download one host OTA chunk from the cached URL                   |
| 2      | Check for a modem firmware update                                 |
| 3      | Check for and apply a modem firmware update                       |
| 4      | Clear the cached host OTA download URL                            |

For this update flow, we will only use `<mode>` 2 and 3.

To check whether a modem firmware update is available:
```
AT%NRFCLOUDFOTA=2,"<projectKey>"
```

When you have followed the steps above correctly, you should expect a response like this:

```
> AT%NRFCLOUDFOTA=2,OM624h5S1Pxxxxxxah441un8Ydxxxxxx
AT%NRFCLOUDFOTA=2,OM624h5S1Pxxxxxxah441un8Ydxxxxxx
OK

%FOTA: CHECKING

%COAP: RESPONSE,2.05

%COAP: DATA,153,{"data": {"url": "https://ota-cdn.memfault.com/11390/ota-payloads/35319xxxxxx?token=FFTII0UPyZyrgw1MkgwxMYWS7KFeP0wF0Xupsxxxxxx&expires=178881xxxxxx=2"}}

%FOTA: UPDATE_AVAILABLE
```

If you get a response ending with `%FOTA: NO_UPDATE`, check whether the modem firmware naming from->to is correct, whether the software group is `mfw`, and whether you have activated the FOTA release.

When we get the message `%FOTA: UPDATE_AVAILABLE`, we can start the download and update of the modem firmware. It is not necessary to check first; it is also possible to start mode 3 directly. It will report if no firmware update is available.

To check and apply a modem firmware update:
```
AT%NRFCLOUDFOTA=3,"<projectKey>"
```

The modem will start downloading the image and apply the update automatically. Make sure you do not disconnect or power down the modem during this process.

You should see something like this:
```
> AT%NRFCLOUDFOTA=3,OM624h5S1Pxxxxxxah441un8Ydxxxxxx
AT%NRFCLOUDFOTA=3,OM624h5S1Pxxxxxxah441un8Ydxxxxxx
OK

%FOTA: CHECKING

%COAP: RESPONSE,2.05

%COAP: DATA,153,{"data": {"url": "https://ota-cdn.memfault.com/11390/ota-payloads/35319xxxxxx?token=FFTII0UPyZyrgw1MkgwxMYWS7KFeP0wF0Xupsxxxxxx&expires=178881xxxxxx=2"}}

%FOTA: ZONE,1,462848

%FOTA: ERASING

%FOTA: DOWNLOADING

%FOTA: DOWNLOADING,16384

%FOTA: DOWNLOADING,32768

%FOTA: DOWNLOADING,49152

%FOTA: DOWNLOADING,65536

%FOTA: DOWNLOADING,81920

%FOTA: DOWNLOADING,98304

%FOTA: DOWNLOADING,114688

%FOTA: DOWNLOADING,131072

%FOTA: DOWNLOADING,147456

%FOTA: DOWNLOADING,163840

%FOTA: DOWNLOADING,180224

%FOTA: DOWNLOADING,196608

%FOTA: DOWNLOADING,212992

%FOTA: DOWNLOADING,215222

%FOTA: DOWNLOAD_DONE,215222

%FOTA: VERIFYING

%FOTA: VERIFIED

%FOTA: UPGRADING

%FOTA: "APPLY"
%FOTA: "UPDATING",0
%FOTA: "UPDATING",7
%FOTA: "UPDATING",14
%FOTA: "UPDATING",22
%FOTA: "UPDATING",29
%FOTA: "UPDATING",37
%FOTA: "UPDATING",44
%FOTA: "UPDATING",51
%FOTA: "UPDATING",59
%FOTA: "UPDATING",66
%FOTA: "UPDATING",74
%FOTA: "UPDATING",81
%FOTA: "UPDATING",85
%FOTA: "UPDATING",92
%FOTA: "UPDATING",100
%FOTA: "DONE",0

RDY
```
The modem firmware is completed, we can verify this with the command `AT+CGMR`

Validate upgrade of firmware:

```
> AT+CGMR

mfw_nrf93m1_1.5.0

OK
```

We now have succesfully upgraded the modem firmware.

# Modem Firmware update through HTTP or HTTPS `AT%HTTPFOTADL`

The `AT%HTTPFOTADL` command downloads a firmware package from an HTTP or HTTPS server and triggers a firmware update. This command can only be used to update the modem firmware and not the host MCU. 

To download and update a modem firmware:
``` 
AT%HTTPFOTADL="http://firmware.nrfcloud.com/93m1/mfw_nrf93m1_update_from_1.4.1_to_1.5.0.bin",100
``` 

This should result in the following:
``` 
> at%HTTPFOTADL="http://firmware.nrfcloud.com/93m1/mfw_nrf93m1_update_from_1.4.1_to_1.5.0.bin",100

at%HTTPFOTADL="http://firmware.nrfcloud.com/93m1/mfw_nrf93m1_update_from_1.4.1_to_1.5.0.bin",100

OK

%HTTPURC: "FOTA","HTTPSTART"

%HTTPURC: "FOTA","DOWNLOADING",0

%HTTPURC: "FOTA","DOWNLOADING",1

%HTTPURC: "FOTA","DOWNLOADING",2

-----

%HTTPURC: "FOTA","DOWNLOADING",98

%HTTPURC: "FOTA","DOWNLOADING",99

%HTTPURC: "FOTA","DOWNLOADING",100

%HTTPURC: "FOTA","DOWNLOADED"

%FOTA: "APPLY"

%FOTA: "UPDATING",0

%FOTA: "UPDATING",7

%FOTA: "UPDATING",14

-----

%FOTA: "UPDATING",92

%FOTA: "UPDATING",100

%FOTA: "DONE",0

RDY
``` 

In the parameters of the command, we specified a delta image from 1.4.1. If the current modem version is older or newer, you can expect the following error after the image is downloaded:

```
%HTTPURC: "FOTA","DOWNLOADED"

%HTTPURC: "FOTA","VERIFICATION FAILURE",5,4
```

The syntax of the response `%HTTPURC: "FOTA","VERIFICATION FAILURE",<cause>,<detail>` means:
5 = verification error, and 4 = wrong base image.

### Next Step is to enable observability and detect or loaction with WiFi location commands (03-observability.md)
