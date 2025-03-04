---
title: Matter (BETA)
description: Instructions on how to integrate Matter with Home Assistant.
ha_category:
  - Binary sensor
  - Climate
  - Cover
  - Light
  - Lock
  - Sensor
  - Switch
ha_release: '2022.12'
ha_iot_class: Local Push
ha_config_flow: true
ha_codeowners:
  - '@home-assistant/matter'
ha_domain: matter
ha_platforms:
  - binary_sensor
  - climate
  - cover
  - diagnostics
  - event
  - light
  - lock
  - sensor
  - switch
ha_integration_type: integration
---

The Matter integration allows you to control Matter devices on your local Wi-Fi or {% term Thread %} network.

<div class='note warning'>
The integration is marked BETA: Both the Matter standard itself and its implementation within Home Assistant are in an early stage. You may run into compatibility issues and/or other bugs.
</div>

# Introduction - What is Matter?

Matter is a new smart home connectivity standard for home automation products and IoT (Internet of Things) devices, see its [Wikipedia article](https://en.wikipedia.org/wiki/Matter_(standard)).

The initial version 1.0 release of Matter was published in October of 2022. Matter is still in the process of being adopted in the smart home market. It has gotten much publicity because of its promise of interoperability across all ecosystems. The largest tech companies like Google, Apple and Amazon teamed up to develop this new smart home connectivity standard under the roof of the CSA ([Connectivity Standards Alliance](https://csa-iot.org/)). The largest tech companies that are already active in the home automation market have announced that they are or will be working on Matter-compatible products and also joined the development effort.

Matter products run locally and always allow local control, with device control done without the need for any internet connection or cloud services. From a technical perspective, you can use a Matter-compatible device with Home Assistant without connecting to a vendor-specific cloud. However, some vendors may require you to set up an account before you can enable Matter support for some products, (especially for commercial manufacturer's own branded gateways/bridges/hubs/controllers sold as appliances).

Unlike other common radio-based protocols for IoT, (like Zigbee, Z-Wave, and Bluetooth), the Matter standard specification itself does not contain its own proprietary radio protocol or network transport protocol, but instead, it is a service control protocol that runs **on top** of the existing network infrastructure at the application level, with all Matter devices communicating using standard IP-based (IPv6) communication over your existing [local area network (i.e. LAN networks like Wi-Fi and Ethernet)](https://en.wikipedia.org/wiki/Local_area_network) or [Thread (Low-Power Wireless Personal Area Network)](https://en.wikipedia.org/wiki/Thread_(network_protocol)) depending on the type of device.

Home Assistant is a so-called "_controller_" in a Matter ecosystem, meaning that it can control Matter-based devices. Other examples of Matter controllers are the Google Nest products, Apple HomePod speakers, Samsung SmartThings Station, and some newer Amazon Echo devices.

## Bridge devices

One of the great things about Matter is that you can have both Wi-Fi and Thread based devices on the same controller.
Next to actual devices (like actors or sensors), you will also see bridges. The bridge connects the network over Ethernet or Wi-Fi and bridges multiple devices into a Matter network. A great example is the Philips Hue V2 bridge, which is a Zigbee hub and a Matter bridge. This bridge exposes all Zigbee devices already connected to the bridge as Matter devices on the network. Also, Aqara, SwitchBot, and IKEA have launched such Hub devices.

<div class='note'>
Home Assistant, as a Matter controller, only supports **control** of Matter devices. Home Assistant is not a bridge itself and it cannot turn existing devices within Home Assistant into Matter compatible devices.
</div>

## Thread

Matter goes hand-in-hand with (but is not the same as) [Thread](/integrations/thread). Thread is a low power radio mesh networking technology. Much like Zigbee, but with the key difference that it is _IP-addressable_, making it the perfect companion transport protocol for Matter.

<div class='note'>
Many devices that (will) hit the market will use Thread for radio communication and Matter as a control protocol, but this is not guaranteed. For example, Thread-based devices are available that only support Apple HomeKit or some vendor-specific communication protocol. There are also a few cases where you need to apply for a (beta) firmware update on the device to enable Matter as a communication protocol. Therefore, do not assume Matter support when you see a Thread logo when looking for devices. Please be sure to look for the *Matter* logo itself (on either Wi-Fi/Ethernet-based devices or Thread) or any other confirmation by the manufacturer that the device supports Matter.
</div>

## Bluetooth used during commissioning

Most (if not all) Matter-compliant devices will also have a Bluetooth chip onboard, this is to ease commissioning (a somewhat technical term for adding a device to your controller). Bluetooth will not be used to control a device but only to pair it after unboxing or factory resetting. The Home Assistant controller uses the Home Assistant Companion app to do {% term commissioning %}, so you can bring your phone close to the device you want to {% term commission %}. The controller will then send your network credentials to your device over Bluetooth in the {% term commissioning %} process. If that succeeds, the device will communicate over its native interface, meaning Wi-Fi, Ethernet, or Thread.

<div class='note'>
Although your Home Assistant server might have a Bluetooth adapter on board that the controller can use to {% term commission %} devices, we choose not to utilize that adapter. Mainly to prevent issues with the built-in Bluetooth integration but also because it makes more sense to bring your mobile devices close to the Matter device you'd like to {% term commission %}.
</div>

## Multi fabric: join to multiple controllers

One of the great features of Matter is the so-called _Multi Fabric_ feature: you can join the same device to multiple controllers. For example, simultaneously add it to Google Home, Apple Home, and Home Assistant. The standard describes that each device should be able to at least support 5 different fabrics simultaneously.

For devices where Home Assistant provides a native integration (with local API), Matter may not be the best option. Matter, being a universal standard, might not have the nitty-gritty features that come with a product-specific protocol. A good example is Philips Hue: the communication over Matter only provides the basic controls over lights, while the official [Hue integration](/integrations/hue) brings all Hue unique features like (dynamic) scenes, entertainment mode, etc.

![image](/images/integrations/matter/matter_thread_infographic.webp)

Image taken from [this excellent article by The Verge](https://www.theverge.com/23165855/thread-smart-home-protocol-matter-apple-google-interview) about Matter that shows the landscape of Matter, Thread, Border routers and bridges in a nice visualized way.

{% include integrations/config_flow.md %}

For communicating with Matter devices, the Home Assistant integration runs its own "Matter controller" in a separate process which will be launched as an add-on. This add-on runs the controller software and connects your Matter network (called Fabric in technical terms) and Home Assistant. The Home Assistant Matter integration connects to this server via a WebSocket connection.

### Supported installation types

It is recommended to run the Matter add-on on Home Assistant OS. This is currently the best-supported option.

If you run Home Assistant in a container, you can run a Docker image of the [Matter server](https://github.com/home-assistant-libs/python-matter-server). The requirements and instructions for your host setup are described on that GitHub page.

Running Matter on a Home Assistant Core installation is not supported.

## Adding a Matter device to Home Assistant

Each Matter network is called a fabric. Each home automation controller that controls Matter devices has its own "fabric". You can add devices directly to the fabric of your Home Assistant instance, or share them from another fabric (ie Google, Apple) to Home Assistant's fabric. We're going to explore all these options below.

### Prerequisites

- Make sure you have the latest version of Home Assistant installed.
- On the device packaging, check for both the Matter logo and for either the Wi-Fi or the Thread logo.
- Check if the QR code is only on the packaging or if it is also on the device.
  - If it is only on the packaging, snap a picture of the QR code and the device and store the image and the numerical code in a save place.
  - If you lose the QR code and disconnect the device at some point, you won't be able to connect to that device again without the QR code.
- If you are adding a Wi-Fi-based Matter device: Matter devices often use the 2.4&nbsp;GHz frequency for Wi-Fi. For this reason, make sure your phone is in the same 2.4&nbsp;GHz network where you want to operate your devices.
- In Home Assistant, have the Matter integration installed.
  - Go to {% my integrations title="**Settings** > **Devices & services**" %}.
  - Add the **Matter (BETA)** integration.
  - When prompted to **Select the connection method**:
    - If you run Home Assistant OS in a regular setup: select **Submit**.
      - This will install the official Matter server add-on.
    - If you are already running the Matter server in another add-on, in or a custom container:
      - Deselect the checkbox, then select **Submit**.
      - In the next step, provide the URL to your Matter server.
- Have either an Android or iPhone ready and Bluetooth enabled. For information why Bluetooth is required, refer to the section on [Bluetooth used during commissioning](#bluetooth-used-during-commissioning):
  - Android:
    - Have an Android phone (a full Android, not F-Droid).
    - Have the latest version of the Home Assistant Companion app installed.
    - Have Google Home app installed on the Android.
    - We are not going to add the new device to Google Home. The app is needed because Google included the Matter SDK there.
    - If you are using Thread: Make sure there is a Thread border router device (Nest Hub v2 or Nest Wi-Fi Pro) present in your home network.
  - iPhone
    - Have the iOS version 16 or higher
    - Have the latest version of the Home Assistant Companion app installed.
    - If you are using Thread: Make sure there is a Thread border router device (HomePod Mini or V2, Apple TV 4K) present in your home network.
- Make sure the device is in close range of the border router and your phone.

### To add a new device using the iOS Companion app

This will use the Bluetooth connection of your phone to add the device.

1. Open The Home Assistant app on your phone.
2. Go to {% my integrations title="**Settings** > **Devices & services**" %}.
3. On the **Devices** tab, select the **Add device** button.
4. Select **Add Matter device**.
5. Scan the QR-code of the Matter device with your phone camera or select **More options...** to manually enter the Commission code.
6. Select **Add to Home Assistant**.
   - This starts the commissioning process which may take a few minutes.
7. If you're adding a test board or beta device, you might get a prompt about an **Uncertified Accessory**. In this dialog, select **Add Anyway**.
8. If prompted, enter a custom **Accessory Name**.
   - You can type whatever you like here.
   - This is an internal reference for iOS. It won't be visible in Home Assistant.
   - After entering a name, select **Continue**.
9. Once the process is complete, select **Done**.
   - You are now redirected to the device page within Home Assistant. It is ready for use.

<lite-youtube videoid="8y79Kq3QfCQ" videotitle="Add Matter device via iOS app in Home Assistant"></lite-youtube>

### To add a new device using the Android Companion app

This will use the Bluetooth connection of your phone to add the device.

1. Open The Home Assistant app on your phone.
2. Power up the device by plugging it in or add a battery. Most devices will now go into pairing mode.
   - For some devices, you need to enable a pairing mode (like you do with Z-Wave or Zigbee device).
   - The instructions on how to set the device in pairing mode can usually be found in the device documentation.
3. For some devices, at this point, your phone shows a pop-up, prompting you to **Scan the QR code**.
   - Scan the QR code.
   - When prompted to **Choose an app**, make sure to select Home Assistant.
   - Once the process is complete, select **Done**, then select **Add device**.
4. If you did not see a pop-up, go to {% my integrations title="**Settings** > **Devices & Services**" %}.
   - On the **Devices** tab, select the **Add device** button.
   - Select **Add Matter device**.
   - Scan the QR-code of the Matter device with your phone camera or select **Setup without QR-code** to manually enter the commission code.
      - This starts the commissioning process which may take a few minutes.
   - If you're adding a test board (e.g. ESP32 running the example apps) and commissioning fails, you might need to take some actions in the Google Developer console, have a look at any instructions for your test device.
   - Once the process is complete, select **Done**.
5. To view the device details, go to {% my integrations title="**Settings** > **Devices & Services**" %} and select the **Matter** integration.
6. By default, the device gets a factory specified name. To rename it, on the device page, select the pencil to edit and rename the device.
   ![image](/images/integrations/matter/matter-android-rename.png)
7. Your device is now ready to use.

<lite-youtube videoid="Fk0n0r0eKcE" videotitle="Add Matter device via Android app in Home Assistant"></lite-youtube>

### Share a device from Apple Home

This method will allow you to select a Matter device from Apple Home and share it to Home Assistant. The result is that the device can be controlled from both Apple Home and Home Assistant at the same time.

1.  Find your device in Apple Home and press the jogwheel to edit it. In the page with detailed descriptions and settings for the device, scroll all the way down and press the button **Turn On Pairing Mode**.
2.  You are now given a Setup code, copy this to the clipboard.
3.  Follow the [Add a device using the iOS Companion app](#add-a-device-using-the-ios-companion-app) directions above to add the device to Home Assistant where you paste the code you just received from Apple Home.

<lite-youtube videoid="nyGyZv90jnQ" videotitle="Share Matter device from Apple Home to Home Assistant"></lite-youtube>

### Share a device from Google Home

This method will allow you to share a device that was added to Google Home to Home Assistant. The result is that the device can be controlled from both Google Home and Home Assistant at the same time.

1.  Open the device in Google Home and press the settings button (jog wheel) in the top right.
2.  Click **Linked Matter apps and services**.
3.  Press the button **Link apps and services** to link the device to Home Assistant.
4.  Choose Home Assistant from the list, you are redirected to the Home Assistant Companion app now. Press **Add device**.
5.  Your device will now be added to Home Assistant. When the process finishes, you're redirected to the device page in Home Assistant.

<lite-youtube videoid="-B4WWevd2JI" videotitle="Share Matter device from Google Home to Home Assistant"></lite-youtube>


## Experiment with Matter using a ESP32 dev board

You do not yet have any Matter-compatible hardware but you do like to try it out or maybe create your own DIY Matter device? We have [prepared a page for you](https://nabucasa.github.io/matter-example-apps/) where you can easily flash Matter firmware to a supported ESP32 development board. We recommend the M5 Stamp C3 device running the Lighting app.

NOTE for Android users: You need to follow the instructions at the bottom of the page to add the test device to the Google developer console, otherwise {% term commissioning %} will fail. iOS users will not have this issue but they will get a prompt during {% term commissioning %} asking if you trust the development device.

1. Make sure you use Google Chrome or Microsoft Edge browser.
2. Open https://nabucasa.github.io/matter-example-apps/
3. Attach the ESP32 device using a USB cable.
4. Select the radio button next to the example you like to set up, in case of an M5 Stamp, click **Lighting app for M5STAMP C3**.
5. Select **Connect**.
6. In the popup dialog that appears, choose the correct serial device. This will usually be something like "cu-usbserial" or alike.
7. Click **Install Matter Lighting app example** and let it install the firmware on the device. This will take a few minutes.
8. Once the device is flashed with the Matter firmware, connect to the device again but this time choose **Logs & console**.
9. You are presented with a console interface where you see live logging of events. This is an interactive shell where you can type commands. For a list of all commands, type **matter help** and press enter.
10. To add the device, we need the QR code. In the console, type in `matter onboardingcodes ble` and copy/paste the URL into your browser.
11. Use the QR code to add the device using one of the above instructions on your phone, e.g. using the Home Assistant Companion app.

## Troubleshooting

### General recommendations

- Using Thread-based Matter devices in Home Assistant requires Home Assistant OS version 10 and above. Not using Home Assistant OS is at your own risk. We do provide some [documentation](https://github.com/home-assistant-libs/python-matter-server/blob/main/README.md) on how to run the Matter Server as a Docker container. The documentation includes a description of the host and networking requirements.

- To use Thread devices you will need a Thread Network with at least one Thread Border Router in your network nearby the Thread device(s). Apple users need for example the Apple TV 4K or the HomePod Mini, while Google users need a Nest Hub V2. Use the Thread integration in Home Assistant to diagnose your Thread network(s).

- Start simple and work from there, keep your network simple and add for example an ESP32 test device. Once that works, move on to the next step or more devices.

- Realize that you are an early adopter, both on the hardware side and on the software (controller) side so you may run into compatibility issues or features that are still missing. Report any issues you may find and help out others if you find a workaround or tested a device.

- Make sure IPv6 (multicast) traffic travels freely from your network to the Home Assistant host. There is no requirement to have an IPv6-enabled internet connection or DHCPv6 server. However, IPv6 support has to be enabled on Home Assistant. Go to **{% my network title="Settings > System > Network" %}**, and make sure **IPv6** is set to **Automatic** or **static**, depending on your network setup. If you're unsure, use **Automatic**.

- For more detailed information on network configuration, refer to the [README of the Matter server repository](https://github.com/home-assistant-libs/python-matter-server/blob/main/README.md).

### I do not see the button "Commission using the Companion app"

This button will only be visible within the Home Assistant Companion App (so not in the browser) and your device meets all requirements for Matter support.

- For iOS, minimum version is iOS 16 (minimal 16.3 is preferred) and the most recent version of the HA companion app.
- For Android, minimum version is 8.1 and the most recent version of the (full) HA Companion app, downloaded from the Play Store.

### When I'm trying to commission using the Android app, I get an error stating "Matter is currently unavailable"

See above, make sure your device meets all requirements to support Matter. Update Android to the latest version and the Home Assistant Companion app. To quickly verify if your device meets all requirements to support Matter, on your Android device, go to **Settings** > **Google** > **Devices & Sharing**. There should be an entry there for **Matter devices**.

Some users have reported that uninstalling and reinstalling the Google Home app fixed this issue for them.
Also see this [extended troubleshooting guide](https://developers.home.google.com/matter/verify-services) from Google.

### Unable to commission devices, it keeps giving errors or stops working randomly

The Matter protocol relies on (local) IPv6 and mDNS (multicast traffic) which should be able to travel freely in your network. Matter devices that use Wi-Fi (including Thread Border routers) must be on the same LAN/VLAN as Home Assistant. Matter devices that only use Thread must be joined to Thread networks for which there is at least one border router connected to the Home Assistant LAN.

If you experience any issues with discovering devices (for example, if the initial {% term commissioning %} keeps failing or if devices become unavailable randomly), investigate your network topology. For instance, a setting on your router or Wi-Fi access point to "optimize" multicast traffic can harm the (discovery) traffic from Matter devices. Keep this in mind when you experience issues trying to add or control Matter devices. Protocols like Matter are designed for regular residential network setups and may not integrate well with enterprise networking solutions like VLANs, Multicast filtering, and (malfunctioning) IGMP snooping. To avoid issues, try to keep your network topology as simple and flat as possible.
