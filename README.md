# PicoMicropython_HowDoI
Some tips for me, myself and I, and everyone who wants to use them

## Non blocking structure for timing in loops
This example shows how to display output values  that are periodically measured,  
with different timings and without a sleep functions that blocks in the main loop.
In addition it provides nbsecs that is incremented every second.


```python
#....

# Timing intervals:
interval_oled = 100
interval_print = 1000
interval_uart = 500


# Initialize timings:
now = time.ticks_ms()
last_oled = now
last_print = now
last_uart = now
last_second = now

nbsecs = 0

# main loop:
while True:

    now = time.ticks_ms()
    measure()

    # every second:
    if time.ticks_diff(now, last_second) >= 1000:
        last_second = now
        nbsecs += 1

    # print
    if time.ticks_diff(now, last_print) >= interval_print:
        led.value(1)
        print_values()
        last_print = now
        led.value(0)

    # oled
    if time.ticks_diff(now, last_oled) >= interval_oled:
       last_oled = now
       oled_values()

    # uart 
    if time.ticks_diff(now, last_uart) >= interval_uart:
        last_uart = now
        uart_values()

    # ...
```
