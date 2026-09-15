# Get the nRF Edge AI Add-on Code

Source: [Setting up the SDK — nRF Connect SDK - Edge AI Add-on 2.1.0 documentation](https://nrfconnectdocs.nordicsemi.com/addons/addon-edge-ai/latest/setting_up/sdk_setup.html#id1)

The Edge AI Add-on is distributed as a Git repository and is managed through its own west manifest. The compatible nRF Connect SDK version is specified in the `west.yml` file.

To get the Edge AI Add-on code, you can either:

- Use the [nRF Connect for Visual Studio Code](https://docs.nordicsemi.com/r/bundle/nrf-connect-vscode) extension, which provides a convenient way to clone the add-on and the compatible nRF Connect SDK version.

---

## Method 1: nRF Connect for Visual Studio Code

> **Note:** Use this method when you wish to specifically evaluate Edge AI capabilities, but do not have the nRF Connect SDK setup yet.

Clone the Edge AI Add-on code, together with the compatible nRF Connect SDK:

1. Ensure you have installed [Visual Studio Code](https://code.visualstudio.com) and the [nRF Connect for Visual Studio Code](https://docs.nordicsemi.com/r/bundle/nrf-connect-vscode) extension.
2. Follow the [nRF Connect SDK installation guide](https://nrfconnectdocs.nordicsemi.com/ncs/3.4.0/nrf/installation/install_ncs.html) to install nRF Connect SDK prerequisites and toolchain v3.4.0.

   > **Note:** The compatible version of the nRF Connect SDK will be cloned with the Edge AI Add-on repository in the following steps. The version of nRF Connect SDK is fixed to the version of Edge AI Add-on and is hard-coded in the `west.yml` file of the Edge AI Add-on.

3. Open the nRF Connect extension in Visual Studio Code by clicking its icon in the Activity Bar.
4. In the extension's Welcome view, click **Create a new application**. The list of actions appears in the Visual Studio Code quick pick.
5. Click **Browse nRF Connect SDK Add-on Index**. The list of available nRF Connect SDK add-ons appears in the Visual Studio Code quick pick.
6. Select **Edge AI Add-on**.
7. Select the add-on version to install (__Edge AI Add-on v2.3.0__). Depending on the speed of your internet connection, the update might take some time.

---

## Method 2: Add-on as an Extra Zephyr Module
Before using this approach, ensure that a compatible version of the nRF Connect SDK is installed. To identify the compatible nRF Connect SDK version, check the `west.yml` file of the Edge AI Add-on. Since west does not manage the add-on in this setup, you are responsible for keeping the versions synchronized.

1. Clone the Add-on repository:

   ```
   git clone --branch v2.1.0 https://github.com/nrfconnect/sdk-edge-ai
   ```

2. Set the CMake or environment variable `EXTRA_ZEPHYR_MODULES` to the Add-on code path. Use an absolute path to ensure proper path resolution. Check the [Environment Variables](https://nrfconnectdocs.nordicsemi.com/ncs/3.3.0/zephyr/develop/env_vars.html) documentation for different ways of setting environment variables in Zephyr.

   > **Note:** If you wish to:
   > - Use Edge Impulse in Zephyr library deployment.
   > - Use Edge Impulse samples from Add-on.
   >
   > Repeat the steps above for [edge-impulse-sdk-zephyr](https://github.com/edgeimpulse/edge-impulse-sdk-zephyr). The `EXTRA_ZEPHYR_MODULES` variable should be set to `<path to Add-on>;<path to Edge Impulse SDK>`.

---
   [Next: _Building a Wake Word and Keyword Spotting_](02-wake-word-kws-training.md)
