# Riferimento moduli

Ogni modulo è uno script bash eseguito come **root nel chroot** dell'immagine, in ordine alfabetico (vedi [architecture.md](architecture.md#ordine-effettivo-di-esecuzione-target-raspberry)). Struttura comune:

```bash
set -xe; export LC_ALL=C
source /common.sh; install_cleanup_trap   # helper di CustoPiZer
source /files/00-config                   # BASE_USER, DIST_NAME, MOONRAKER_CONFIG, create_user_directories, is_raspbian
```

Legenda colonna **Pin**: se il repo esterno è bloccato a una versione. `HEAD` = si prende sempre l'ultimo commit al momento della build.

## Configurazione condivisa — `modules/generic/files/00-config`

| Variabile / funzione | Valore |
|---|---|
| `BASE_USER` | `pi` |
| `DIST_NAME` | `G1OS` |
| `HOLD_PKGS` | `initramfs-tools`, `initramfs-tools-core` |
| `PRINTER_DATA_DIRS` | `printer_data` + `config comms logs systemd` |
| `MOONRAKER_CONFIG` | `/home/pi/printer_data/config/moonraker.conf` |
| `create_user_directories()` | crea le dir sopra con owner `pi` (solo se mancano) |
| `is_raspbian()` | `1` se esistono `$BOOT_PATH/config.txt` e `/etc/rpi-issue` |

## Moduli generic

| Script | Cosa fa | Repo esterno | Pin |
|---|---|---|---|
| `00-upgrade` | `apt-mark hold HOLD_PKGS`, `apt-get upgrade` (`--force-confnew`), autoremove, aggiunge piwheels a `/etc/pip.conf` | — | — |
| `30-headless-nm` | installa `uuid`; copia `headless_nm.txt.template` e `WiFi-README.txt` in `$BOOT_PATH`; installa `/usr/local/bin/headless_nm` + `headless_nm.service` | — | — |
| `31-wifi-powersave-off` | regola udev `070-wifi-powersave.rules` (`iw <wlan> set power_save off`) | — | — |
| `32-canbus` | abilita `systemd-networkd`, disabilita `-wait-online`, regola udev `tx_queue_len=128` e `25-can.network` (BitRate 1M) | — | — |
| `50-klipper4pallet` | deps (toolchain AVR/ARM, numpy, matplotlib…), gruppi `tty,dialout`, clona **klipper4pellet** in `~/klipper`, venv `~/klippy-env` + `numpy<1.26`, **rimuove `c_helper.so`**, installa `klipper.service` + `klipper.env` | gingeradditive/klipper4pellet | HEAD |
| `51-moonraker` | clona Moonraker, esegue `scripts/install-moonraker.sh -s -z`, copia `moonraker.conf` | Arksine/moonraker | HEAD |
| `52-mainsail` | nginx + sito `mainsail`, logrotate 2 giorni, scarica l'ultima `mainsail.zip`, aggiunge `www-data` al gruppo `pi`, clona `mainsail-config` e linka `mainsail.cfg` | mainsail-crew/mainsail (latest release), mainsail-config | latest / HEAD |
| `53-crowsnest` | clona crowsnest, installa pacchetti da `pkglist-rpi.sh`, `make install` unattended | mainsail-crew/crowsnest | HEAD |
| `54-timelapse` | ffmpeg, clona moonraker-timelapse, symlink componente in Moonraker e `timelapse.cfg` in config; appende sezione **commentata** a `moonraker.conf` | mainsail-crew/moonraker-timelapse | HEAD |
| `55-sonar` | clona sonar (`main`), `make install` unattended | mainsail-crew/sonar | branch main |
| `56-klipperscreen4pellet` | clona **klipperscreen4pellet** in `~/KlipperScreen`, esegue `KlipperScreen-install.sh` (BACKEND=X, SERVICE=Y, START=0, NETWORK=Y), appende `update_manager KlipperScreen` | gingeradditive/klipperscreen4pellet | HEAD |
| `57-Kiauh` | clona kiauh (solo clone, nessuna installazione) | dw-0/kiauh | HEAD |
| `58-Obico` | clona moonraker-obico, `yes "" \| ./install.sh -L -U` | TheSpaghettiDetective/moonraker-obico | HEAD |
| `60-Kamp` | clona KAMP, symlink `config/KAMP`, copia `KAMP_Settings.cfg`, appende update_manager | kyleisah/Klipper-Adaptive-Meshing-Purging | HEAD |
| `60-mainsailos` | crea `/etc/g1os-release`, hostname `g1os`, installa `python3-serial`, `python3-opencv` | — | — |
| `61-EnableUSB` | `pmount`, symlink `gcodes/media → /media`, regola udev `usbstick.rules` + unit `usbstick-handler@.service` (monta in sola lettura) | — | — |
| `62-PowerButton` | clona pi-power-button, esegue `./script/install` (servizio init.d su GPIO3) | Howchoo/pi-power-button | HEAD |
| `69-G1Config` | `python3-flask`, clona **G1-Configs**, esegue `bash ./install.sh` come `pi` | gingeradditive/G1-Configs | HEAD |
| `99-unhold-packages` | **non fa nulla** (riga commentata) | — | — |

