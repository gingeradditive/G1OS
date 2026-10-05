# Architettura

## 1. Struttura del repository

```
config.yml                 # Matrice dei target di build (immagine base, ARCH, special modules, metadati RPi Imager)
VERSION                    # Versione corrente → RELEASE_TAG
modules/
  generic/                 # Script eseguiti per TUTTI i target
    files/                 # File statici copiati in /files dentro il chroot
  raspberry/               # Script solo per type: raspberry (si fondono con generic/)
    files/
  armbian/                 # Script solo per type: armbian (attualmente inutilizzati)
  special/                 # Script per singola board, elencati in special_modules (inutilizzati)
patches/                   # Script legacy di MainsailOS da lanciare a mano su dispositivi vecchi (obsoleti)
.github/workflows/
  build.yml                # Build su push a develop / PR / manuale → artifact
  release.yml              # Release manuale: bump VERSION, tag, build, upload, rpi-imager.json, changelog
cliff.toml, cliff-release.toml   # Config git-cliff per CHANGELOG
```

## 2. Pipeline di build

```
push develop / PR / workflow_dispatch
        │
        ▼
generate-matrix  ── legge config.yml → una job per ogni voce (oggi: raspberry_pi-arm64-bookworm)
        │
        ▼
build (runner ubuntu-24.04-arm)
  1. RELEASE_TAG = contenuto di VERSION
  2. scarica immagine base (torrent via aria2c o wget) e verifica sha256
  3. xz -d → build/input.img
  4. cp modules/generic/*  → scripts/
     cp modules/<type>/*   → scripts/        (stessi nomi ⇒ il file del type SOVRASCRIVE il generic)
     rm scripts/*.disabled
  5. (se special_modules) copia modules/special/<nome> in scripts/
  6. config.local: ogni chiave di matrix.env diventa EDITBASE_<chiave>=<valore>
  7. OctoPrint/CustoPiZer@main, input custopizer: "main"  (workspace=build, scripts=scripts, env RELEASE_TAG)
  8. rinomina output.img → <DATA>-G1OS-<name>-<VERSION>.img, xz -9, sha256
  9. upload artifact (build.yml) / upload su GitHub Release + JSON RPi Imager (release.yml)
```

### Cosa fa CustoPiZer (importante per capire i bug)

Codice: <https://github.com/OctoPrint/CustoPiZer/tree/main/src> (`customize`, `common.sh`, `start_chroot_script`, `end_chroot_script`).

1. Copia `input.img` → `output.img`, la ingrandisce di `EDITBASE_IMAGE_ENLARGEROOT` MB (7000), monta root e boot.
2. Esporta **`BOOT_PATH`**: `/boot/firmware` se compare in `/etc/fstab` (Bookworm e Trixie), altrimenti `/boot`.
3. Esegue `start_chroot_script`: `apt-get update`, installa polkit, crea `/usr/sbin/policy-rc.d` con `exit 101` (**i servizi non partono durante la build**).
4. Copia `scripts/files/` → **`/files`** nel chroot.
5. Esegue ogni file di `scripts/` (esclusi dir, dotfile, `*.disabled`) **in ordine `sort` alfabetico**, con `chroot . /bin/bash /chroot_script`. Ogni script è un processo separato: le variabili non passano da uno script all'altro (per questo tutti fanno `source /files/00-config`).
6. Rimuove `/files`, esegue `end_chroot_script` (rimuove policy-rc.d, `/common.sh`, `apt clean`).

Funzioni fornite da `/common.sh` e usate dai moduli: `install_cleanup_trap`, `systemctl_if_exists`, `is_installed`, `is_in_apt`, `echo_green`/`echo_red`.

Tutti i moduli usano `set -xe`: **un comando che fallisce interrompe la build** (eccezioni: comandi in `|| true`, la parte sinistra di `&&`, sottoshell nelle pipe senza `pipefail`).

### Ordine effettivo di esecuzione (target raspberry)

