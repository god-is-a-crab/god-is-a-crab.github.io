---
layout: post
title:  LSE Stability vs Supply Voltage Stability and LSE Drive
date:   2026-03-04 17:24:00 +1000
categories: PCB
---

There is an interesting relationship between the stability of a crystal oscillator and the stability of its supply voltage and MCU drive capability. Here I've used the [ABS07-32.768KHZ-1-T](https://abracon.com/parametric/crystals/ABS07-32.768KHZ-1-T) on an STM32L4 board.

## LSI with unstable voltage
![](/assets/images/lse_stability/lsi_unstable_supply_lowest_drive.png)

For reference, this is the LSI output with an unstable voltage (voltage drifting between 2.7V - 3.0V) sampled by a logic analyzer. Despite the unstable voltage, the LSI is relatively stable with only a 100 Hz difference between the Fmin and Fmax. It has a bit of inacurracy with a 200 Hz difference from its 32 kHz specification.

## LSE with stable voltage and lowest drive
![](/assets/images/lse_stability/lse_stable_supply_lowest_drive.png)
With a stable voltage (3.3V with little drift) and the lowest LSE drive (LSEDRV), the LSE is not as stable as the LSI with about 250 Hz difference between Fmin and Fmax. The accuracy is good however, being very close to 32.768 kHz.

## LSE with unstable voltage and lowest drive
![](/assets/images/lse_stability/lse_unstable_supply_lowest_drive.png)
With an unstable voltage and the lowest drive, the LSE is extremely unstable and inaccurate with a significant 3.5 kHz difference between Fmin and Fmax.

## LSE with stable voltage and highest drive
![](/assets/images/lse_stability/lse_stable_supply_highest_drive.png)
Increasing the drive shows negligible improvement in LSE stability with a stable voltage.

## LSE with unstable voltage and highest drive
![](/assets/images/lse_stability/lse_unstable_supply_highest_drive.png)
Increasing the drive significantly improves LSE stability with an unstable voltage.