## Moduli raspberry

| Script | Cosa fa |
|---|---|
| `10-config-raspberry` | `i2c-tools`; appende `boot-config.txt` a `config.txt`; toglie console seriale da `cmdline.txt`; abilita `i2c-dev`; disabilita `hciuart`, `bluetooth`; (tenta di) aumentare lo swap |
| `11-fix-wifi` | `rfkill unblock wifi`, `WirelessEnabled=true` nello stato NetworkManager |
| `61-postrename` | copia `files/postrename` in `/postrename` e lo aggiunge a `/etc/rc.local` (eseguito a ogni boot finché non si auto-rimuove) |
| `63-SplashScreen` | `fbi`; `disable_splash=1` in config.txt; parametri quiet in cmdline; crea e abilita `splashscreen.service` che mostra `~/printer_data/config/splash.png` |
| `98-remove-passwordless-sudo` | commenta la riga di `pi` in `/etc/sudoers.d/010_pi-nopasswd` |

### `postrename` (runtime, primo boot)

Se l'utente UID 1000 **non** è `pi` (rinominato da Raspberry Pi Imager), lo script:

1. ferma `moonraker klipper nginx sonar crowsnest` (+ `KlipperScreen` se trovato),
2. sostituisce `/home/pi/mainsail` nel sito nginx,
3. `sed s/pi/<utente>/g` su unit file dei servizi e su `printer_data/systemd/*.env`,
4. corregge gli shebang dei venv `klippy-env`, `moonraker-env`, `.KlipperScreen-env`,
5. corregge regole polkit di Moonraker/KlipperScreen, log path e logrotate di crowsnest,
6. ricrea i symlink di crowsnest, sonar, timelapse, mainsail.cfg, kiauh,
7. `daemon-reload`, riavvia i servizi, attende 30 s, si cancella da `/` e da `rc.local`, **reboot**.

Tutto ciò che **non** è in questa lista resta con path `/home/pi` (vedi [known-issues.md](known-issues.md)).

### `headless_nm` (runtime, ogni boot)

Se trova `headless_nm.txt` sotto `/boot` (l'utente lo crea copiando il template sulla partizione FAT):
legge `SSID`, `PASSWORD`, `HIDDEN`, `REGDOMAIN` → scrive `/etc/NetworkManager/system-connections/preconfigured.nmconnection` (PSK calcolata con `wpa_passphrase`), imposta il regdomain in `cmdline.txt`, `chmod 600`, `nmcli connection reload`, **cancella il file di setup**. Log in journal con tag `headless_nm`.

## File statici rilevanti (`modules/generic/files/`)

| File | Destinazione |
|---|---|
| `klipper.service` | `/etc/systemd/system/klipper.service` (User=pi, path hard-coded) |
| `klipper.env` | `~/printer_data/systemd/klipper.env` (path hard-coded `/home/pi`) |
| `moonraker.conf` | `~/printer_data/config/moonraker.conf` (+ snippet appesi da timelapse, KlipperScreen, KAMP) |
| `mainsail-nginx/*` | `/etc/nginx/sites-available/mainsail`, `/etc/nginx/conf.d/{common_vars,upstreams}.conf` |
| `070-wifi-powersave.rules` | `/etc/udev/rules.d/` |
| `canbus/10-can.rules`, `canbus/25-can.network` | `/etc/udev/rules.d/`, `/etc/systemd/network/` |
| `headless-nm/*` | `/usr/local/bin/headless_nm`, unit, template + README in `/boot/firmware` |

## Moduli non usati

- `modules/armbian/*`, `modules/special/*`: ereditati da MainsailOS per Orange Pi/Armbian; disattivati in `config.yml`. Non hanno le personalizzazioni G1 (splash, postrename…).
- `patches/*`: script per MainsailOS 1.x/Bullseye. Gli URL puntano a `mainsail-crew/G1OS` (inesistente). Non usare.