| # | Script | Origine |
|---|---|---|
| 1 | `00-upgrade` | generic |
| 2 | `10-config-raspberry` | raspberry |
| 3 | `11-fix-wifi` | raspberry |
| 4 | `30-headless-nm` | generic |
| 5 | `31-wifi-powersave-off` | generic |
| 6 | `32-canbus` | generic |
| 7 | `50-klipper4pallet` | generic |
| 8 | `51-moonraker` | generic |
| 9 | `52-mainsail` | generic |
| 10 | `53-crowsnest` | generic |
| 11 | `54-timelapse` | generic |
| 12 | `55-sonar` | generic |
| 13 | `56-klipperscreen4pellet` | generic |
| 14 | `57-Kiauh` | generic |
| 15 | `58-Obico` | generic |
| 16 | `60-Kamp` | generic |
| 17 | `60-mainsailos` | generic |
| 18 | `61-EnableUSB` | generic |
| 19 | `61-postrename` | raspberry (esce subito se `INIT_FORMAT` è `cloudinit*`) |
| 20 | `61-postrename-cloudinit` | generic (attivo con `INIT_FORMAT=cloudinit-rpi`) |
| 21 | `62-PowerButton` | generic |
| 22 | `63-SplashScreen` | raspberry |
| 23 | `69-G1Config` | generic |
| 24 | `98-remove-passwordless-sudo` | raspberry |

Note sull'ordinamento (`LC_ALL=C`): le maiuscole vengono prima delle minuscole, quindi `60-Kamp` < `60-mainsailos`. Le dipendenze implicite contano: ad es. `54-timelapse`, `56-…`, `60-Kamp` appendono a `moonraker.conf` **solo se esiste già** (creato da `51-moonraker`); `62-PowerButton` e gli installer esterni usano `sudo` e funzionano solo perché il passwordless sudo viene rimosso dopo (`98-…`).

## 3. Il sistema sul dispositivo

### Utente e path

- Utente: `pi` (UID 1000) con password **`raspberry`** (`BASE_USER` / `BASE_PASSWORD` in `modules/generic/files/00-config`), impostata in build da `10-config-raspberry` tramite `userconf`; SSH e `getty@tty1` abilitati. Molti file hanno `/home/pi` **scritto a mano** (vedi known-issues).
- Rinomina utente (Trixie, cloud-init): se in Raspberry Pi Imager si sceglie un altro nome, al primo boot `mainsailos-prerename.service` legge lo user-data di cloud-init e rinomina `pi` (account, gruppo, home); dopo `cloud-final`, `mainsailos-postrename.service` corregge servizi/venv/symlink usando `/usr/local/lib/mainsailos/postrename-lib`, logga in `/var/log/mainsailos-postrename.log` e riavvia. Il vecchio hook `/postrename` via `rc.local` resta solo per `INIT_FORMAT=systemd`.
- Hostname di default: `g1os` (da `DIST_NAME` in lowercase) → `http://g1os.local`.
- Release file: `/etc/g1os-release` → `G1OS release <VERSION> (bookworm)`.

```
/home/pi/
  klipper/                  # clone di gingeradditive/klipper4pellet
  klippy-env/               # venv Python di Klipper
  moonraker/  moonraker-env/
  mainsail/                 # build statica di Mainsail (ultima release)
  mainsail-config/          # mainsail.cfg (macro)
  crowsnest/  sonar/  moonraker-timelapse/
  KlipperScreen/  .KlipperScreen-env/   # clone di gingeradditive/klipperscreen4pellet
  kiauh/  moonraker-obico/  pi-power-button/
  Klipper-Adaptive-Meshing-Purging/
  G1-Configs/               # gingeradditive/G1-Configs (config stampante + app Flask, installer proprio)
  printer_data/
    config/                 # printer.cfg (da G1-Configs), moonraker.conf, mainsail.cfg→, timelapse.cfg→,
                            # KAMP/→, KAMP_Settings.cfg, crowsnest.conf, splash.png (atteso)
    gcodes/media → /media   # chiavette USB (61-EnableUSB)
    logs/                   # klippy.log, moonraker.log, crowsnest.log, mainsail-*.log→/var/log/nginx
    comms/                  # klippy.sock, klippy.serial
    systemd/                # klipper.env, moonraker.env, crowsnest.env, sonar.env
```

