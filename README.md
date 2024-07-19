# ESP AccuClock
Atomic time accuracy for your wall clock.

Many of modern devices use a special signal for their time keeping. The signal comes from a small component, called *crystal oscillator*, which is vibrating at a very high rate - in a perfect world, it should be exactly 32768 times a second. 

Unfortunately, that rate might be varying. The temperature of the device, and imperfections in manufacturing might contribute to small drifts. In the scale of week, your clock could be off by few seconds.

This project aims to create a more stable replacement for the resonating crystals. As a clock reference, we're going to use atomic time, publicly available on the internet. The hardware part is a ESP8266 board, which is perfect for this task - it has WiFi capability, and is very cheap.

## Several things to consider

* ESP drives its pins with 3.3 volts. This might be too much for the circuitry of the clocks, which expects weak sine signal for amplification.
* ESP provides a library for generating waveforms, which are not disturbed by other tasks ran by RTOS.
  * The frequency seems to be stable, but its generation is tied to CPU cycles. This means that in khz range, we can only tune to frequencies with 20-ish Hz accuracy.
  * We should dynamically switch between two closest freqencies, while keeping track of how many wave cycles were lost/overran.
  * Our aim is not to have the exact frequency at all times, but to provide a signal which, on average, has the same number of cycles over a longer period of time.
* For the best accuracy, use NTP library with millisecond sync ablility.
* Adjusting the clock after sync is done by slightly slowing down/speeding up the frequency over some period (30s)
* Create a test program for second ESP, which will be used for measuring the stability.
  

