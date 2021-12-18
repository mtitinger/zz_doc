# HMI GPIOs and LEDs

## Feature Status and References

| Technical Notes and Specification | Current [Maturity Grade](../01_developement_methods/SEPASSRFNT-96-developement.md#Maturity Grades)| Comments |
| :---: | :---: | --- |
|[SEPASSRFNT-45](https://jira.open-groupe.com/browse/SEPASSRFNT-45) | MG40 (18/12/21) | development complete for BSP part and EMC testing, but tlgate-app must still be cabled to using those GPIOs |

## Feature Description

### The LEDS related to the HMI

* In an early stage, until the application tlgate starts, leds are setup using an init script to make then blink, so that in case of misconfiguration or failure of tlgate blinking remains.

* once tlgate has started, the leds are managed by the application.

### The GPIO Expander, that is used to manage redundency

For testing, the GPIO Chip can be identified with:

```
root@am64xx-tlgate:/sys/class/gpio/gpiochip329# gpiodetect
gpiochip0 [600000.gpio] (87 lines)
gpiochip1 [601000.gpio] (88 lines)
gpiochip2 [pcf8574] (8 lines)
gpiochip3 [MAX14830] (16 lines)
gpiochip4 [MAX14830] (16 lines)
gpiochip5 [MAX14830] (16 lines)
gpiochip6 [MAX14830] (16 lines)
```

Note: tlgate-app must be adapted, to bind those gpio, using the file API with sysfs (for the leds), or libgpio (for the reduncency GPIOs).

* specified in SEPASSRFNT-79
* coded in SEPASSRFNT-80

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
```

Assigning to the 'panic' state:
```json
echo panic > /sys/class/leds/sys-err/trigger
```

## Testing

BSP level tests are available from the BSP test-suite, or from the EMC test-suite, see related documentation.

[Back](toc.md)