### Servizi

| Servizio | Origine unit file | Note |
|---|---|---|
| `klipper.service` | `modules/generic/files/klipper.service` | Args in `printer_data/systemd/klipper.env`; `ExecStartPre` cancella `c_helper.so` vuoto |
| `moonraker.service` | generato da `install-moonraker.sh` | porta 7125 |
| `nginx.service` | pacchetto Debian | sito `/etc/nginx/sites-available/mainsail`, porta 80 |
| `crowsnest.service` | `make install` di crowsnest | webcam 8080-8083 |
| `sonar.service` | `make install` di sonar | keepalive Wi-Fi |
| `KlipperScreen.service` | installer KlipperScreen (BACKEND X) | |
| `moonraker-obico.service` | installer obico | |
| `headless_nm.service` | `files/headless-nm/` | oneshot al boot: legge `/boot/firmware/headless_nm.txt` |
| `mainsailos-prerename.service`, `mainsailos-postrename.service` | `files/cloudinit/` | rinomina utente al primo boot (cloud-init), si disabilitano a fine lavoro |
| `g1-flask.service` | `G1-Configs/install.sh` | server Flask di G1-Configs, gira come root |
| `splashscreen.service` | generato inline in `63-SplashScreen` | `fbi` su tty1 |
| `usbstick-handler@.service` | generato inline in `61-EnableUSB` | attivato da regola udev |
| `listen-for-shutdown` | installer pi-power-button (init.d) | pulsante su GPIO3 |
| `systemd-networkd` | abilitato da `32-canbus` | interfacce `can*` a 1 Mbit |

### Rete / porte

| Porta | Servizio |
|---|---|
| 80 | nginx → Mainsail statico; `/websocket`, `/printer`, `/api`, `/access`, `/machine`, `/server` → Moonraker |
| 7125 | Moonraker (diretto) |
| 8080-8083 | crowsnest (proxati su `/webcam/` … `/webcam4/`) |

Moonraker considera *trusted* tutte le reti private (10/8, 172.16/12, 192.168/16, link-local): nessuna autenticazione in LAN.

### Configurazione hardware (Raspberry)

Da `modules/raspberry/files/boot-config.txt`, appeso a `/boot/firmware/config.txt`:

- `enable_uart=1` + `dtoverlay=disable-bt` → UART PL011 su GPIO14/15 per la MCU della stampante, Bluetooth disabilitato.
- `console=serial0,115200` rimosso da `cmdline.txt`.
- `dtparam=spi=on` (accelerometro / input shaper), `dtparam=i2c_arm=on`, modulo `i2c-dev`.
- `gpu_mem` per modello (sul Pi 4, target unico, `gpu_mem=256`).
- Swap: su Trixie `rpi-swap` (drop-in `/etc/rpi/swap.conf.d/10-mainsailos.conf`, 256 MiB, max 1024 MiB); su Bookworm `dphys-swapfile`.
- `disable_splash=1` aggiunto da `63-SplashScreen`.

## 4. Release

`release.yml` (solo manuale, input `version` SemVer):

1. Su `develop`: scrive `VERSION`, commit `chore: bump version to vX`, push, tag `X`.
2. Draft release con changelog git-cliff dal tag precedente.
3. Build della matrice (escluse voci con `build_only: true`), upload `.img.xz`, `.sha256`, JSON RPi Imager.
4. Pubblica la release, crea `rpi-imager.json` combinato (l'upload FTP è attivo solo per owner `Mainsail-Crew`, quindi mai per G1OS).
5. Rigenera `CHANGELOG.md` e committa su `develop`.

Conseguenza: ogni release aggiunge 2 commit automatici a `develop` → fare `git pull` prima di lavorare.
