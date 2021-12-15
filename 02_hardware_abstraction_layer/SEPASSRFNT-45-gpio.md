# GPIOs and LEDs

NOTE: For further design details, please also refer to [SEPASSRFNT-45](https://jira.open-groupe.com/browse/SEPASSRFNT-45)

## LEDs
The indicators on tlgate board are configured with the gpio-leds driver, so that userspace doesn't need to mess with `/sys/class/gpio`, and doesn't need to know which gpio a LED is connected to.

We have the following LEDs declared:
- `sys-err`
- `test-voyants`
- `status1`
- `status2`
- `status3`
- `status4`

The names can be changed using the *label* property.

One can drive those through the `/sys/class/leds` directory, for example:
```json
echo 255 > /sys/class/leds/status1/brightness
```

Blinking can be achieved automatically (the kernel takes care of the timing) with:
```json
echo timer > /sys/class/leds/status1/trigger
echo 500 > /sys/class/leds/status1/delay_on
echo 100 > /sys/class/leds/status1/delay_off
```

## GPIOs
In addition to the pinmux in device tree, GPIOs can be declared with names in the `gpio-line-names` array. It allows switching on/off based on names rather than ID. This is typically useful for the libgpio library.
By default, a few tools are installed in the distribution, allowing to trigger GPIOs:

```json
# gpioinfo
gpiochip0 - 24 lines:
        line   0: "GPIO_eMMC_RSTn" unused input active-high
        line   1: "CAN_MUX_SEL" unused input active-high
        line   2: "GPIO_CPSW1_RST" unused input active-high
        line   3: "GPIO_RGMII1_RST" unused input active-high
...
gpiochip2 - 88 lines:
        line   0:      unnamed    "sys-err"  output  active-high [used]
        line   1:      unnamed       unused   input  active-high
        line   2:      unnamed "test-voyants" output active-high [used]
        line   3: "GPIO_EMMC_RSTn" unused input active-high
...
```

One can notice the LEDs also appear in the GPIO list.

Then you can switch GPIO on or off:
```json
gpioset `gpiofind GPIO_EMMC_RSTn`=1
```

Read the help message for gpioset to discover all the options. For example it's able to maintain a GPIO on or off for a given amount of time. **Though, be careful with the fact that
libgpiod uses file descriptors to access gpios and once the file descriptor is closed the GPIO returns to its default case. This is typically the case when gpioset exits.**

[Back](toc.md)
