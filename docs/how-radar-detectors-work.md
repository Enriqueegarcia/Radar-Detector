# How Radar Detectors Work

## 1. What a detector is listening for

Police speed guns work by sending out energy and measuring what bounces back off a car:

- **Radar guns** send a continuous microwave signal. The reflection from a moving car comes back at a slightly shifted frequency (the Doppler effect), and that shift gives the speed.
- **Laser (LIDAR) guns** fire short infrared light pulses. They time how long each pulse takes to come back and work out the speed from how fast that distance changes.

A radar detector is a **receive-only** device. It doesn't need to measure speed. It only needs to notice that a police signal is present, ideally while that signal is still far away or bouncing off cars ahead of you.

## 2. Frequencies and bands

| Band | Frequency | Where / notes |
|------|-----------|---------------|
| **X** | 10.500 – 10.550 GHz (usually 10.525 GHz) | Old US radar; still used in a few areas. Also used by automatic door openers, a classic source of false alerts. |
| **Ku** | ~13.450 GHz | Mostly Europe; rare in the US. |
| **K** | 24.050 – 24.250 GHz (usually 24.125 / 24.150 GHz) | Common in the US and worldwide. Also used by car blind-spot and adaptive-cruise sensors, today's biggest false-alert source. Low-power photo-enforcement radars (e.g. MultaRadar "MRCD/MRCT") work here too. |
| **Ka** | 33.4 – 36.0 GHz | The main modern US police band. Common frequencies: ~33.8, 34.0, 34.3, 34.7, 35.5, 35.9 GHz. Narrow beam, hardest to detect early. |
| **Laser / LIDAR** | ~904–905 nm infrared light (not radio) | Very narrow beam. When a detector sees it, the speed has usually been measured already. |

**POP / instant-on modes:** many guns sit silent and only transmit for a fraction of a second when the officer fires them. Detectors pick these up mainly from earlier shots at cars ahead of you.

## 3. Inside a radar detector

```
 Horn antenna ──► Mixer ◄── Swept local oscillator (LO)
                    │
                    ▼
          Intermediate frequency (IF)
                    │
          Filters + amplifier + detector
                    │
                    ▼
     Microcontroller / DSP ──► Alert (beep, voice, LED, Bluetooth)
                    ▲
           GPS (lockouts, speed), settings
```

1. **Horn antenna.** A small metal horn, tuned to roughly 10–36 GHz, collects microwave energy from ahead (and from behind in some models).
2. **Superheterodyne receiver.** A mixer combines the incoming signal with a local oscillator (LO). The LO repeatedly sweeps its frequency. Many designs run around 11.5 GHz and use its harmonics to cover the K and Ka bands. When a real signal is present, a lower "intermediate frequency" comes out of the mixer that is easy to amplify and measure.
3. **Sweep and identify.** The microcontroller knows where the LO was in its sweep when a signal appeared, so it can work out the band and the exact frequency (e.g. "Ka 34.7").
4. **False-alert filtering.** This is what separates good detectors from cheap ones:
   - Signature filtering that recognises car blind-spot sensors (K band) and door openers (X band).
   - GPS lockouts that learn and mute fixed false sources at known locations.
   - Muting below a set speed, and filtering for signal strength and how it changes over time.
5. **Laser sensor.** Separate photodiodes behind infrared filters watch for 904 nm pulse trains at the right pulse rates.

### Side effect: detectors leak a signal

The swept LO leaks a little energy out of the antenna. **Radar-detector detectors** (e.g. the VG-2 / Spectre types used in places where detectors are banned) pick up that leak. "Stealth" detectors shield their LO to stay hidden.

## 4. Why range matters

A detector is useful because radar beams spread out and reflect. It can hear a gun aimed at another car well before you're in range yourself. Ka band and laser have narrow beams, so detection range is much shorter. With laser, the alert usually comes too late to act on.

## 5. Legal notes (check your jurisdiction)

- **US:** using a receive-only radar detector is legal in private vehicles in most states. It is **illegal in Virginia and Washington DC**, in commercial vehicles over 10,000 lb (federal rule), and on military bases.
- **Many countries ban or restrict them**, including much of Europe and parts of Canada and Australia. Some also ban GPS speed-camera alerts.
- **Radar and laser jammers are illegal** under US federal law (FCC) and almost everywhere else, and this project won't build one. Everything here is receive-only.
