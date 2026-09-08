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


## Daisy chaining serial informations

![Picture](DaisyChain)

I prefer my measuring data travelling through cables under my control instead of going I don't know where into clouds or through WLAN.  
But then I have the problem that there are multiple serial outputs that should be bundled  into one for treatment, maybe by Home Assistant.
My data are usually text outputs that are tab separated for different values and terminated by a line feed.
The advantage is that I can easily plot them.

My solar stepup converter (module A) could for example output this:
```  
A	28996357	48.9	9.77	478	0.42	1158146	33.6	28.5	34.8	30.2	813	0	931.3946
A	28996359	48.82	9.82	479	0.42	1158146	33.7	28.7	35.1	30.2	813	0	954.4613
A	28996361	48.74	9.83	479	0.42	1158147	33.8	28.8	35.3	30.2	813	0	977.5269
```

whilest a double 100A Ampèremeter (module B) outputs values like this:

```  
60	-0.07	0	0.017	-0.007	0	0	4.82
62	-0.07	0	-0.023	-0.007	0	0	4.82
63	-0.07	0	0.044	-0.007	0	0	4.83

```

I want them fitted together like this:

```  
A	28996357	48.9	9.77	478	0.42	1158146	33.6	28.5	34.8	30.2	813	0	931.3946		60	-0.07	0	0.017	-0.007	0	0	4.82
A	28996359	48.82	9.82	479	0.42	1158146	33.7	28.7	35.1	30.2	813	0	954.4613		62	-0.07	0	-0.023	-0.007	0	0	4.82
A	28996361	48.74	9.83	479	0.42	1158147	33.8	28.8	35.3	30.2	813	0	977.5269		63	-0.07	0	0.044	-0.007	0	0	4.83

```

Module A outputs its values continuously every 2s.
Module B has a jumper to select between continuous output (for debugging) or output on demand, every time module A outputs a new line character.

```python
#....

uart = UART(0, baudrate=9600)
jmp_uart = Pin(2, Pin.IN, Pin.PULL_UP)

while True:
    
    #...
        
    # uart continuously when jumper set:
    if jmp_uart.value() == 0:
        if time.ticks_diff(now, last_uart) >= interval_uart:
            last_uart = now
            uart_values()
    
    else:       
        # jumper not set: uart values on demand, daisy chain with incoming data
        if uart.any():
            s = uart.read()
            
            if  '\n' in s:
                s = s.rstrip()
                uart.write(s + '\t\t')
                
                uart_values()
            else:
                uart.write(s)
    # ...
```


