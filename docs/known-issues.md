# Problemi noti e punti fragili

Risultato di un'analisi statica del codice (2026-10-05, versione 2.0.10). **Non** sono stati verificati su un dispositivo: ogni voce indica come confermarla. Severità:

- **Alta**: l'utente finale vede un malfunzionamento.
- **Media**: malfunzionamento in condizioni specifiche (utente rinominato, hardware specifico, ecc.).
- **Bassa**: codice morto, incoerenze, rischio futuro.

## Riepilogo

| ID | Sev. | Area | Titolo |
|---|---|---|---|
| [B01](#b01) | ~~Alta~~ — | Build | Repo esterni non bloccati a versione — **non è un bug (scelta voluta)** |
| [B02](#b02) | ~~Alta~~ ✅ | Moonraker | `update_manager KlipperScreen` punta al repo upstream invece che a klipperscreen4pellet — **risolto** |
| [B03](#b03) | ~~Alta~~ ✅ | Permessi | `moonraker.conf` e `KAMP_Settings.cfg` copiati da root → non modificabili da Mainsail — **risolto** |
| [B04](#b04) | ~~Media~~ ✅ | Rename utente | Molti path `/home/pi` non vengono corretti da `postrename` — **risolto (solo utente `pi`)** |
| [B05](#b05) | ~~Media~~ ✅ | Rename utente | `postrename` può abortire a metà lasciando i servizi fermi — **risolto da upstream** |
| [B06](#b06) | ~~Media~~ ✅ | Boot | Splash screen: parametri quiet scritti nel `cmdline.txt` sbagliato — **risolto** |
| [B07](#b07) | Bassa | Hardware | Pulsante di spegnimento su GPIO3 in conflitto con I2C abilitato — **non si corregge** |
| [B08](#b08) | ~~Media~~ ✅ | USB | Montaggio chiavette: solo partizioni `sdX[0-9]`, un solo device alla volta — **risolto** |
| [B09](#b09) | ~~Media~~ ✅ | Pacchetti | `initramfs-tools` resta in hold per sempre sull'immagine finale — **risolto (hold rimosso)** |
| [B10](#b10) | ~~Bassa~~ ✅ | udev | Virgola mancante nelle regole udev Wi-Fi powersave e CAN — **risolto** |
| [B11](#b11) | ~~Bassa~~ ✅ | Swap | Il resize dello swap non viene mai eseguito — **risolto da upstream** |
| [B12](#b12) | ~~Bassa~~ ✅ | Klipper | Pulizia `c_helper.so` corrotto solo se di dimensione 0 al runtime — **risolto** |
| [B13](#b13) | ~~Bassa~~ ✅ | Wi-Fi | `headless_nm`: password < 8 caratteri lascia una connessione rotta — **risolto** |
| [B14](#b14) | Bassa | CI | `CustoPiZer@main` non bloccato |
| [B15](#b15) | Bassa | Varie | Residui MainsailOS (link, patch, branding, variabili inesistenti) — **in gran parte risolto** |
| [B16](#b16) | Media | Sicurezza | Credenziali di default `pi`/`raspberry` con SSH attivo — **non si corregge (scelta consapevole)** |
| [B17](#b17) | ~~Media~~ — | Build | Immagine base `raspios_lite_arm64_latest` non bloccata — **non è un bug (scelta voluta)** |
| [B18](#b18) | Media | Trixie | Componenti G1 non ancora verificati su Trixie (power button, Obico, KlipperScreen fork) |
| [B19](#b19) | ~~Alta~~ ✅ | KlipperScreen | I dialoghi di conferma si chiudono da soli in meno di un secondo — **risolto nel fork (da unire)** |
| [B20](#b20) | ~~Media~~ ✅ | KlipperScreen | Il resoconto di screw adjustment non si apre più — **risolto nel fork (da unire)** |
| [B21](#b21) | Alta | Wi-Fi | Dal touchscreen non si aggiunge né rimuove una rete (polkit) — da verificare su Trixie |
| [B22](#b22) | ~~Media~~ ✅ | Moonraker | `moonraker.conf` sovrascritto da G1-Configs: sezioni G1OS perse, KlipperScreen riavvia Klipper — **risolto in G1-Configs (da unire)** |
| [B23](#b23) | Bassa | Sistema | Fuso orario `Europe/London` di default |
| [B24](#b24) | Bassa | Crowsnest | `crowsnest.service` in `failed` sulle stampanti senza camera |
| [B25](#b25) | ~~Media~~ ✅ | Moonraker | `moonraker.asvc` vuoto → «not permitted to restart service 'crowsnest'» — **risolto** |

---

### B01
**Repo esterni non bloccati a versione** — **Non è un bug (decisione 2026-10-05)**

> L'immagine viene buildata dalla pipeline solo ai rilasci: prendere `HEAD` dei repo esterni al momento del rilascio è il comportamento voluto. Resta valido il consiglio di debug: per regressioni tra due versioni dell'immagine, confrontare prima i commit dei repo esterni. Descrizione originale:

Tutti i `git clone` dei moduli `5x`/`6x` prendono `HEAD` del branch di default (Mainsail: `releases/latest`). Rebuildare lo stesso commit G1OS in giorni diversi produce immagini diverse; un commit rotto su klipper4pellet / klipperscreen4pellet / G1-Configs / Moonraker finisce direttamente nell'immagine.

- Dove: `modules/generic/50-klipper4pallet`, `51-moonraker`, `52-mainsail`, `53-crowsnest`, `54-timelapse`, `55-sonar`, `56-klipperscreen4pellet`, `57-Kiauh`, `58-Obico`, `60-Kamp`, `62-PowerButton`, `69-G1Config`.
- Conferma: confronta `git -C ~/<repo> log -1` su due unità flashate da build diverse.
- Fix: aggiungere `-b <tag>` / `git checkout <sha>` almeno per i repo Ginger (klipper4pellet, klipperscreen4pellet, G1-Configs), magari definendo le versioni in `00-config`. Per bug di regressione tra due versioni dell'immagine, **prima** di cercare nel codice G1OS confrontare i commit dei repo esterni.

### B02
**`update_manager KlipperScreen` punta al repo sbagliato** — ✅ Risolto in G1OS (2026-10-05), **senza effetto sul dispositivo**

> Il `moonraker.conf` finale è quello di G1-Configs (vedi [B03](#b03)), che sovrascrive la sezione aggiunta da G1OS; lì `origin` è già il fork ma `managed_services = klipper` (dovrebbe essere `KlipperScreen`) e `[update_manager] channel: dev`. Fix in G1OS: `origin` → `gingeradditive/klipperscreen4pellet.git`, `primary_branch: master`, rimosso il commento "Uncomment to enable". Verificato che `scripts/system-dependencies.json` e `scripts/KlipperScreen-requirements.txt` esistono nel fork. Verifica sul dispositivo: KlipperScreen non deve risultare "invalid" nell'Update Manager. Descrizione originale:

`modules/generic/files/moonraker_klipperscreen4pellet.conf:8` → `origin: https://github.com/KlipperScreen/KlipperScreen.git`, ma `~/KlipperScreen` è un clone di `gingeradditive/klipperscreen4pellet`. Moonraker confronta `origin` con il remote reale: il repo viene segnato come **invalid** (aggiornamenti bloccati) oppure, dopo un "recover", verrebbe riallineato all'upstream perdendo le modifiche pellet.

- Conferma: in Mainsail → Machine → Update Manager, KlipperScreen appare "invalid"/"dirty"; `grep -A3 "KlipperScreen" ~/printer_data/logs/moonraker.log`.
- Fix: `origin: https://github.com/gingeradditive/klipperscreen4pellet.git` (+ `primary_branch` corretto). Verificare anche che `scripts/system-dependencies.json` esista nel fork.
- Nota: i commenti nello stesso file ("Uncomment to enable") non corrispondono, la sezione è già attiva.

### B03
**File di config creati da root** — ✅ Risolto (2026-10-05)

> Fix: in `51-moonraker` e `60-Kamp` `cp`/`ln -s` girano con `sudo -u "${BASE_USER}"`. **Correzione dell'analisi**: per `moonraker.conf` era un falso positivo, perché `install-moonraker.sh` (eseguito come `pi`) crea già il file e il `cp` di root sovrascrive un file esistente mantenendo l'owner `pi`. Per lo stesso motivo `G1-Configs/install.sh` riesce a sovrascriverlo: **sul dispositivo il `moonraker.conf` è quello di `G1-Configs/Configs/moonraker.conf`**, e le sezioni aggiunte da G1OS (`54-timelapse`, `56-klipperscreen4pellet`, `60-Kamp`) vanno perse. Il problema era reale solo per `KAMP_Settings.cfg` (file nuovo, creato da root). Verifica: `ls -l ~/printer_data/config/`. Descrizione originale:

- `modules/generic/51-moonraker:68` → `cp /files/moonraker.conf …/config/moonraker.conf` eseguito da root ⇒ owner `root:root`.
- `modules/generic/60-Kamp:42` → `cp …/KAMP_Settings.cfg …/config/` da root.

Moonraker gira come `pi`: Mainsail può leggere ma non salvare questi file (errore di salvataggio nell'editor). A meno che `G1-Configs/install.sh` (69) non faccia un `chown -R` su `printer_data` — da verificare in quel repo.

- Conferma: `ls -l ~/printer_data/config/` su un dispositivo appena flashato.
- Fix: `chown "${BASE_USER}:${BASE_USER}"` dopo i `cp`, oppure `sudo -u "${BASE_USER}" cp …`.

### B04
**Path `/home/pi` non gestiti da `postrename`** — ✅ Risolto (2026-10-05): supportato solo l'utente `pi`

> Fix: `mainsailos-prerename` non rinomina più l'utente: se lo user-data di Imager chiede un nome diverso, lo riscrive in `pi` (password, chiavi SSH e sudo del cliente vengono applicati a `pi`, `usermod -p` per l'hash perché cloud-init lo ignora su utenti esistenti). La home resta `/home/pi` e `mainsailos-postrename` è un no-op. Il flusso legacy `INIT_FORMAT=systemd` (non attivo) rinominerebbe ancora l'utente. Verifica su hardware: flash con utente `foo` in Imager → login come `pi` con la password scelta, `journalctl -t mainsailos-prerename`. Descrizione originale:

Se in Raspberry Pi Imager si sceglie un utente diverso da `pi`, la home viene spostata (su Trixie da `mainsailos-prerename`) e `postrename-lib` corregge solo alcuni file (vedi [modules.md](modules.md#postrename-runtime-primo-boot)). Restano rotti:

| Elemento | Dove nasce |
|---|---|
| symlink assoluto `config/KAMP → /home/pi/Klipper-Adaptive-Meshing-Purging/Configuration` | `60-Kamp:41` |
| `splashscreen.service` → `/home/pi/printer_data/config/splash.png` | `63-SplashScreen:32` |
| `moonraker-obico.service` + venv + `moonraker-obico.cfg` | `58-Obico` (non in `SERVICES`) |
| `g1-flask.service` (`/home/pi/G1-Configs/Flask`), tema Mainsail in `/home/pi/printer_data/config/.theme`, `chown pi:pi` | `G1-Configs/install.sh` (path scritti a mano) |
| pi-power-button | `62-PowerButton` |
| `usbstick-handler`, symlink `gcodes/media` | ok (path assoluto `/media`) |

Inoltre `postrename` usa `sed 's/pi/<user>/g'` su interi file: sostituisce **ogni** occorrenza di "pi" (es. `api`, `pip`, `spi`, `gpio` se presenti), fragile per unit file aggiunti in futuro.

- Fix: aggiungere i servizi/file mancanti a `modules/generic/files/cloudinit/postrename-lib` (vale sia per cloud-init sia per il flusso legacy), usare `s|/home/pi/|/home/<user>/|g` e `s/^User=pi$/…/`, e creare il symlink KAMP relativo. In alternativa supportare ufficialmente solo l'utente `pi`.

### B05
> ✅ **Risolto da upstream** (cherry-pick di `7cc5ccc`): la logica è in `postrename-lib`, con `getent passwd 1000`, `find -maxdepth 1`, ogni passo dentro `run_step` (un errore non blocca il resto), controllo di `/boot/firmware/cmdline.txt` e della keyword `resize` di Trixie. Su Trixie il flusso attivo è quello cloud-init. Descrizione originale (codice vecchio):

**`postrename` può interrompersi a metà** — Media

`modules/raspberry/files/postrename` usa `set -Ee`. Punti a rischio:

- `:32` `DEFAULT_USER=$(grep "1000" /etc/passwd …)` → match su qualunque campo contenente "1000": può restituire più righe.
- `:35` `KS_INSTALLED` richiede esattamente `1` risultato di `find -name KlipperScreen`; se ce ne sono di più KlipperScreen viene ignorato.
- `:165` `find … -iname crowsnest | xargs ln -s` → se trova più file con quel nome, il secondo `ln` fallisce ("File exists") ⇒ script esce dopo aver **fermato i servizi**. Al boot successivo riparte da capo (la riga in `rc.local` è rimossa solo alla fine).
- `:205` controlla `/boot/cmdline.txt`, ma su Bookworm il file reale è `/boot/firmware/cmdline.txt`: il controllo "firstrun non ancora finito" non funziona.

- Conferma: `journalctl -b -1 -u rc-local`, presenza di `/postrename` dopo il primo boot.
- Fix: `getent passwd 1000 | cut -d: -f1`, `ln -sf` + `head -n1`, path `cmdline.txt` corretto.

### B06
**Splash screen: `cmdline.txt` sbagliato** — ✅ Risolto (2026-10-05)

> Fix: `63-SplashScreen` ora usa `${BOOT_PATH}/config.txt` e `${BOOT_PATH}/cmdline.txt`. Restano aperti i dettagli minori sotto (splash.png, doppio enable, `sudo`). Descrizione originale:

`modules/raspberry/63-SplashScreen:22` usa `/boot/cmdline.txt`; su Bookworm quello è un file segnaposto, il vero file è `$BOOT_PATH/cmdline.txt` (`/boot/firmware/cmdline.txt`). I parametri `logo.nologo consoleblank=1 loglevel=0 quiet` quindi **non vengono applicati** (boot verboso, logo Raspberry visibile). Anche `config.txt` è hard-coded (`:21`) invece di `$BOOT_PATH`.

Altri dettagli: l'immagine `splash.png` non è fornita da questo repo (deve arrivare da G1-Configs; se manca, `fbi` fallisce); il servizio viene abilitato due volte; `sudo` superfluo (si è già root).

- Conferma: `cat /boot/firmware/cmdline.txt` su un dispositivo.
- Fix: usare `"$BOOT_PATH"/cmdline.txt` e `"$BOOT_PATH"/config.txt`.

### B07
**Power button su GPIO3 vs I2C** — Bassa — **Non si corregge (decisione 2026-10-05)**

> L'immagine è destinata **solo a Raspberry Pi 4**, quindi l'incompatibilità di `RPi.GPIO` con Pi 5 non è rilevante. Non modificare il modulo `62-PowerButton`. Resta da tenere presente il conflitto con l'I2C se si collegano dispositivi I2C.

`pi-power-button` ascolta su GPIO3 (pin 5), che è anche **SCL dell'I2C** abilitato da `boot-config.txt:53` (`dtparam=i2c_arm=on`). Un dispositivo I2C sul bus può causare spegnimenti indesiderati, o il pulsante può interferire con l'I2C.

- Conferma: `systemctl status listen-for-shutdown` / `ps aux | grep listen-for-shutdown`.
- Fix: valutare `dtoverlay=gpio-shutdown` in `boot-config.txt` (nativo, funziona su tutti i modelli) invece del repo esterno.

### B08
**Montaggio chiavette USB limitato** — ✅ Risolto (2026-10-05)

> Fix: la regola udev ora è `SUBSYSTEM=="block", KERNEL=="sd[a-z]*", ENV{ID_FS_USAGE}=="filesystem"` (monta anche chiavette senza tabella partizioni e partizioni ≥10) e il messaggio usa `${ENABLEUSB_RULE_FILE}`. Il punto di mount unico `/media` e la sola lettura sono **voluti**: la stampante ha una sola porta USB e si supporta ufficialmente una partizione alla volta. Verifica: chiavetta formattata senza partizioni → file visibili in `gcodes/media`. Descrizione originale:

`modules/generic/61-EnableUSB`:

- `:24` regola udev `KERNEL=="sd[a-z][0-9]"`: chiavette senza tabella partizioni (`/dev/sda`) o partizione ≥10 non vengono montate.
- `:34` `pmount … %I .` monta tutte le chiavette **sullo stesso punto** (`/media`): con due chiavette la seconda fallisce o copre la prima.
- montaggio `-r` (sola lettura): non si possono salvare file sulla chiavetta (probabilmente voluto).
- `:60` stampa `${RULES_FILE}` (variabile inesistente) — solo cosmetico.
- `udevadm control --reload-rules` nel chroot fallisce silenziosamente (è a sinistra di `&&`).

### B09
**`initramfs-tools` in hold permanente** — ✅ Risolto (2026-10-05)

> Fix definitivo: seguendo upstream (`a607941`) il blocco dei pacchetti è stato **rimosso del tutto**: niente più `HOLD_PKGS`, `apt-mark hold` in `00-upgrade` né modulo `99-unhold-packages`. Se la build fallisse durante `apt-get upgrade` per un problema di initramfs/kernel nel chroot, è il motivo per cui era stato introdotto `bbc3c10`. Verifica: `apt-mark showhold` vuoto. Descrizione originale:

`00-upgrade:29` mette in hold `HOLD_PKGS`; `99-unhold-packages:20` è commentato e usa comunque la variabile sbagliata (`PKGS` invece di `HOLD_PKGS`). L'immagine finale ha `initramfs-tools` bloccato: `apt upgrade` sul dispositivo non lo aggiorna mai, e aggiornamenti kernel che lo richiedono possono rimanere "kept back".

- Conferma: `apt-mark showhold`.
- Fix: decidere se l'hold è voluto solo durante la build (allora riattivare l'unhold con `HOLD_PKGS`) o anche dopo (documentarlo).

### B10
**Virgole mancanti nelle regole udev** — ✅ Risolto (2026-10-05)

> Fix: aggiunte le virgole in `070-wifi-powersave.rules` e `canbus/10-can.rules`. Verifica: `udevadm verify /etc/udev/rules.d/*.rules`. Descrizione originale:

- `modules/generic/files/070-wifi-powersave.rules:3` `KERNEL=="wlan*" \` → manca `,` prima di `RUN+=`.
- `modules/generic/files/canbus/10-can.rules:1` `KERNEL=="can*"  ATTR{…}` → manca `,`.

udev tollera la virgola mancante ma logga un warning; versioni future potrebbero scartare la regola. Su Trixie (systemd 257) è disponibile `udevadm verify`.
- Conferma: `udevadm verify /etc/udev/rules.d/*.rules`; `iw wlan0 get power_save` deve dire `off`.

### B11
**Resize swap mai eseguito** — ✅ Risolto da upstream (`426a216`): usa `${SWAP_CONF_FILE}` su Bookworm e un drop-in `rpi-swap` su Trixie.

Descrizione originale:

`modules/raspberry/10-config-raspberry:73` testa `${PICONFIG_SWAP_CONF_FILE}` (non definita) invece di `${SWAP_CONF_FILE}` ⇒ il blocco non viene mai eseguito, lo swap resta al default (100 MB). Correggere solo dopo aver valutato l'impatto sulla SD.

### B12
**Pulizia `c_helper.so`** — ✅ Risolto (2026-10-05)

> Fix: `ExecStartPre` in `klipper.service` prova a caricare `c_helper.so` con `ctypes.CDLL` (Python del venv) e lo cancella se non è caricabile; Klipper lo ricompila. Prefisso `-`: un errore del controllo non blocca l'avvio. `Wants=network-online.target` **non** aggiunto di proposito: Klipper non usa la rete e ritarderebbe l'avvio sulle stampanti offline (come upstream). Verifica: `truncate -s 4096 ~/klipper/klippy/chelper/c_helper.so && sudo systemctl restart klipper` → Klipper riparte. Descrizione originale:

Il fix `ca4b0dd` rimuove `c_helper.so` in build e in `klipper.service` (`ExecStartPre`) cancella il file **solo se vuoto** (`-size 0`). Un file non vuoto ma corrotto (scrittura interrotta da spegnimento durante la prima compilazione) non viene rimosso ⇒ Klipper non parte ("can't connect to klipper").
- Workaround sul campo: `rm ~/klipper/klippy/chelper/c_helper.so && sudo systemctl restart klipper`.
- Nota: `klipper.service` ha `After=network-online.target` senza `Wants=network-online.target` (l'ordinamento non ha effetto).

### B13
**`headless_nm` con password non valida** — ✅ Risolto (2026-10-05)

> Fix: `validate_config` controlla SSID e password (tramite `wpa_passphrase`) **prima** di toccare i file: se non validi logga l'errore ed esce lasciando intatta la connessione esistente (anche quella di Imager, che prima veniva cancellata). `HIDDEN` vuoto o non valido → `false`. La connessione viene scritta in `.preconfigured.nmconnection.tmp` (0600) e spostata al posto di quella vecchia solo a fine generazione. `headless_nm.txt` non valido resta in `/boot/firmware` (va corretto dall'utente). Descrizione originale:

`files/headless-nm/headless_nm:138`: con password < 8 o > 63 caratteri `wpa_passphrase` fallisce; con `set -e` lo script esce dopo aver già scritto `preconfigured.nmconnection` senza sezione di sicurezza e senza `chmod 0600` (`:212`) ⇒ NetworkManager ignora il file, e `headless_nm.txt` **non** viene cancellato. Anche `HIDDEN` vuoto produce `hidden=` (non valido).
- Conferma: `journalctl -t headless_nm`.

### B14
**CustoPiZer non bloccato** — Bassa

`.github/workflows/build.yml:139` e `release.yml:201` usano `OctoPrint/CustoPiZer@main`: una modifica upstream può rompere la build senza cambi in questo repo. Fissare a un SHA.

### B15
**Residui MainsailOS** — Bassa — in gran parte risolto (2026-10-05)

> Fatto: rimossa `patches/`; `cliff.toml` punta a `gingeradditive/G1OS`; `README.md` e `CONTRIBUTING.md` riscritti per G1OS (con credits a MainsailOS/Mainsail Crew); `config.yml` dichiara solo `pi4-64bit`; tolto "SV1" da `klipper.service`; modulo rinominato `50-klipper4pellet`. **Restano aperti**: icona Imager `os.mainsail.xyz/rpi-imager.png` in `config.yml` (serve un PNG Ginger ospitato), link Discord Mainsail in `.github/ISSUE_TEMPLATE/config.yml` e `.github/label-actions.yml`, `.github/FUNDING.yml` (Patreon/Ko-fi Mainsail), testo `description` in `config.yml`. Descrizione originale:

- `patches/*.sh`: URL `mainsail-crew/G1OS` inesistenti, pensati per Bullseye. Da rimuovere o riscrivere.
- `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md` (link `mainsail-crew/MainsailOS`), issue template, `rpi_json.icon` (`os.mainsail.xyz`): branding non G1.
- Il README cita CustomPiOS, ma la build usa CustoPiZer.
- Nome file `50-klipper4pallet` (typo "pallet").
- `klipper.service` Description "SV1".
- `config.yml` dichiara in `rpi_json.devices` anche `pi3-64bit` e `pi5-64bit`, ma il target supportato è solo Raspberry Pi 4.

### B16
**Credenziali di default con SSH attivo** — Media (scelta consapevole, 2026-10-05)

Da `426a216` (Trixie non ha più un utente preconfigurato): `BASE_PASSWORD=raspberry` in `00-config`, impostata da `10-config-raspberry` con `userconf`; `userconfig.service` disabilitato; SSH abilitato. Ogni stampante esce con `pi`/`raspberry` raggiungibile in SSH, a meno che l'utente non imposti credenziali in Raspberry Pi Imager (cloud-init).
- Decisione (2026-10-05): si lascia `pi`/`raspberry` per l'accesso dell'assistenza; chi vuole credenziali diverse le imposta in Raspberry Pi Imager.
- **Non usare `chage -d 0 pi`**: `KlipperScreen.service` (klipperscreen4pellet) ha `PAMName=%u`, con password scaduta il controllo account PAM blocca la sessione e il touchscreen non parte.
- Alternative valutate e scartate per ora: `BASE_PASSWORD` non banale (comunque uguale su tutte le unità), SSH solo con chiave (`PasswordAuthentication no` + chiave Ginger).

### B17
**Immagine base non bloccata** — **Non è un bug (decisione 2026-10-05)**

> Come [B01](#b01): si builda ai rilasci e si vuole l'ultima Raspberry Pi OS. Se una build fallisce sulla verifica sha256 nei giorni di uscita di una nuova release, rilanciarla più tardi. Descrizione originale:

`config.yml` usa `https://downloads.raspberrypi.org/raspios_lite_arm64_latest.torrent` (+ `.sha256`). Ogni nuova release di Raspberry Pi OS cambia la base senza commit in questo repo (stesso problema di [B01](#b01)); se torrent e sha256 vengono aggiornati in momenti diversi, la verifica fallisce.
- Fix: usare l'URL datato `…/images/raspios_lite_arm64-AAAA-MM-GG/…` come si faceva per Bookworm.

### B18
**Componenti G1 da verificare su Trixie** — Media

Verificato in modo statico (2026-10-05): tutti i pacchetti apt dei moduli G1 esistono in Trixie; i requirements di klipper4pellet hanno già pin per Python ≥ 3.12; la build CI su Trixie (`build.yml`) completa tutti i moduli.
- `62-PowerButton`: l'immagine base Trixie lite (2026-09-15) include `python-is-python3` (lo shebang `#!/usr/bin/env python` funziona) e `python3-rpi-lgpio` (shim `RPi.GPIO`). Lo script init.d gira tramite il generatore SysV di systemd 257 (deprecato: si romperà con systemd ≥ 258 / Debian Forky). Da verificare a runtime: `wait_for_edge` con rpi-lgpio.
- `56-klipperscreen4pellet`: `PyGObject<3.51` compila su Python 3.13 (deps di build in `system-dependencies.json`, CI verde).
- `58-Obico`: installer esterno, avvio con Python 3.13 non verificato.
- `G1-Configs/install.sh`: `sudo pip3 install flask` fallisce per PEP 668 (innocuo: Flask arriva da apt in `69-G1Config`).

Checklist su Pi 4 (immagine da `build.yml`):
1. Primo boot **senza** personalizzazione in Imager: login `pi`/`raspberry`, Mainsail e KlipperScreen funzionanti.
2. Primo boot con utente `foo` + password in Imager ([B04](#b04)): login come `pi` con la password scelta, `/home/pi` intatta, `journalctl -t mainsailos-prerename`.
3. `ls -l ~/printer_data/config/` tutto `pi:pi`; salvataggio di `moonraker.conf` da Mainsail ([B03](#b03)).
4. Update Manager: KlipperScreen non "invalid" ([B02](#b02)).
5. Pulsante di spegnimento: `systemctl status listen-for-shutdown`, pressione → shutdown.
6. `systemctl status moonraker-obico`.
7. Chiavetta senza tabella partizioni → file in `gcodes/media` ([B08](#b08)).
8. `udevadm verify /etc/udev/rules.d/*.rules`, `iw wlan0 get power_save` = `off` ([B10](#b10)).
9. `truncate -s 4096 ~/klipper/klippy/chelper/c_helper.so && sudo systemctl restart klipper` → Klipper riparte ([B12](#b12)).
10. `headless_nm.txt` con password di 5 caratteri → connessione esistente intatta, errore in `journalctl -t headless_nm` ([B13](#b13)).
11. Senza sessione SSH aperta: aggiungere e rimuovere una rete Wi-Fi da KlipperScreen ([B21](#b21)).
12. Disattiva motori da KlipperScreen: il dialogo resta aperto e «Accetta» invia `M18` ([B19](#b19)); `SCREW_ADJUSTMENT` apre il pannello `bed_level` ([B20](#b20)).

---

Le voci B19–B24 vengono dall'analisi sul campo del 2026-10-02 (G1-0096-26, G1OS 2.1.0 Bookworm, confronto con G1-0063-25 su 2.0.5), verificate il 2026-10-05 sull'HEAD dei repo.

### B19
**Dialoghi di conferma che si chiudono da soli** — ✅ Risolto in `klipperscreen4pellet`, branch `fix/confirm-dialogs-screws-panel` (da unire in `master`)

Il commit upstream `e3dfb677` (13 lug 2026, entrato con il merge `f4923aa6`) chiude le conferme quando `idle_timeout.state` diventa `Printing`. Sulla G1 succede ogni secondo per `[delayed_gcode FEEDER_CHECK_STATUS]` (in ogni `printer.cfg` di G1-Printers), quindi ogni conferma (Disattiva motori, Force Z, `SAVE_CONFIG`) si chiude in < 1 s e il tocco finisce sul pannello sotto.
- Fix: in `ks_includes/notification_handler.py` le conferme si chiudono solo quando `print_stats.state` diventa `printing`; tolto il `return` che saltava i controlli successivi.

### B20
**Resoconto di screw adjustment non visualizzato** — ✅ Risolto in `klipperscreen4pellet`, stesso branch di [B19](#b19)

Da upstream `c2187163`/`ce1fbadd` il pannello `bed_level` si apre solo se l'aggiornamento contiene `max_deviation` falso, ma gli aggiornamenti contengono solo i campi cambiati e `max_deviation` (sempre `None`) non arriva mai. Anche il `return` di [B19](#b19) saltava il controllo.
- Fix: il pannello si apre quando l'aggiornamento porta `results` non vuoti e `max_deviation` (letto dallo stato completo) non è impostato; resta l'intento upstream di non aprirlo con `MAX_DEVIATION=`.

### B21
**Wi-Fi dal touchscreen: `Insufficient privileges`** — Alta — da verificare su Trixie

Su Bookworm (2.0.5 e 2.1.0): la sessione di KlipperScreen su `tty7` non è mai attiva (Xorg su `tty2`) e `49-polkit-pkla-compat.rules` risponde «no» prima di `KlipperScreen.rules`; funziona solo con una sessione SSH `pi` aperta. Non è una regressione di 2.1.0.
- Su Trixie la base lite ha `polkitd` senza `polkitd-pkla` e CustoPiZer installa `polkitd` al posto di `policykit-1`: il file `49-polkit-pkla-compat.rules` non dovrebbe esistere e decide `KlipperScreen.rules` (gruppi `network`/`klipperscreen`, indipendente dalla sessione). Verifica: `ls /usr/share/polkit-1/rules.d/ /etc/polkit-1/rules.d/` e punto 11 della checklist di [B18](#b18).
- Se servisse un fix: regola con nome che preceda `49-` (es. `20-KlipperScreen.rules`) nell'installer del fork.
- Stampanti Bookworm già consegnate: non correggibili da una nuova immagine.

### B22
**`moonraker.conf` sovrascritto da G1-Configs** — ✅ Risolto in `G1-Configs`, branch `fix/moonraker-update-manager` (da unire in `main`)

`G1-Configs/install.sh` (`69-G1Config`) copia `Configs/moonraker.conf` sopra quello costruito da G1OS (vedi [B03](#b03)): le sezioni aggiunte da `56-klipperscreen4pellet` e `60-Kamp` vanno perse (quella di `54-timelapse` è commentata). Nella versione G1-Configs `[update_manager KlipperScreen]` aveva `managed_services = klipper` e niente `virtualenv`/`requirements`.
- Fix in G1-Configs: `managed_services = KlipperScreen`, `virtualenv`/`requirements`/`system_dependencies`, aggiunto `[update_manager Klipper-Adaptive-Meshing-Purging]`.
- `channel: dev` **resta**: klipper4pellet ha solo i tag upstream (ultimo raggiungibile `v0.13.0`), con `stable` Moonraker riporterebbe Klipper a quel tag perdendo le modifiche pellet. Per usare `stable` bisogna prima taggare le release di klipper4pellet.
- `install.sh` copia la config solo all'installazione: le stampanti già consegnate non ricevono il fix con l'aggiornamento di G1-Configs.
- `modules/generic/files/moonraker_klipperscreen4pellet.conf` e `moonraker_kamp.conf` di G1OS restano senza effetto finché G1-Configs sovrascrive il file.

### B23
**Fuso orario `Europe/London`** — Bassa

G1OS non imposta il fuso orario: resta il default di Raspberry Pi OS (log un'ora indietro rispetto all'Italia), a meno che non lo si scelga in Raspberry Pi Imager.
- Fix possibile: `timedatectl set-timezone`/`/etc/timezone` in `10-config-raspberry` (decidere il default, le stampanti vanno anche all'estero).

### B24
**`crowsnest.service` in `failed` senza camera** — Bassa

`ERROR: No usable Devices Found. Stopping` a ogni avvio sulle stampanti senza camera (già così su 2.0.5). Innocuo ma sporca `systemctl --failed`.
- Da decidere: camera sempre montata? Altrimenti lasciare così o disabilitare il servizio di default.

### B25
**`moonraker.asvc` vuoto** — ✅ Risolto (2026-10-05)

Segnalato dal campo: warning `[update_manager crowsnest]: Moonraker is not permitted to restart service 'crowsnest'`; il workaround era cancellare `~/printer_data/moonraker.asvc` (vuoto) e riavviare. Moonraker scrive la lista di default solo se il file **non esiste**: un file vuoto resta vuoto per sempre. Nessun installer della build (Moonraker, crowsnest v5, sonar, Obico, KlipperScreen, G1-Configs) scrive il file; Moonraker lo crea al primo avvio con `write_text()` senza `fsync`, e uno spegnimento a interruttore subito dopo (ext4, allocazione ritardata) lascia un file da 0 byte. Causa probabile, non confermata.
- Fix in `51-moonraker`: il file viene creato in build con `moonraker/assets/default_allowed_services` (contiene `crowsnest`, `KlipperScreen`, `sonar`, `moonraker-obico`), e un drop-in `moonraker.service.d/10-asvc.conf` lo cancella prima dell'avvio se è vuoto (Moonraker lo rigenera).
- Verifica: `: > ~/printer_data/moonraker.asvc && sudo systemctl restart moonraker` → file ripopolato, nessun warning in Mainsail.
