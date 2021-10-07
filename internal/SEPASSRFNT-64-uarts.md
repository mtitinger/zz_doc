# Configuration for quad-uarts chips

## Kernel configuration
We have 4 MAX14830 chips on the board, allowing to drive 16 UARTs total. The devices appear as `/dev/ttyMAX<n>` in the OS. Each chip is on a dedicated SPI bus so that the SPI bandwidth doesn't need to be shared.

Each chip can generate interruptions, which are hooked to the CPU. Interrupts are labeled based on the corresponding SPI bus. They can be traced if you know the interrupt number, which is displayed at driver init time:
```
# dmesg | grep ttyMAX
[    2.499473] spi1.0: ttyMAX0 at I/O 0x0 (irq = 372, base_baud = 2764800) is a MAX14830
[    2.507885] spi1.0: ttyMAX1 at I/O 0x20 (irq = 372, base_baud = 2764800) is a MAX14830
[    2.516235] spi1.0: ttyMAX2 at I/O 0x40 (irq = 372, base_baud = 2764800) is a MAX14830
[    2.524547] spi1.0: ttyMAX3 at I/O 0x60 (irq = 372, base_baud = 2764800) is a MAX14830
```

So interrupt number is 372 (for this MAX14830 at least).

Then you can watch interrupts counter:
```
# cat /proc/interrupts
          CPU0       CPU1
 11:       4080       3899     GICv3  30 Level     arch_timer
...
372:          0          0      GPIO  38 Edge    -davinci_gpio  spi1.0
...
```

Each chip is clocked with an external crystal, which oscillation frequency is 3.686400 MHz. This is configured in the device tree, in the `spi_uart_clk` node.

Also, for each SPI bus we configure the maximum SPI bus frequency. As of today it is set to 20MHz with the `spi-max-frequency` property.

## Userspace

In order to configure the serial lines in RS485 mode, you need to make ioctl calls. This is not possible to configure in the device tree (with the traditional `linux,rs485-enabled-at-boot-time` property) because the driver doesn't manage this.

A typical code sequence to switch to RS485 would be:
```
/* RS485 ioctls: */
#define TIOCGRS485      0x542E
#define TIOCSRS485      0x542F

int fd = open(port, O_RDWR);

struct serial_rs485 rs485conf;

if (ioctl (fd, TIOCGRS485, &rs485conf) < 0) {
	fprintf( stderr, "Error reading ioctl port (%d): %s\n",  errno, strerror( errno ));
}

printf("Port currently RS485 mode is %s\n", (rs485conf.flags & SER_RS485_ENABLED) ? "set" : "NOT set");

rs485conf.flags |= SER_RS485_ENABLED | SER_RS485_RTS_ON_SEND;

if (ioctl (fd, TIOCSRS485, &rs485conf) < 0) {
	fprintf( stderr, "Error sending ioctl port (%d): %s\n",  errno, strerror( errno ));
}
```
[Back](toc.md)
