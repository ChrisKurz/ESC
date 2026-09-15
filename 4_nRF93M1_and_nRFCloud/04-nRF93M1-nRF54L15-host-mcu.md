# Serial Modem Host Application

Host-side firmware for Nordic Smart Modem modules. This repository is built on nRF Connect SDK (NCS) and follows the modular zbus + SMF architecture used by the Asset Tracker Template.
 

[Serial Modem Host Applications](https://github.com/nrfconnect/ncs-serial-modem-host-applications/)

nrfutil device list
VCOM0

If pip needs to be installed:
python.exe -m pip install


If pip needs a update
python.exe -m pip install --upgrade pip

pip3 install nrfcloud-utils
I needed to upgrade pip3 install --user --upgrade nrfcloud-utils


## Flash Serial-modem 
The serial modem is flashed to nRF54L15
Flash from the folder: `serial-modem\93m1_ppp-nrf93m1-v0.10.0` the file `merged.hex` using the programmer of the nRF Connect For Desktop.

![alt text](images/nRFCloud_host_mcu_00.png)

Reset the nRF93M1DK using the reset button.

## 2. Get the device ID
Open a serial terminal on the host console (uart20). Note the device ID from the boot log:

``<inf> nrf_cloud_info: Device ID: <16-hex-device-id>``
It comes from the ``nRF54L15`` SoC HW ID, not the modem UUID.

```
[00:00:00.028,133] <inf> mflt: GNU Build ID: d43c2ecac7eefecce83fd09cf917e4278e696922
```

## 3. Create a self-signed CA certificate
Once per CA, in the directory where you keep the CA files:

`create_ca_cert -c US -f self_`

```
>create_ca_cert -c US -f self_
INFO     Creating self-signed CA certificate...
INFO     File created: c:\temp\work\self_<serial>_ca.pem
INFO     File created: c:\temp\work\self_<serial>_prv.pem
INFO     File created: c:\temp\work\self_<serial>_pub.pem
```

## 4. Install the Credentials
From the same directory as the CA files:
Where `<serial>` is generated om previous step. 

```
device_credentials_installer \
  --ca self_<serial>_ca.pem --ca-key self_<serial>_prv.pem \
  --id-str <device_id> \
  -s -d --verify --coap --local-cert --cmd-type tls_cred_shell \
  --port <serial_port_VCOM0>
```
On windows make sure that com-port has captil letter "COMxx". On a apple device use `/dev/cu.usbmodem*`
On success it writes locally in the file `onboard.csv`.

The sec tag is 16842753. ??????

## 5. Onboard

Legacy app -> drop down in right top corner  -> User Account -> API Key copy this valua
![alt text](images/nRFCloud_host_mcu_01.png) [alt text](04-nRF93M1-nRF54L15-host-mcu.md)

`nrf_cloud_onboard --api-key <your_api_key> --csv onboard.csv`


The application includes Memfault for remote crash reporting and device health metrics. The SDK uploads data periodically in the background (CONFIG_MEMFAULT_PERIODIC_UPLOAD) over the nRF Cloud CoAP session.

-------
# TODO
- How often is the heartbeat send? Can it be changed? 
- documentend what is in the heartbeat, assume it is not the AT%HEARTBEAT
- shell command version does not match the versions in the filename.
- Shell FOTA i assumme it is the nRF54L15
- Wait for the cellular link, then for the connection and first upload.... how long?

UIDDdir: d43c2ecac7eefecce83fd09cf917e4278e696922


nrfutil device erase
nrfutil device recover
nrfutil device program --firmware firmware\merged.hex

hoeft niet opnieuw te worden aangemaakt na een erase.
create_ca_cert -c US -f self_

device_credentials_installer --ca self_0x177c574a94ec4e9b3707740bf51a1e18c1553393_ca.pem --ca-key self_0x177c574a94ec4e9b3707740bf51a1e18c1553393_prv.pem --id-str 316CD491F94AC082 -s -d --verify --coap --local-cert --cmd-type tls_cred_shell --port COM114

nrf_cloud_onboard --api-key 2db12ed3c9c914c7e2afbb4e0d3368082433d00b --csv onboard.csv
