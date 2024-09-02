# AccuClock-ESP

Atomic time accuracy for your wall clock.

Many of modern devices use a special signal for their time keeping. The signal comes from a small component, called **crystal oscillator**, which is vibrating at a very high rate. *In a perfect world, it should be exactly 32768 times a second.*

Unfortunately, that rate might be varying. The temperature of the device, and imperfections in manufacturing might contribute to small drifts. *In the scale of week, your clock could be off by few seconds.*

This project aims to create a more stable replacement for the resonating crystals. As a reference for our internal clock, we're going to use atomic time, publicly available on the internet - via **NTP protocol**. The hardware part is a **ESP8266** board, which is perfect for this task - it has WiFi capability, and is *very cheap*.

## Several things to consider

* To avoid confusion:
  * **CPU cycles** refer to smallest time division a CPU can operate on. There are 80 million CPU cycles per second on ESP8266 in stock configuration.
  * **Wave period** is a time in which a output signal (waveform) goes through a full cycle - beginning of HIGH state, then changing from HIGH to LOW, and ending exactly before next ramp to HIGH. The output signal is fed to the circuitry of clock, and should have on average 32768 wave periods per second.
* ESP drives its pins with 3.3 volts. This might be too much for the circuitry of the clocks, which expects weak sine signal for amplification.
* ESP provides a library for generating waveforms, which are not disturbed by other tasks ran by RTOS.
  * The frequency seems to be stable, but its generation is tied to CPU cycles. This means that in khz range, we can only tune to frequencies with 20-ish Hz accuracy.
    ```
    System clock is 80MHz
    80000000/32768 = 2441.40625 cpu cycles per one wave period
    ```
  * We should dynamically switch between two closest freqencies, while keeping track of how many wave periods were lost/overran.
    ```
    2441 ccl = 32773 Hz
    2442 ccl = 32760 Hz
    ```
  * Our aim is not to have the exact frequency at all times, but to provide a signal which, on average, has the same number of wave periods over a longer span of time.
  * The internal timekeeping should be based solely on CPU clock. ~~I cannot find clear information whether ESP uses additional RTC clock, or bases its `time` library on CPU clock.~~ My first implementation shows drift that is significant, regular, but too small to be related to waveform generation.
  [Post suggesting that the RTC is emulated](https://www.esp8266.com/viewtopic.php?p=10180)
    * My tests show that timekeeping on ESP is in fact tied to the main clock, as there is no noticeable deviation between CCOUNT and system_get_time() on the long run.
    [My test function](https://gist.github.com/naomai/c6b7b8c9e3b7b3faf1c17dee4658644f)
    
* For the best accuracy, use NTP library with millisecond sync ablility. [ESPNtpClient](https://github.com/gmag11/ESPNtpClient)
  * platformio toolchain 4.2.1 causes crashes during hostname lookup, `platform = espressif8266@4.0.1` seems to work
* Adjusting the time after sync is done by slightly slowing down/speeding up the wave frequency over some time span (30s). *The watch should never stop, or get into hyperspeed. In fact, time sync should not even be noticeable by the user.*
* Should we also implement adjusting for daylight saving time?
* To save power, disable WiFi radio after sync - need to further investigate the behavior of ESPNtpClient.
* Create a test equipment to measure stability of DUT.
  * I will be using ESP32, with built in PCNT hardware pulse counter - minimal coding, and reliable counting of wave periods.

There is a similar project for analog clocks - [ESP8266-WiFi-Analog-Clock](https://github.com/jim11662418/ESP8266-WiFi-Analog-Clock). The program skips the oscillator part by completely replacing the electronics inside clock - manually controls the movement of second hand.
  

