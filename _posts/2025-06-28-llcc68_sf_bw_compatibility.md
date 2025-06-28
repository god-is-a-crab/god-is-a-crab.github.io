---
layout: post
title:  Semtech LLCC68 SF-BW (Spreading Factor - Bandwidth) Compatibility
date:   2025-06-28 13:15:47 +1000
categories: LoRa
---

The Semtech LLCC68 does not support all spreading factors with all bandwidths. Source: see the note under Table 6-1 of the LLCC68 datasheet.

> 1. Not all SF are available for any bandwidth with the LLCC68

This is the **only** mention of it in the entire datasheet.

I've attached a screenshot of what this looks like in a logic analyzer.

![](/assets/images/llcc68_sf10_bw125_analyzer.png)

After the `SetModulationParams` command (OPCODE `0x8B`),
the status becomes `0xB8`, which according to the datasheet 13.5.1, bits 3:1 are equal to `0x4` which means "Command processing error". 

I bought an LLCC68 module off Aliexpress expecting to use SF10 with 125Khz BW - but it does not work. SF10 with 250Khz BW does work however.
