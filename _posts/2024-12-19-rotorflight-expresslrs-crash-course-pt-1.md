---
layout: post
title:  "Rotorflight and ExpressLRS Crash Course - Part 1"
date:   2024-12-19 10:00:00 -0800
categories: rotorflight expresslrs
---
# Rotorflight and ExpressLRS Crash Course - Part 1

This is a crash course to hopefully get you off the ground and flying with Rotorflight using ExpressLRS as your communication link. To keep things as simple as possible, I will leave out the extremely nitty gritty details and only mention and define terms that are relevant to the day-to-day use/setup of Rotorflight. I will do my best to draw real-life analogies with the hopes it helps to simply understanding the stack.

From the "control" perspective, there are a few layers that are important to be familiar with. 

| Component | Description | Examples | 
| -------------- | ----------- | --------|
| Transmitter (TX, Radio)   | If you already fly RC models, you are likely already familiar with your transmitter, but for those unfamiliar, the device that accepts physical input from the user is commonly referred to as the transmitter. Some folks also receiver to the transmitter as a "radio", and as such, radio is a very overloaded term. | RadioMaster TX16S MKII / FrSky X18RS, FrSky X20RS |
| Transmitter Module (TX Module) | The transmitter module is either built into your transmitter (for ELRS, this can be built-in if you own a RadioMaster transmitter with a built-in transmitter module), or attached to the back of your transmitter as a "backpack" | RadioMaster Nomad / BetaFPV SuperG Nano / FrSky Vantac |
| Receiver (RX) | The receiver is the component that is mounted to your model that communicates with the transmitter module. It then communicates over a serial link to the flight controller (which in this case, would be running Rotorflight) | RadioMaster RP2, RP3-H / FlyDragonRC R24D |
| Flight Controller (FC/FBL) | The flight controller is the "brain" of your helicopter model. It communicates with the receiver and drives the servos of the model | RadioMaster Nexus, FlyDragon F722V2.2 | 

