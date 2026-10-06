<!-- THIS FILE IS UPDATED AUTOMATICALLY, ANY CHANGES WILL BE OVERRIDDEN -->
# Changelog
All notable changes to G1OS will be documented in this file.

## [3.0.1](https://github.com/gingeradditive/G1OS/releases/tag/3.0.1) - 2026-10-06
### Bug Fixes and Improvements

- **cloudinit**: Keep build-time groups of pi when user-data sets groups

### Other

- Bump version to v3.0.1

## [3.0.0](https://github.com/gingeradditive/G1OS/releases/tag/3.0.0) - 2026-10-05
### Features

- **cloudinit**: Add cloud-init init_format with pre/post-rename flow
- **crowsnest**: Switch to v5 branch and drop legacy pkglist installer

### Bug Fixes and Improvements

- **headless-nm**: Validate config before replacing the connection
- **klipper**: Adapt numpy and libatlas-base-dev for Trixie
- **klipper**: Remove corrupt c_helper.so before start
- **moonraker**: Point KlipperScreen update_manager to klipperscreen4pellet
- **moonraker**: Prevent empty moonraker.asvc blocking service restarts
- **moonraker,kamp**: Create config files as BASE_USER
- **opi-zero2**: Update overlay configuration for SPI, I2C, and UART ([#355](https://github.com/gingeradditive/G1OS/pull/355))
- **opi3lts**: Update configuration to enable i2c and UART alongside SPI ([#357](https://github.com/gingeradditive/G1OS/pull/357))
- **opi4lts**: Update overlay_prefix for RK3399 and copy fixup script ([#356](https://github.com/gingeradditive/G1OS/pull/356))
- **raspberry**: Configure user, ssh, getty and passwordless sudo for Trixie
- **udev**: Add missing commas in wifi powersave and CAN rules
- **usb**: Mount sticks without partition table
- Use BOOT_PATH in splashscreen and unhold packages after build
- Fix building wheels on armhf images ([#358](https://github.com/gingeradditive/G1OS/pull/358))

### Refactor

- Remove commented code and unused file ([#364](https://github.com/gingeradditive/G1OS/pull/364))

### Documentation

- **known-issues**: Mark B01 unpinned external repos as intended
- **known-issues**: Document B16 default credentials decision
- **known-issues**: Mark B17 unpinned base image as intended
- **known-issues**: Add B18 static findings and Pi 4 test checklist
- **known-issues**: Correct B03 analysis and add field report issues B19-B24
- Update for Trixie, cloud-init and upstream sync

### Other

- **build**: Update runner to ubuntu-24.04-arm
- **workflow**: Update build & release workflow actions ([#362](https://github.com/gingeradditive/G1OS/pull/362))
- Update .gitignore to include JetBrains and Clankers ([#361](https://github.com/gingeradditive/G1OS/pull/361))
- Remove MainsailOS leftovers
- Download base image via HTTPS instead of torrent
- Bump version to v3.0.0

## [2.1.0](https://github.com/gingeradditive/G1OS/releases/tag/2.1.0) - 2026-09-09
### Bug Fixes and Improvements

- Remove pre-built chelper binary to prevent crashes

### Other

- Bump version to v2.1.0

## [2.0.10](https://github.com/gingeradditive/G1OS/releases/tag/2.0.10) - 2026-05-20
### Other

- Bump version to v2.0.10

## [2.0.9](https://github.com/gingeradditive/G1OS/releases/tag/2.0.9) - 2026-04-27
### Other

- Bump version to v2.0.9

## [2.0.8](https://github.com/gingeradditive/G1OS/releases/tag/2.0.8) - 2026-04-15
### Other

- Bump version to v2.0.8

## [2.0.7](https://github.com/gingeradditive/G1OS/releases/tag/2.0.7) - 2026-02-18
### Other

- Bump version to v2.0.7

## [2.0.6](https://github.com/gingeradditive/G1OS/releases/tag/2.0.6) - 2026-02-16
### Features

- **build**: Removed raspicam and serialcomm module ([#83](https://github.com/gingeradditive/G1OS/pull/83))
- **ci**: Add KlipperScreen module ([#284](https://github.com/gingeradditive/G1OS/pull/284))
- **moonraker.conf**: Add mainsail subscription to announcements ([#115](https://github.com/gingeradditive/G1OS/pull/115))
- Allow the app.fluidd.xyz origin
- Add multi mjpegstreamer support in klipper_config dir ([#35](https://github.com/gingeradditive/G1OS/pull/35))
- Add park to CANCEL_PRINT ([#58](https://github.com/gingeradditive/G1OS/pull/58))
- Replaced module busterpatch ([#77](https://github.com/gingeradditive/G1OS/pull/77))
- Added .editorconfig ([#78](https://github.com/gingeradditive/G1OS/pull/78))
- Changed logging and logrotate behavior ([#79](https://github.com/gingeradditive/G1OS/pull/79))
- Added mainsailos module ([#81](https://github.com/gingeradditive/G1OS/pull/81))
- Stop part fan on CANCEL_PRINT ([#103](https://github.com/gingeradditive/G1OS/pull/103))
- Add on_error_gcode to mainsail.cfg ([#116](https://github.com/gingeradditive/G1OS/pull/116))
- Add sonar by default to image ([#107](https://github.com/gingeradditive/G1OS/pull/107))
- Add on_error_gcode to mainsail.cfg ([#116](https://github.com/gingeradditive/G1OS/pull/116))
- Add python3-serial CanBoot dependency ([#129](https://github.com/gingeradditive/G1OS/pull/129))
- Add postrename module ([#128](https://github.com/gingeradditive/G1OS/pull/128))
- Add timelapse module ([#130](https://github.com/gingeradditive/G1OS/pull/130))
- Enable I2C by default ([#196](https://github.com/gingeradditive/G1OS/pull/196))
- Add Orange Pi 3 and 4 LTS ([#186](https://github.com/gingeradditive/G1OS/pull/186))
- Add orange pi zero2 ([#189](https://github.com/gingeradditive/G1OS/pull/189))
- Add BananaPi M2 Zero ([#247](https://github.com/gingeradditive/G1OS/pull/247))
- Adds kiauh module for homebrewed images ([#184](https://github.com/gingeradditive/G1OS/pull/184))
- Add python3-opencv for obico ([#248](https://github.com/gingeradditive/G1OS/pull/248))
- Add module for KIAUH ([#318](https://github.com/gingeradditive/G1OS/pull/318))
- Add CANBUS support ([#339](https://github.com/gingeradditive/G1OS/pull/339))
- Add btt cb1 build_only image ([#328](https://github.com/gingeradditive/G1OS/pull/328))
- Add support for orangepi zero3 ([#336](https://github.com/gingeradditive/G1OS/pull/336))
- Reimplement g1os customization

### Bug Fixes and Improvements

- **Canbus**: Remove `RestartSec=0.1s` from the 25-can.network config. ([#349](https://github.com/gingeradditive/G1OS/pull/349))
- **RpiOS**: Fix download urls to raspios oldstable (bookworm) ([#350](https://github.com/gingeradditive/G1OS/pull/350))
- **build**: Updated download paths ([#65](https://github.com/gingeradditive/G1OS/pull/65))
- **build**: Fixes error in Makefile ([#76](https://github.com/gingeradditive/G1OS/pull/76))
- **build**: Updated torrent download url ([#90](https://github.com/gingeradditive/G1OS/pull/90))
- **build**: Fix mv of image file ([#204](https://github.com/gingeradditive/G1OS/pull/204))
- **build**: Fix location of boot partition ([#289](https://github.com/gingeradditive/G1OS/pull/289))
- **build**: Fix log file upload ([#297](https://github.com/gingeradditive/G1OS/pull/297))
- **ci**: Fix annotations from actions ([#285](https://github.com/gingeradditive/G1OS/pull/285))
- **config.txt**: Fix configuration errors with attached screens ([#119](https://github.com/gingeradditive/G1OS/pull/119))
- **crowsnest**: Fix install of crowsnest ([#111](https://github.com/gingeradditive/G1OS/pull/111))
- **headless_nm**: Fix reading SSID or PW with special chars ([#347](https://github.com/gingeradditive/G1OS/pull/347))
- **lint**: Should fix shellcheck warnings ([#160](https://github.com/gingeradditive/G1OS/pull/160))
- **mainsail**: Changed download url to mainsail-crew url ([#92](https://github.com/gingeradditive/G1OS/pull/92))
- **moonraker**: PKGLIST variable rename ([#298](https://github.com/gingeradditive/G1OS/pull/298))
- **sonar**: Fixes missing moonraker update manager entry ([#112](https://github.com/gingeradditive/G1OS/pull/112))
- **ustreamer**: Disable buffering on webcam proxy entries ([#84](https://github.com/gingeradditive/G1OS/pull/84))
- Nginx config file
- Added http1.1 to moonraker api reverse proxy location ([#75](https://github.com/gingeradditive/G1OS/pull/75))
- Correct on_error_gcode in mainsail.cfg ([#118](https://github.com/gingeradditive/G1OS/pull/118))
- Correct on_error_gcode in mainsail.cfg ([#118](https://github.com/gingeradditive/G1OS/pull/118))
- Fix errors in moved venvs ([#138](https://github.com/gingeradditive/G1OS/pull/138))
- Fixes error unusable wpa_supplicant.txt ([#142](https://github.com/gingeradditive/G1OS/pull/142))
- Fix postrename script ([#150](https://github.com/gingeradditive/G1OS/pull/150))
- Fix shellcheck errors in net module ([#161](https://github.com/gingeradditive/G1OS/pull/161))
- Set wrong source path ([#164](https://github.com/gingeradditive/G1OS/pull/164))
- Add otg_mode=1 for CM4 in config.txt ([#167](https://github.com/gingeradditive/G1OS/pull/167))
- Fix SC2086 in armbian module ([#173](https://github.com/gingeradditive/G1OS/pull/173))
- Fixes error setting link to macro ([#175](https://github.com/gingeradditive/G1OS/pull/175))
- Fix shellcheck errors ([#185](https://github.com/gingeradditive/G1OS/pull/185))
- Fix syntax error in net module ([#191](https://github.com/gingeradditive/G1OS/pull/191))
- Fix compress step ([#205](https://github.com/gingeradditive/G1OS/pull/205))
- Fix rpi-image.json workflow in Release.yml ([#206](https://github.com/gingeradditive/G1OS/pull/206))
- Remove enduser support msg from zero2 images ([#209](https://github.com/gingeradditive/G1OS/pull/209))
- Fix rpi-imager json value format for extract_size & image_download_size ([#212](https://github.com/gingeradditive/G1OS/pull/212))
- Fix firstboot issue ([#214](https://github.com/gingeradditive/G1OS/pull/214))
- Load `i2c-dev` modules ([#217](https://github.com/gingeradditive/G1OS/pull/217))
- Fix broken udev package ([#224](https://github.com/gingeradditive/G1OS/pull/224))
- Fix udev for version 'rp1+deb11u2' ([#226](https://github.com/gingeradditive/G1OS/pull/226))
- Remove legacy cam stack ([#227](https://github.com/gingeradditive/G1OS/pull/227))
- Fix error in udev-fix.sh ([#228](https://github.com/gingeradditive/G1OS/pull/228))
- Add crowsnest log path & pkglist link in postrename script ([#235](https://github.com/gingeradditive/G1OS/pull/235))
- Fix typo in tools_dir var, Line171 ([#237](https://github.com/gingeradditive/G1OS/pull/237))
- Fix wifi connectivity ([#240](https://github.com/gingeradditive/G1OS/pull/240))
- Fix armbian-release file error ([#241](https://github.com/gingeradditive/G1OS/pull/241))
- Fix error autologin on serial tty ([#242](https://github.com/gingeradditive/G1OS/pull/242))
- Fix 'is_board_type' function ([#243](https://github.com/gingeradditive/G1OS/pull/243))
- Fix Torrent and Checksum download URLs ([#244](https://github.com/gingeradditive/G1OS/pull/244))
- Removing 'klippy' alias ([#246](https://github.com/gingeradditive/G1OS/pull/246))
- Fix motd on armbian and orange pi based images ([#272](https://github.com/gingeradditive/G1OS/pull/272))
- Fix version number in release ([#275](https://github.com/gingeradditive/G1OS/pull/275))
- Fix version number in release workflow ([#276](https://github.com/gingeradditive/G1OS/pull/276))
- Add supported sbc to non rpi images ([#277](https://github.com/gingeradditive/G1OS/pull/277))
- Fix current version number ([#317](https://github.com/gingeradditive/G1OS/pull/317))
- Improve special modules check in build & release workflow ([#329](https://github.com/gingeradditive/G1OS/pull/329))
- Enable WiFi per default on RPI SBCs ([#334](https://github.com/gingeradditive/G1OS/pull/334))
- Fix parsing array to copy special modules ([#338](https://github.com/gingeradditive/G1OS/pull/338))
- Fix upgrade rpi images ([#342](https://github.com/gingeradditive/G1OS/pull/342))
- Fix armbian-motd ([#352](https://github.com/gingeradditive/G1OS/pull/352))
- Armbian-motd ([#353](https://github.com/gingeradditive/G1OS/pull/353))
- Hold linux kernel updates
- Remove klipper

### Refactor

- **klipper**: Refactor klipper and is-pre-install module ([#113](https://github.com/gingeradditive/G1OS/pull/113))
- **mainsail.cfg**: Substituting `/home/pi` with `~` ([#114](https://github.com/gingeradditive/G1OS/pull/114))
- **mainsail.cfg**: Substituting `/home/pi` with `~` ([#114](https://github.com/gingeradditive/G1OS/pull/114))
- Updated input shaper dependencies to python3 ([#74](https://github.com/gingeradditive/G1OS/pull/74))
- Add `enable_auto_refresh: True` ([#133](https://github.com/gingeradditive/G1OS/pull/133))
- Deactivate IPv6 in nginx per default ([#157](https://github.com/gingeradditive/G1OS/pull/157))
- Change behavior of piconfig module ([#180](https://github.com/gingeradditive/G1OS/pull/180))
- Use mv to move the image from the workspace to the root ([#203](https://github.com/gingeradditive/G1OS/pull/203))
- Drop armbian_pkgupgrade ([#210](https://github.com/gingeradditive/G1OS/pull/210))
- Change version handling ([#296](https://github.com/gingeradditive/G1OS/pull/296))
- Transition from CustomPiOs to CustoPiZer ([#314](https://github.com/gingeradditive/G1OS/pull/314))
- Use systemctl_if_exists in 10-config-raspberry ([#332](https://github.com/gingeradditive/G1OS/pull/332))

### Documentation

- Improve reamde.md ([#54](https://github.com/gingeradditive/G1OS/pull/54))
- Fix typo in readme.md ([#91](https://github.com/gingeradditive/G1OS/pull/91))
- Correct screenshot image URL ([#93](https://github.com/gingeradditive/G1OS/pull/93))
- Add mainsailos logo ([#124](https://github.com/gingeradditive/G1OS/pull/124))
- Update README for improved readability ([#144](https://github.com/gingeradditive/G1OS/pull/144))
- Adds faq section
- Fix broken README link to the docs ([#231](https://github.com/gingeradditive/G1OS/pull/231))
- Readme CanBoot -> katapult ([#320](https://github.com/gingeradditive/G1OS/pull/320))
- Readme fix newline ([#321](https://github.com/gingeradditive/G1OS/pull/321))
- Update README.md ([#327](https://github.com/gingeradditive/G1OS/pull/327))

### Other

- **README**: Update README according to latest changes. ([#110](https://github.com/gingeradditive/G1OS/pull/110))
- **build**: 0.5.0 bump
- **build**: Refactor build dependend files ([#154](https://github.com/gingeradditive/G1OS/pull/154))
- **build**: Add new matrix workflow ([#253](https://github.com/gingeradditive/G1OS/pull/253))
- **ci**: Skip build on push to master branch ([#280](https://github.com/gingeradditive/G1OS/pull/280))
- **crowsnest**: Update crowsnest module ([#123](https://github.com/gingeradditive/G1OS/pull/123))
- **docs**: Fix urls, add includes ([#122](https://github.com/gingeradditive/G1OS/pull/122))
- **klipper**: Update klipper and input shaper to py3 ([#105](https://github.com/gingeradditive/G1OS/pull/105))
- **moonraker**: Refactored moonraker module ([#89](https://github.com/gingeradditive/G1OS/pull/89))
- **release**: Skip the ftp upload, if repo != Mainsail-Crew ([#269](https://github.com/gingeradditive/G1OS/pull/269))
- **release**: Generate changelog from 0.0.0, if repo != Mainsail-Crew ([#270](https://github.com/gingeradditive/G1OS/pull/270))
- **release**: Update publish release action ([#268](https://github.com/gingeradditive/G1OS/pull/268))
- **release**: Fix changelog workflow ([#271](https://github.com/gingeradditive/G1OS/pull/271))
- **workflow**: Rework release workflow for multi builds ([#181](https://github.com/gingeradditive/G1OS/pull/181))
- **workflows**: Update pull_request trigger
- Moves $httpupgrade and upstream servers to common config
- Update default moonraker.conf
- Remove job_queue and postprocessing from moonraker.conf
- Updated .editorconfig for yml files ([#86](https://github.com/gingeradditive/G1OS/pull/86))
- Push versionnumber to 0.6.1
- Update moonraker.conf ([#101](https://github.com/gingeradditive/G1OS/pull/101))
- Add funding informations ([#120](https://github.com/gingeradditive/G1OS/pull/120))
- Add Issue Templates ([#121](https://github.com/gingeradditive/G1OS/pull/121))
- Bump version to 0.7.0 ([#131](https://github.com/gingeradditive/G1OS/pull/131))
- Add wireless-tools as moonraker dependency ([#137](https://github.com/gingeradditive/G1OS/pull/137))
- Add workflow to close stale issues / pull requests ([#139](https://github.com/gingeradditive/G1OS/pull/139))
- Add additional Input shaper dependencies ([#140](https://github.com/gingeradditive/G1OS/pull/140))
- Bump version to v0.7.1 ([#145](https://github.com/gingeradditive/G1OS/pull/145))
- Rework build workflow with source image cache ([#146](https://github.com/gingeradditive/G1OS/pull/146))
- Only build an image on push in master/develop branch ([#148](https://github.com/gingeradditive/G1OS/pull/148))
- Change cron interval of stale action ([#149](https://github.com/gingeradditive/G1OS/pull/149))
- Update versions according to bullseye ([#147](https://github.com/gingeradditive/G1OS/pull/147))
- Rework build workflow for multiple images ([#152](https://github.com/gingeradditive/G1OS/pull/152))
- Fix image name ([#153](https://github.com/gingeradditive/G1OS/pull/153))
- Impove shellcheck and auto read version number ([#155](https://github.com/gingeradditive/G1OS/pull/155))
- Add Raspberry 64bit config ([#156](https://github.com/gingeradditive/G1OS/pull/156))
- Upload failed logfile ([#163](https://github.com/gingeradditive/G1OS/pull/163))
- Add armbian module ([#165](https://github.com/gingeradditive/G1OS/pull/165))
- Update modules according to path changes ([#166](https://github.com/gingeradditive/G1OS/pull/166))
- Update BuildImages workflow ([#171](https://github.com/gingeradditive/G1OS/pull/171))
- Improved shellcheck lint ([#172](https://github.com/gingeradditive/G1OS/pull/172))
- Remove github-token for build action ([#178](https://github.com/gingeradditive/G1OS/pull/178))
- Add "not-on-Github" bot for issues ([#179](https://github.com/gingeradditive/G1OS/pull/179))
- Fix changelog in release workflow ([#182](https://github.com/gingeradditive/G1OS/pull/182))
- Removes fkms overlays ([#183](https://github.com/gingeradditive/G1OS/pull/183))
- Remove unattended-upgrades service ([#215](https://github.com/gingeradditive/G1OS/pull/215))
- Revert firstboot fix ([#219](https://github.com/gingeradditive/G1OS/pull/219))
- Update crowsnest module ([#221](https://github.com/gingeradditive/G1OS/pull/221))
- Update download urls for armbian & orangepi ([#233](https://github.com/gingeradditive/G1OS/pull/233))
- Update .gitignore ([#254](https://github.com/gingeradditive/G1OS/pull/254))
- Refactor build & release workflow to remove external actions ([#264](https://github.com/gingeradditive/G1OS/pull/264))
- Update issue bot text ([#265](https://github.com/gingeradditive/G1OS/pull/265))
- Update generate json to new format ([#266](https://github.com/gingeradditive/G1OS/pull/266))
- Fix typo in bot text ([#273](https://github.com/gingeradditive/G1OS/pull/273))
- Update urls and base image version ([#288](https://github.com/gingeradditive/G1OS/pull/288))
- Add piwheels config, if it doesnt exists ([#290](https://github.com/gingeradditive/G1OS/pull/290))
- Change module order ([#291](https://github.com/gingeradditive/G1OS/pull/291))
- Switch to armbian for Opi Zero2 ([#292](https://github.com/gingeradditive/G1OS/pull/292))
- Bump version to v2.0.0
- Add attribute to config.yml to add "build_only" images ([#325](https://github.com/gingeradditive/G1OS/pull/325))
- Bump version to v2.1.0
- Bump version to v2.2.0
- Bump version to v2.2.1
- Bump version to v2.2.2
- Bump version to v2.0.3
- Disable orange pi and renable update fixed package
- Bump version to v2.0.5
- Disable raspberry pi armhf build
- Bump version to v2.0.6


