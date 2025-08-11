---
layout: post
title:  Low-Power LoRa Wakeup Protocols
date:   2025-08-11 15:16:00 +1000
categories: LoRa
---

## Event Loops and Wakeup

A battery-powered device with a low duty-cycle typically uses the _event loop_ design pattern - the device sleeps until it is required to perform an action. This pattern optimizes power usage because such a device should spend most of its time in a low-power sleep state and a small amount of time awake. The device is woken up by an event, usually some sort of hardware interrupt. This post expores the scenario where the event to wake up from is a signal from an external LoRa node (which I will call the gateway).

![Event loop diagram](/assets/images/event_loop.png)

## Assumptions & Requirements
Before discussing the two devised protocols, I will define some assumptions and requirements that went into creating these protocols.
- No time sychronization - I don't want to use an external crystal oscillator or introduce any time synchronization logic.
- TX duty cycle of the gateway is not important
- Gateway must identify itself to the device

## Analysis
For each protocol, I will compute the number of days a device could theoretically operate with that protocol. The power supply will be a 1.85 Wh battery. I will ignore parasitic losses in energy like voltage conversion loss and battery self-discharge. The LoRa parameters are:

- Frequency=434 Mhz
- SF9
- BW=125 kHz

This gives us the approximate values:
- TX current = 107 mA, [Semtech SX1268 datasheet table 3-5](https://semtech.my.salesforce.com/sfc/p/#E0000000JelG/a/RQ000008nGa9/v.ea8jp.cso4PrL7koonfOfzpF4krk_NglUXEPVQRzg)
- TX period @ SF9 = 103ms, [LoRa Calculator](https://www.semtech.com/design-support/lora-calculator)
- 4 symbol CAD, [AN1200.48 Table 1](https://semtech.my.salesforce.com/sfc/p/#E0000000JelG/a/3n000000qSr3/s723KzQi7zXFhBPKqesoF11.MtwYu6B8pdEQcmInlkE)
- 19.145 ms CAD duration, [AN1200.48 Table 96](https://semtech.my.salesforce.com/sfc/p/#E0000000JelG/a/3n000000qSr3/s723KzQi7zXFhBPKqesoF11.MtwYu6B8pdEQcmInlkE)
- 3.84 mA CAD current, [AN1200.48 Table 96](https://semtech.my.salesforce.com/sfc/p/#E0000000JelG/a/3n000000qSr3/s723KzQi7zXFhBPKqesoF11.MtwYu6B8pdEQcmInlkE)
- Sleep current = 22 uA (made-up number for a typical device)

## Protocol 1 - TX Beacon
With this approach, the device acts as a beacon, periodically transmitting a packet - the gateway listens in RX mode for the beacon and responds with an _ACK_ when it receives the beacon. Reception of this _ACK_ is the wakeup event for the device. 

![TX beacon diagram](/assets/images/tx_beacon.png)

#### Characteristics
- Power consumption: High because transmitting is energy expensive
- Latency: High because beacon period needs to be long to offset the expensive beacon transmits
- Device TX duty cycle: Medium - depends on beacon period
- Gateway TX duty cycle: Minimal

#### Operation Time Calculation

I will choose a 60 second period which gives an average latency of 30 seconds.

```
Active duration per minute (s) = 0.103
Sleep duration per minute (s) = 59.897
Pulse power usage (W) = 0.353
Sleep power usage (W) = 7.26e-05 (72.6 uW)
Active energy used per minute (J) = 0.0364 (36.4 mJ)
Sleep energy used per minute (J) = 0.004349 (4.35 mJ)
Total energy used per minute (J) = 0.04072 (40.72 mJ)
Battery energy = 6660 J
Minutes of operation = 163564
Days of operation = 113 days
```

## Protocol 2 - CAD Windows
This protocol makes use of the Channel Activity Detection (CAD) feature in Semtech LoRa transceivers - the device periodically enters CAD mode, allowing the gateway a small window to wake the device by transmitting. The gateway should loop between TX and CAD to try wake the device while also listening for a response from the device. There is a chance that the CAD window by the device doesn't detect the gateway's transmit because of the interval between transmits, this is why the device should do a second CAD if the first one doesn't detect anything. The completion of this handshake is the wakeup event.

![CAD windows diagram](/assets/images/cad_windows.png)

#### Characteristics
- Power consumption: Low - CAD is very cheap (~25x less current and 1/5 the duration for 4 symbol CAD)
- Latency: Low - can increase the frequency of CAD windows because CAD is cheaper which significantly improves latency
- Device TX duty cycle: Minimal, only transmits when activity detected
- Gateway TX duty cycle: High, recommended to not run the TX-CAD loop for longer than the CAD window period

I will choose a 10 second period which gives an average latency of 5 seconds.

```
Active duration per minute (s) = 0.22974
Sleep duration per minute (s) = 59.77026
Pulse power usage (W) = 0.012672 (12.7 mW)
Sleep power usage (W) = 7.26e-05 (72.6 uW)
Active energy used per minute (J) = 0.0029 (2.9 mJ)
Sleep energy used per minute (J) = 0.00434 (4.34 mJ)
Total energy used per minute (J) = 0.00725 (7.25 mJ)
Battery energy = 6660 J
Minutes of operation = 918546
Days of operation = 637 days
```

## Appendix

#### Script to calculate operation time
```python
def compute_days_of_operation(pulse_current, pulse_duration, pulse_per_minute):
    voltage = 3.3
    battery_energy = 1.85 * 3600
    sleep_current = 22e-6 # 22 uA
    active_duration = pulse_duration * pulse_per_minute
    sleep_duration = 60 - active_duration

    pulse_power = pulse_current * voltage
    sleep_power = sleep_current * voltage
    active_energy = pulse_power * active_duration
    sleep_energy = sleep_power * sleep_duration
    energy_per_minute = sleep_energy + active_energy
    minutes_of_operation = battery_energy / energy_per_minute
    days_of_operation = minutes_of_operation / 60 / 24

    print("Pulses per minute =", pulse_per_minute)
    print("Active duration per minute (s) =", active_duration)
    print("Sleep duration per minute (s) =", sleep_duration)
    print("Pulse power usage (W) =", pulse_power)
    print("Sleep power usage (W) =", sleep_power)
    print("Active energy used per minute (J) =", active_energy)
    print("Sleep energy used per minute (J) =", sleep_energy)
    print("Total energy used per minute (J) =", energy_per_minute)
    print("Minutes of operation =", minutes_of_operation)
    print("Days of operation =", days_of_operation)

if __name__ == "__main__":
    print("# TX beacon")
    compute_days_of_operation(0.107, 0.103, 1)
    print("")
    print("# CAD windows")
    compute_days_of_operation(0.00384, 0.019145, 12)
```