ExpressLRS, commonly abbreviated to ELRS, is an extremely popular open source radio control link that can be used to provide radio control experiences targeting a range from long-distance (more commonly used in FPV) to low-latency (what we're after in the RC helicopter world). For analogy purposes, you can think of ExpressLRS as a way to provide the ability to send text messages between phones.

You may have heard terms like "Crossfire" or "CRSF" thrown around. Crossfire, commonly abbreviated to CRSF, is a serial protocol that serves as a standard of a protocol between the receiver and flight controller. CRSF can be equated to the contents of the text message that is sent between phones. CRSF facilitates the return of telemetry data from the flight controller, and can even be used (via the telemetry link) to allow for forward programming (programming the flight controller from your transmitter) protocols which is how the Lua scripts work. 

To summarize, ExpressLRS and CRSF are the "communication link" between your transmitter and flight controller. For simplistic purposes most condense referral to them to simply "ELRS". In the phone model, the pilot (you), use your transmitter (phone) to send control input to your flight controller. Your flight controller is a phone that receives your text messages and interprets them, and provides responses.

## Links to Components

For the sake of putting pictures to words, here is a list of the common components and links to various vendors that sell these products. Note these are NOT affiliate links, but are links to retailers that have supported me through my journey in the hobby. My preferred retailers are [BuddyRC](https://www.buddyrc.com) and [New England RC](https://www.newenglandrc.us). Both have excellent customer support and quick shipping, and are small businesses that truly believe in keeping the future of the RC helicopter hobby alive.

Note that if you are purchasing a TX16S MKII, there are a LOT of options (more than just the internal module) so you should do research before committing to ensure you are purchasing what you want.

| Component | Type | Link |
| --------- | ------ | ---- |
| RadioMaster TX16S MKII ELRS | Transmitter | [BuddyRC](https://www.buddyrc.com/products/radiomaster-tx16s-mark-ii-radio-controller-mode-2?variant=42991349563630) |
| RadioMaster TX16S MKII 4-in-1 | Transmitter | [New England RC](https://newenglandrc.us/products/rms-0019-radiomaster-tx16s-mark-ii-radio-controller-4in1-mode-2) / [BuddyRC](https://www.buddyrc.com/products/radiomaster-tx16s-mark-ii-radio-controller-mode-2?variant=42991349530862) |
| RadioMaster TX16S Battery | Transmitter Battery | [New England RC](https://newenglandrc.us/products/ga-b-tx-4000-2s1p-jst?_pos=1&_psq=tx16+battery&_ss=e&_v=1.0) / [BuddyRC](https://www.buddyrc.com/products/radiomaster-tx16s-li-ion-battery) | 
| RadioMaster Ranger Micro ELRS TX Module | Transmitter Module | [New England RC](https://newenglandrc.us/products/rms-0034-radiomaster-ranger-micro-2-4ghz-elrs-module) / [BuddyRC](https://www.buddyrc.com/products/radiomaster-ranger-micro-2-4ghz-elrs-module-combo-package?variant=45280406470894) |
| RadioMaster Nexus | Flight Controller | [New England RC](https://newenglandrc.us/products/radiomaster-nexus-rotorflight2-helicopter-flight-controller) / [BuddyRC](https://www.buddyrc.com/products/nexus-helicopter-flybarless-flight-controller) |
| RadioMaster RP3-H | Receiver | [New England RC](https://newenglandrc.us/products/radiomaster-rp3-h-helicopter-nexus-rotorflight-elrs-receiver) / [BuddyRC](https://www.buddyrc.com/products/rp3-h-expresslrs-2-4ghz-nano-receiver) |
| FrSky X18RS | Transmitter | [New England RC](https://newenglandrc.us/products/frsky-tandem-x18rs) |
| FrSky X20RS | Transmitter | [New England RC](https://newenglandrc.us/products/frsky-tandem-x20rs) |
| FlyDragon F722v2.2 | Flight Controller + Receiver | [New England RC](https://newenglandrc.us/products/steam-fd-f722-fbl-steam-flydragon-rotorflight-f722-fbl-v2-2) |


## ExpressLRS Bind Phrase

In the conventional world of RC models, pilots are likely familiar with the binding process, which involved some type of sequence to facilitate binding the transmitter (module) and receiver together. With ExpressLRS, the binding process can be eliminated through the use of a `bind phrase`. 

The `bind phrase` is a way to pre-program binding a transmitter module and receiver. When you set it properly, when your transmitter (and module) + receiver are turned on, they will already be bound! It is a phrase that should be at least 8 characters, and MUST be unique to yourself. Note that it is NOT a password, and you should NOT use a password for it! 

Example bind phrases could be something like `iamoats87`, `iamchriskim`, `chriskimselrsbinding`, and so forth. Once you choose your unique bind phrase, it's best to stick with it, otherwise it can be a pain to run through all of your ELRS receivers to update them later on.

[More information on ExpressLRS Bind Phrases](https://www.expresslrs.org/quick-start/binding/#unique-phrase)

## Software Required

#### Common
* [ExpressLRS Configurator](https://github.com/ExpressLRS/ExpressLRS-Configurator/releases)
* [Rotorflight Configurator](https://github.com/rotorflight/rotorflight-configurator/releases)

##### FrSky
* [Ethos Suite](https://github.com/FrSkyRC/ETHOS-Feedback-Community/releases?q=%22Ethos+Suite*%22&expanded=true) (If you have a FrSky radio, if you are using EdgeTX/RadioMaster, you do not need this!)

#### Windows:

* [ImpulseRC DFU Driver Fixer](https://impulserc.blob.core.windows.net/utilities/ImpulseRC_Driver_Fixer.exe)
* [Windows .NET Framework 4.5]([https://www.microsoft.com/en-au/download/details.aspx?id=30653) (Only required if the Impulse DFU Driver Fixer does not work right!)

## Updating

1. Transmitter (EdgeTX/Ethos)
2. Transmitter Module (ExpressLRS)
3. Load ExpressLRS/Rotorflight Lua Scripts
4. Flight Controller (Rotorflight)
5. Receiver (ExpressLRS)

### 1. Transmitter

Depending on your transmitter, the procedure to update your transmitter differs. 

#### EdgeTX (RadioMaster)
Updating your radio will depend on the manufacturer of your radio. If you are running a RadioMaster radio, you will likely be using EdgeTX and need to update your radio with EdgeTX. Your radio will also announce "Welcome to EdgeTX" when it powers on.

EdgeTX provides a very convenient web-based companion tool called [EdgeTX Buddy](https://buddy.edgetx.org/)

For more information on updating EdgeTX, refer to the official EdgeTX manual. For convenience, I've linked to their quickstart guide for installing/updating EdgeTX here: [EdgeTX Quickstart Guide](https://manual.edgetx.org/installing-and-updating-edgetx)

#### Ethos (FrSky)
FrSky transmitters run an operating system called Ethos. Ethos is similar to EdgeTX, but has a lot of focus put on allowing it to be user friendly. For information on Ethos, you can find their information site here: [Ethos Website](https://ethos.frsky-rc.com/)

FrSky provides a tool called Ethos Suite, and there is a great video tutorial on updating Ethos here: [Ethos Suite Tutorial](https://www.youtube.com/watch?v=nObYVHEYXao) 

FrSky stores information for downloads on Ethos here: [ETHOS Feedback Community](https://github.com/FrSkyRC/ETHOS-Feedback-Community)

### 2. Transmitter Module

#### Overview

The transmitter module is either internal to your transmitter, or an external backpack. Depending on your configuration, you will either plug a USB cable into your transmitter (if the module is internal) or directly into the backpack for updates. For our purposes, we generally fly 2.4 GHz, so when searching for your corresponding transmitter module, you do NOT want to use the 900 MHz one.

#### Flashing using ExpressLRS Configurator

Using the ExpressLRS Configurator, on the `Configurator` tab, you can select your firmware version. It's best to keep your firmware versions in-sync, but there can be version skew (differences) on minor version to a certain extent. More information can be found on the ExpressLRS website around their targeted skew policy. 

| Module | ExpressLRS Configurator Device Category | ExpressLRS Configurator Device |
| ------ | --------------- | ------ |
| RadioMaster TX16S MKII Internal ELRS | RadioMaster 2.4 GHz | RadioMaster TX16S Internal 2.4GHz TX | 
| RadioMaster Boxer Internal ELRS | RadioMaster 2.4 GHz | RadioMaster Boxer Internal 2.4GHz TX |
| RadioMaster Zorro Internal ELRS | RadioMaster 2.4 GHz | RadioMaster Zorro Internal 2.4GHz TX |
| RadioMaster Ranger Micro | RadioMaster 2.4 GHz | RadioMaster Ranger Micro 2.4GHz TX |
| RadioMaster Nomad | RadioMaster | RadioMaster Nomad 2.4/900 TX |
| BetaFPV SuperG Nano | BETAFPV 2.4 GHz | BETAFPV SuperG 2.4 GHz Gemini TX |   
| FrSky Vantac ELRS TX Module | Vantac 2.4 GHz | Vantac Lite 2.4GHz TX | 

Once you've selected the correct target for your transmitter module and it is plugged in, you will need to configure some device options. If flying in the US, the `Regulatory domains 2.4 GHz band` option that is selected should be `2.4 GHz ISM (Standard)`. If you want to configure a bind phrase, select the checkbox, and fill in your desired bind phrase.

Optionally, you can specify your home WiFi network with the `Wi-Fi name (SSID)` and `Wi-Fi password` options. This is a very neat feature of ExpressLRS which has it automatically connect to your specified Wi-Fi network for on-the-fly configuration of the module after it has been flashed.

Note that the configurator will build the module's firmware from source, and flash it to the device. This means your bind phrase (if selected) and other options will be burned into the firmware, essentially customizing it for yourself.

##### EdgeTX (RadioMaster)

If you are flashing an internal TX module on an EdgeTX enabled transmitter, your transmitter should be powered on and the USB mode selected should be `USB Serial (Debug)`. Your Flashing Method should be `EdgeTX Passthrough`.

You can download a copy of the Lua scripts using the `DOWNLOAD LUA SCRIPTS` (we'll cover uploading them to the radio in the next section) which are required to enable your radio to communicate with your TX module. Note, a lot of RadioMaster radios already have a copy of the Lua scripts, but for posterity it's best to update the Lua scripts to match your flashed version of ExpressLRS to guarantee compatibility.

##### Ethos (FrSky)

If you are using Ethos, you do NOT use the Lua scripts provided by the ExpressLRS Configurator. I'll link to the correct places to get the Lua script in the next section.

#### Flashing
Under the Actions tab, you will select your transmitter (or transmitter module) in the serial selection list. If this is the first time you are flashing your module, or you have purchased your module second hand, it may be a good idea to select `Erase before flash`. `Force flash` is not generally needed except for on very old FlyDragon V2 units which were flashed incorrectly from the factory. Once this is done you can click `FLASH` and wait for the results.

### 3. Lua Scripts

You will need to update/upload Lua scripts to your transmitter to ensure that it can communicate with your transmitter module.

#### EdgeTX (RadioMaster)

##### ExpressLRS

If you have downloaded the Lua script, it will likely have been saved as `elrsV3.lua` on your computer. You will either need to pull the SD card out of your radio (make sure it is off before you do this!!!!) or connect your transmitter to your computer and place it in "USB Storage (SD)" mode. You will place your `elrsV3.lua` file in the folder found at `SCRIPTS` -> `TOOLS`. If using a RadioMaster radio, there will already be a `elrsV3.lua`, you can backup the original file and and overwrite this.

##### Rotorflight

If you are using Rotorflight, you can download the Rotorflight Lua scripts here: [Rotorflight Lua Scripts Release List](https://github.com/rotorflight/rotorflight-lua-scripts/releases/)

You should unzip the scripts and copy the `SCRIPTS` folder to the root of your radio storage, which will place the required scripts into their specific folders.

#### Ethos (FrSky)

##### ExpressLRS

Depending on your version of Ethos, you will need to use a different set of ExpressLRS scripts. You can find a file called `elrs.zip` at each 

[Ethos 1.5.x ELRS Lua](https://github.com/FrSkyRC/ETHOS-Feedback-Community/tree/1.5/lua/modules/elrs)

[Ethos 1.6.x ELRS Lua](https://github.com/FrSkyRC/ETHOS-Feedback-Community/tree/1.6/lua/modules/elrs)

You can download the Lua script for your specific firmware by clicking the dropdown in the top left (which will say `1.5` or `1.6`, and then searching for your corresponding version like `1.5.19`.

For convenience, here is a link to the latest ELRS zip based on version:

[Ethos ELRS Lua Download - 1.5.x](https://github.com/FrSkyRC/ETHOS-Feedback-Community/raw/refs/heads/1.5/lua/modules/elrs/elrs.zip)

[Ethos ELRS Lua Download - 1.5.19](https://github.com/FrSkyRC/ETHOS-Feedback-Community/raw/refs/tags/1.5.19/lua/modules/elrs/elrs.zip)

[Ethos ELRS Lua Download - 1.6.x](https://github.com/FrSkyRC/ETHOS-Feedback-Community/raw/refs/heads/1.6/lua/modules/elrs/elrs.zip)

Using `Ethos Suite`, you can upload the zip file to your radio under `Lua development tools` -> `Install Lua scripts` -> `OPEN .ZIP`

##### Rotorflight

If you are using Rotorflight, you can download the Rotorflight Ethos Lua scripts here: [Rotorflight Lua Scripts Release List](https://github.com/rotorflight/rotorflight-lua-scripts/releases/)[Rotorflight Ethos Lua Scripts Release List](https://github.com/rotorflight/rotorflight-lua-ethos/releases/)

Using `Ethos Suite`, you can upload the zip file to your radio under `Lua development tools` -> `Install Lua scripts` -> `OPEN .ZIP`

### 4. Flight Controller

Updating your flight controller is quite easy. Note, if you are on a Windows laptop, you may need to run the [ImpulseRC Driver Fixer](https://impulserc.blob.core.windows.net/utilities/ImpulseRC_Driver_Fixer.exe) to enable updating Rotorflight using the configurator. Personally, I was only able to get the driver fixer to work while I had a flight controller actively plugged into my laptop, but your experience may vary.

Using the Rotorflight configurator, plug in your flight controller, and select the `Firmware Flasher` tab. Depending on the version of the `Rotorflight Configurator` you have, you can click the `Detect` button, which should auto-select your board. If this is not available, here is a list of common flight controllers/boards

| Flight Controller | Board |
| ------ | --------------- |
| FlyDragon V2 | FLYDRAGONF722_V2 | 
| FlyDragon V2.2 | FLYDRAGONF722\_V2_2 |
| RadioMaster Nexus | NEXUS_F7 |

If this is your first time flashing your flight controller, it may not be a bad idea to perform a `Full chip erase`, especially if your flight controller is running an older version of Rotorflight (for example, your flight controller has Rotorflight 2.0 (4.3.0) and you want to flash Rotorflight 2.1 (4.4.0).

Once your board is selected, click `Load Firmware [Online]` which will download the firmware for you. You can then hit `Flash Firmware` and this will flash the firmware to your flight controller.

### 5. Receiver

The receiver is either internal to your flight controller (for example, with a FlyDragon F722V2 or F722V2.2), or an external module (RadioMaster RP-3H). For our purposes, we generally fly 2.4 GHz, so when searching for your corresponding receiver, you do NOT want to use a 900 MHz option.

Using the ExpressLRS Configurator, on the `Configurator` tab, you can select your firmware version. It's best to keep your firmware versions in-sync, but there can be version skew (differences) on minor version to a certain extent. More information can be found on the ExpressLRS website around their targeted skew policy. 

| Module | ExpressLRS Configurator Device Category | ExpressLRS Configurator Device |
| ------ | --------------- | ------ |
| RadioMaster RP3-H | RadioMaster 2.4 GHz | Radiomaster RP3-H Diversity 2.4GHz RX | 
| FlyDragon F722V2 / F722V2.2 | FlyDragonRC 2.4 GHz | FD R24D 2.4GHz RX | 

If you are using Rotorflight, we can use the (very convenient) `Betaflight Passthrough` feature, which allows us to flash the module directly through Rotorflight.

Once you've selected the correct target for your transmitter module and it is plugged in, you will need to configure some device options. If flying in the US, the `Regulatory domains 2.4 GHz band` option that is selected should be `2.4 GHz ISM (Standard)`. If you want to configure a bind phrase, select the checkbox, and fill in your desired bind phrase. You must make sure your bind phrase (if specified) and regulatory domains match your transmitter module, otherwise things will not work!

On older versions of the Express LRS configurator, there was an option called `LOCK_ON_FIRST_CONNECTION` or `Lock after first connection`, which when enabled, saves the connection settings that were used by the transmitter module and receiver and provides a faster re-connection. I enable this feature.

Optionally, you can specify your home WiFi network with the `Wi-Fi name (SSID)` and `Wi-Fi password` options. This is a very neat feature of ExpressLRS which has it automatically connect to your specified Wi-Fi network for on-the-fly configuration of the module after it has been flashed. This provides the ability to update the receiver without having to plug it back into your computer, or even configure it using the ExpressLRS web configuration interface.

Note that the configurator will build the module's firmware from source, and flash it to the device. This means your bind phrase (if selected) and other options will be burned into the firmware, essentially customizing it for yourself. This is only relevant if you ever decide to sell your module later on.

At this point, you can plug your Rotorflight FBL into your computer. Make sure the Rotorflight configurator is not open or connected to the module. Select your flight controller in the `Manual serial device selection` window, and hit `FLASH`!

If this is the first time you are flashing your module, or you have purchased your module second hand, it may be a good idea to select `Erase before flash`, as the previous owner may have enabled . `Force flash` is not generally needed except for on very old FlyDragon V2 units which were flashed incorrectly from the factory. Once this is done you can click `FLASH` and wait for the results.