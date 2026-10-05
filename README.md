<p align="center">
<img src=".github/sdcard-logo.png" style="width:40%; max-width: 300px;" >
</p>

# G1OS

A [Raspberry Pi OS](https://www.raspberrypi.com/software/) based distribution designed specifically for the **Ginger G1** 3D Printer. It includes all the necessary software and optimizations to get started with **Klipper Firmware** and **Mainsail** for 3D printing with **pellet extrusion** technology.

G1OS supports the **Raspberry Pi 4** only.

## What's included?

G1OS comes with the following pre-installed and configured software:

-   [Klipper4Pellet (Customized Klipper Firmware for Pellet 3D Printing)](https://github.com/gingeradditive/klipper4pellet)
-   [KlipperScreen4Pellet (Touchscreen UI for the Ginger G1)](https://github.com/gingeradditive/klipperscreen4pellet)
-   [G1-Configs (Ginger G1 printer configuration)](https://github.com/gingeradditive/G1-Configs)
-   [Moonraker (API for Klipper)](https://github.com/Arksine/moonraker)
-   [Mainsail (Klipper Web Interface)](https://github.com/mainsail-crew/mainsail)
-   [Crowsnest (Webcam Streaming)](https://github.com/mainsail-crew/crowsnest)
-   [Sonar (Keepalive Daemon)](https://github.com/mainsail-crew/sonar)
-   [Nginx (Web Server & Proxy)](https://nginx.org/en/)

## G1OS also includes:

-   **Preconfigured Serial Connection** for the Ginger G1 using Hardware UART (PL011).
-   **Preinstalled Dependencies** for Klipper's Input Shaper. Simply build the [klipper_mcu](https://www.klipper3d.org/RPi_microcontroller.html) and install the service. See [Klipper documentation](https://www.klipper3d.org/Measuring_Resonances.html) for more info.
-   **Preinstalled Python3-serial package**, required for [CanBoot](https://github.com/Arksine/CanBoot).
-   **USB stick automount**: G-code files on a USB stick show up in the `media` folder of Mainsail and KlipperScreen.

## Default user

G1OS supports only the `pi` user. If you set a different username in Raspberry Pi Imager, the password, SSH keys and other settings you chose are applied to the `pi` user.

## FAQ

**Q:** How do I report a bug?  
**A:** Please ensure it's not a configuration issue with:

-   Klipper
-   Moonraker
-   Crowsnest
-   Sonar

If the issue is specific to the **G1OS** setup or the Ginger G1 printer, report it via the [G1OS GitHub Issues](https://github.com/gingeradditive/G1OS/issues). Provide detailed information to help us resolve the problem quickly.

**Q:** What is the philosophy behind G1OS?  
**A:** We maintain a **KISS** principle—Keep It Simple and Straightforward. G1OS is built on the same foundation as **MainsailOS**, but optimized for pellet printing with the Ginger G1. We aim to keep things as close to the Raspberry Pi OS and MainsailOS documentation as possible, providing extra documentation only where needed.

**Q:** How can I contribute?  
**A:** Contributions are always welcome! Please check out [CONTRIBUTING.md](CONTRIBUTING.md).

## Build your own / Development

To build your own version of **G1OS**, simply fork this repository, enable workflows, and each push will trigger an automated image build (`.github/workflows/build.yml`).

Images are built with [CustoPiZer](https://github.com/OctoPrint/CustoPiZer): the bash modules in `modules/` run in a chroot of the Raspberry Pi OS base image. See [docs/](docs/README.md) for the build pipeline, the module reference and debugging notes.

## Credits

G1OS is a **fork of [MainsailOS](https://github.com/mainsail-crew/MainsailOS)**, tailored for the unique requirements of pellet-based 3D printing with the Ginger G1. Many thanks to the [Mainsail Crew](https://github.com/mainsail-crew) for MainsailOS, Mainsail, Crowsnest and Sonar: if you find their work useful, consider [supporting them](https://docs.mainsail.xyz/credits).
