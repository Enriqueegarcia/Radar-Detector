# Plan: Radar Detection with an iPhone + Wireless Adapter

## 1. The hard constraint first

**An iPhone and a Wi-Fi adapter can't pick up police radar on their own.**

| Device | What it can receive | Police radar bands |
|--------|--------------------|--------------------|
| iPhone Wi-Fi | 2.4 / 5 / 6 GHz (up to ~7.1 GHz), and only valid Wi-Fi packets | X 10.5, K 24.1, Ka 33.4–36 GHz |
| iPhone Bluetooth | 2.4 GHz | — |
| iPhone UWB | ~6.5 / 8 GHz, Apple protocols only | — |
| iPhone cellular / 5G mmWave | Carrier bands only (mmWave ~28/39 GHz on some US models) | — |
| USB/Wi-Fi adapter | 2.4 / 5 / 6 GHz | — |

Three separate reasons:

1. **Wrong frequencies.** Every radio in these devices sits below 10 GHz, apart from cellular mmWave. Their antennas, amplifiers and filters block anything outside their own bands.
2. **No raw radio access.** iOS gives apps no raw RF data. Even a Wi-Fi card in monitor mode only reports decoded 802.11 packets, not "there's energy at frequency X".
3. **No USB Wi-Fi on iOS.** iPhones don't support plugging in USB Wi-Fi adapters for this kind of use.

So the plan uses the iPhone as the **brain and display** (GPS, map, alerts, logging, lockouts). The **"wireless adapter" is a small Bluetooth (BLE) board** that relays alerts from a sensor that actually can see radar or laser.

```
 [ Radar/laser sensor ] ──► [ ESP32 BLE board ] ──BLE──► [ iPhone app ]
                                                          GPS · alerts · lockouts · map · log
```

## 2. Options, from easiest to hardest

### Option A: Off-the-shelf Bluetooth detector + iPhone (recommended starting point)
- Buy a detector with Bluetooth and an iOS app, e.g. Escort (Drive Smarter app), Cobra (iRadar), or Uniden/Radenso models with app support.
- The detector does the RF work. The phone adds crowd-sourced alerts, GPS lockouts and maps.
- **Effort:** none. **Cost:** ~$150–$700. **Works on:** X / K / Ka / laser.

### Option B: Any detector + DIY BLE bridge + our own iPhone app (best fit for this repo)
- Take an ordinary detector and read its alert output: the speaker or alert LED, through an optocoupler or audio-level sensing, or its data port if it has one.
- An **ESP32** board (~$5–10) turns that into BLE messages ("alert on/off, strength, band if known").
- Our **iOS app** (Swift / SwiftUI) receives them and adds:
  - CoreBluetooth connection to the ESP32
  - CoreLocation GPS: tags every alert with location and speed
  - **GPS lockouts**: auto-mutes alerts that keep repeating at the same spot (door openers, store sensors)
  - Speed-based muting, spoken alerts, a map of alert history, and CSV export
- **Effort:** moderate. **Cost:** ~$60–$150 (cheap detector + ESP32). **Works on:** whatever the detector sees.

### Option C: DIY laser (LIDAR) sensor
- Laser is light, not microwaves, so a hobbyist can build the sensor itself:
  - A 904 nm-sensitive photodiode (e.g. BPW34 / SFH 213) behind an IR-pass filter
  - A fast transimpedance amplifier and comparator
  - An ESP32 that measures the pulse rate and sends it to the iPhone over BLE
- **Caveat:** laser beams are narrow, so an alert usually means you've already been measured. Treat this as a learning project, not real protection.
- **Effort:** moderate. **Cost:** ~$20–$40.

### Option D: Real DIY microwave receiver (advanced / research)
- A software-defined radio (SDR) such as HackRF (tops out at 6 GHz) or RTL-SDR (~1.7 GHz) can't reach these bands directly. It needs a **downconverter** in front:
  - **X band:** a 10 GHz amateur-radio receive converter or a modified satellite LNB with a horn
  - **K / Ka band:** a 24 GHz or 34 GHz receive mixer + LO + horn. These parts are expensive and hard to work with.
- Signal processing (FFT, threshold, band ID) runs on a Raspberry Pi, which sends alerts to the iPhone over BLE.
- **Must be receive-only.** Don't use transmitting Doppler modules (e.g. HB100 at 10.525 GHz, CDM324 at 24 GHz) as "receivers": they transmit, which causes interference and can break FCC rules.
- **Effort:** high. **Cost:** $200–$1000+. Sensitivity will be far below a commercial detector.

### Option E: No hardware
- Crowd-sourced speed-trap reports in Waze / Apple Maps / Google Maps. Our app could combine these with Option B alerts later.

## 3. Proposed build for this repo (Option B, then C)

**Phase 1: iPhone app skeleton**
- SwiftUI app: alert screen, settings, log
- CoreLocation GPS speed and position; background location for driving
- Simulated alert source for testing without hardware

**Phase 2: BLE bridge**
- ESP32 firmware (Arduino/PlatformIO): a BLE GATT service with an "alert" characteristic (band, strength, timestamp)
- Input circuit: optocoupler on the detector's alert LED/speaker, or an analog read of alert volume
- App: CoreBluetooth scan / connect / auto-reconnect

**Phase 3: Smart features**
- GPS lockouts (learned false-alert locations)
- Mute below X mph; spoken alerts (AVSpeechSynthesizer)
- Map of alert history; CSV export

**Phase 4 (optional): DIY laser sensor**
- Photodiode front-end + pulse measurement on the ESP32, using the same BLE alert format

**Proposed repo layout**
```
ios/        SwiftUI app (Xcode project)
firmware/   ESP32 BLE bridge (PlatformIO)
hardware/   Schematics: detector tap, laser sensor
docs/       Research + plan (this folder)
```

**What you'll need:** a Mac with Xcode to build and install the iOS app (this cloud session can write the code but can't build or run iOS apps), an ESP32 dev board, and a donor radar detector.

## 4. Legal checklist
- Check the detector laws where you drive (banned in Virginia and Washington DC, in commercial vehicles, and in many countries).
- Receive-only. No jammers and no transmitting modules.
- Keep the app glanceable and voice-first so it doesn't distract while driving.
