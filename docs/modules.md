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
| `BASE_PASSWORD` | `raspberry` (password di default dell'utente, impostata in build) |
| `PRINTER_DATA_DIRS` | `printer_data` + `config comms logs systemd` |
| `MOONRAKER_CONFIG` | `/home/pi/printer_data/config/moonraker.conf` |
| `create_user_directories()` | crea le dir sopra con owner `pi` (solo se mancano) |
| `is_raspbian()` | `1` se esistono `$BOOT_PATH/config.txt` e `/etc/rpi-issue` |

## Moduli generic

| Script | Cosa fa | Repo esterno | Pin |
|---|---|---|---|
| `00-upgrade` | `apt-get upgrade` (`--force-confnew`), autoremove, aggiunge piwheels a `/etc/pip.conf` | — | — |
| `30-headless-nm` | installa `uuid`; copia `headless_nm.txt.template` e `WiFi-README.txt` in `$BOOT_PATH`; installa `/usr/local/bin/headless_nm` + `headless_nm.service` | — | — |
| `31-wifi-powersave-off` | regola udev `070-wifi-powersave.rules` (`iw <wlan> set power_save off`) | — | — |
| `32-canbus` | abilita `systemd-networkd`, disabilita `-wait-online`, regola udev `tx_queue_len=128` e `25-can.network` (BitRate 1M) | — | — |
| `50-klipper4pallet` | deps (toolchain AVR/ARM, numpy, matplotlib, `libatlas-base-dev` solo se esiste in apt), gruppi `tty,dialout`, clona **klipper4pellet** in `~/klipper`, venv `~/klippy-env` + `numpy` (senza pin: `numpy<1.26` non supporta Python 3.13), **rimuove `c_helper.so`**, installa `klipper.service` + `klipper.env` | gingeradditive/klipper4pellet | HEAD |
| `51-moonraker` | clona Moonraker, (solo armhf: pre-installa pillow nel venv), esegue `scripts/install-moonraker.sh -s -z`, copia `moonraker.conf` | Arksine/moonraker | HEAD |
| `52-mainsail` | nginx + sito `mainsail`, logrotate 2 giorni, scarica l'ultima `mainsail.zip`, aggiunge `www-data` al gruppo `pi`, clona `mainsail-config` e linka `mainsail.cfg` | mainsail-crew/mainsail (latest release), mainsail-config | latest / HEAD |
| `53-crowsnest` | clona crowsnest (branch di default = `v5`), `make install` unattended: le dipendenze le installa l'installer di crowsnest | mainsail-crew/crowsnest | HEAD (v5) |
| `54-timelapse` | ffmpeg, clona moonraker-timelapse, symlink componente in Moonraker e `timelapse.cfg` in config; appende sezione **commentata** a `moonraker.conf` | mainsail-crew/moonraker-timelapse | HEAD |
| `55-sonar` | clona sonar (`main`), `make install` unattended | mainsail-crew/sonar | branch main |
| `56-klipperscreen4pellet` | clona **klipperscreen4pellet** in `~/KlipperScreen`, esegue `KlipperScreen-install.sh` (BACKEND=X, SERVICE=Y, START=0, NETWORK=Y), appende `update_manager KlipperScreen` | gingeradditive/klipperscreen4pellet | HEAD |
| `57-Kiauh` | clona kiauh (solo clone, nessuna installazione) | dw-0/kiauh | HEAD |
| `58-Obico` | clona moonraker-obico, `yes "" \| ./install.sh -L -U` | TheSpaghettiDetective/moonraker-obico | HEAD |
| `60-Kamp` | clona KAMP, symlink `config/KAMP`, copia `KAMP_Settings.cfg`, appende update_manager | kyleisah/Klipper-Adaptive-Meshing-Purging | HEAD |
| `60-mainsailos` | crea `/etc/g1os-release`, hostname `g1os`, installa `python3-serial`, `python3-opencv` | — | — |
| `61-postrename-cloudinit` | solo se `EDITBASE_INIT_FORMAT` è `cloudinit*`: installa `cloud-init`, `python3-yaml`, gli script `mainsailos-prerename`/`-postrename` e la lib condivisa, abilita le due unit | — | — |
| `61-EnableUSB` | `pmount`, symlink `gcodes/media → /media`, regola udev `usbstick.rules` + unit `usbstick-handler@.service` (monta in sola lettura) | — | — |
| `62-PowerButton` | clona pi-power-button, esegue `./script/install` (servizio init.d su GPIO3) | Howchoo/pi-power-button | HEAD |
| `69-G1Config` | `python3-flask`, clona **G1-Configs**, esegue `bash ./install.sh` come `pi` (copia `Configs/*` in `printer_data/config`, temi Mainsail/KlipperScreen, crea `g1-flask.service`, ripristina il DB di Moonraker, copia i gcode di fabbrica) | gingeradditive/G1-Configs | HEAD |

## Moduli raspberry

| Script | Cosa fa |
|---|---|
| `10-config-raspberry` | `i2c-tools`; appende `boot-config.txt` a `config.txt`; toglie console seriale da `cmdline.txt`; abilita `i2c-dev`; disabilita `hciuart`, `bluetooth`; swap 256/1024 MB (`dphys-swapfile` o `rpi-swap`); imposta la password di `pi` con `userconf`, disabilita `userconfig.service`, abilita `getty@tty1` e SSH, abilita sudo senza password per la build |
| `11-fix-wifi` | `rfkill unblock wifi`, `WirelessEnabled=true` nello stato NetworkManager |
| `61-postrename` | solo con `INIT_FORMAT=systemd` (oggi **non attivo**): installa `/postrename` + lib condivisa e lo aggiunge a `/etc/rc.local` |
| `63-SplashScreen` | `fbi`; `disable_splash=1` in config.txt; parametri quiet in cmdline; crea e abilita `splashscreen.service` che mostra `~/printer_data/config/splash.png` |
| `98-remove-passwordless-sudo` | `raspi-config nonint do_sudo_pass 0` (Trixie) e/o commenta la riga di `pi` in `/etc/sudoers.d/010_pi-nopasswd` |

### Rinomina utente (runtime, primo boot)

Con `INIT_FORMAT=cloudinit-rpi` (configurazione attuale): `mainsailos-prerename` rinomina `pi` nel nome scelto in Raspberry Pi Imager (letto dallo user-data di cloud-init) e sposta la home; poi `mainsailos-postrename` esegue le trasformazioni di `postrename-lib`, ognuna dentro `run_step` (un errore non blocca le successive). Log: `/var/log/mainsailos-postrename.log`.

Le trasformazioni di `postrename-lib`, se l'utente UID 1000 **non** è `pi`:

1. ferma `moonraker klipper nginx sonar crowsnest` (+ `KlipperScreen` se trovato),
2. sostituisce `/home/pi/mainsail` nel sito nginx,
3. `sed s/pi/<utente>/g` su unit file dei servizi e su `printer_data/systemd/*.env`,
4. corregge gli shebang dei venv `klippy-env`, `moonraker-env`, `.KlipperScreen-env`,
5. corregge regole polkit di Moonraker/KlipperScreen, log path e logrotate di crowsnest,
6. ricrea i symlink di crowsnest, sonar, timelapse, mainsail.cfg, kiauh,
7. `daemon-reload`, riavvia i servizi, si disabilita/si cancella, **reboot**.

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
