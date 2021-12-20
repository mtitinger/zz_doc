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

Note: tlgate-app must be adapted, to bind those gpio, using the file API with sysfs (for the leds), or libgpiod (for the reduncency GPIOs). Read the libgpiod section below.

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


## libgpiod
All GPIOs are managed through a library which allows handling names instead of indexes. The benefit is a better hardware abstraction.
Typically, the tlgate application should be revisited to replace `ioctl()` calls to calls to the library, which itself takes care of `/dev/gpiochipX`.

Here is the typical path to control an output GPIO:
```
#include <gpiod.h>

struct gpiod_chip *chip;
struct gpiod_line *line;

chip = gpiod_chip_open_by_name("pcf8574");

/* find OUT3 pin */
line = gpiod_chip_find_line(chip, "OUT3");

/* config as output and set a description */
gpiod_line_request_output(line, "tlgate-app-out3", 0);

/* set value */
gpiod_line_set_value(line, 1);
```

To manage a GPIO as input:
```
#include <gpiod.h>

struct gpiod_chip *chip;
struct gpiod_line *line;

chip = gpiod_chip_open_by_name("pcf8574");

/* find IN2 pin */
line = gpiod_chip_find_line(chip, "IN2");

/* config as input and set a description */
gpiod_line_request_input(line, "tlgate-app-in2");

/* get value */
int value = gpiod_line_get_value(line);
```

If one needs to monitor state changes ("events"), the API functions to use are typically:
```
gpiod_line_request_both_edges_events();
gpiod_line_event_get_fd();
gpiod_line_event_read_fd_multiple();
...
```

The complete documentation for gpiolib can be found here: [http://phwl.org/assets/images/2021/02/libgpiod-ref.pdf]

The main repository for libgpiod is: [https://git.kernel.org/pub/scm/libs/libgpiod/libgpiod.git]

**Important Note**

Once someone calls:
```
gpiod_line_release(line);
gpiod_chip_close(chip);
```
... then the GPIO lines are set back to their initial values. As a consequence, the application that manages GPIOs should keep control for its whole life time.
Using the command-line tools such as `gpioinfo`, `gpiofind` and `gpioset` will result in non-persistent changes.

[Back](toc.md)
