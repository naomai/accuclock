# ESP AccuClock
Atomic time accuracy for your wall clock.

Many of modern devices use a special signal for their time keeping. The signal comes from a special crystal, which is vibrating at a very high rate - in a perfect world, it should be exactly 32768 times a second. 

Unfortunately, that rate might be varying. The temperature of the device, and imperfections in production might contribute to small drifts. In the scale of weeks, this can add up to few seconds of difference.

This project aims to create a more stable replacement for the resonating crystals. As a clock reference, we're going to use atomic time, publicly available on the internet. The hardware part is a ESP8266 board, which is perfect for this task - it has WiFi capability, and is very cheap.
