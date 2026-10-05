# Problemi noti e punti fragili

Risultato di un'analisi statica del codice (2026-10-05, versione 2.0.10). **Non** sono stati verificati su un dispositivo: ogni voce indica come confermarla. Severità:

- **Alta**: l'utente finale vede un malfunzionamento.
- **Media**: malfunzionamento in condizioni specifiche (utente rinominato, hardware specifico, ecc.).
- **Bassa**: codice morto, incoerenze, rischio futuro.

## Riepilogo

| ID | Sev. | Area | Titolo |
|---|---|---|---|
| [B01](#b01) | ~~Alta~~ — | Build | Repo esterni non bloccati a versione — **non è un bug (scelta voluta)** |
| [B02](#b02) | Alta | Moonraker | `update_manager KlipperScreen` punta al repo upstream invece che a klipperscreen4pellet |
| [B03](#b03) | Alta | Permessi | `moonraker.conf` e `KAMP_Settings.cfg` copiati da root → probabilmente non modificabili da Mainsail |
| [B04](#b04) | Media | Rename utente | Molti path `/home/pi` non vengono corretti da `postrename` |
| [B05](#b05) | ~~Media~~ ✅ | Rename utente | `postrename` può abortire a metà lasciando i servizi fermi — **risolto da upstream** |
| [B06](#b06) | ~~Media~~ ✅ | Boot | Splash screen: parametri quiet scritti nel `cmdline.txt` sbagliato — **risolto** |
| [B07](#b07) | Bassa | Hardware | Pulsante di spegnimento su GPIO3 in conflitto con I2C abilitato — **non si corregge** |
| [B08](#b08) | Media | USB | Montaggio chiavette: solo partizioni `sdX[0-9]`, un solo device alla volta |
| [B09](#b09) | ~~Media~~ ✅ | Pacchetti | `initramfs-tools` resta in hold per sempre sull'immagine finale — **risolto (hold rimosso)** |
| [B10](#b10) | Bassa | udev | Virgola mancante nelle regole udev Wi-Fi powersave e CAN |
| [B11](#b11) | ~~Bassa~~ ✅ | Swap | Il resize dello swap non viene mai eseguito — **risolto da upstream** |
| [B12](#b12) | Bassa | Klipper | Pulizia `c_helper.so` corrotto solo se di dimensione 0 al runtime |
| [B13](#b13) | Bassa | Wi-Fi | `headless_nm`: password < 8 caratteri lascia una connessione rotta |
| [B14](#b14) | Bassa | CI | `CustoPiZer@main` non bloccato |
| [B15](#b15) | Bassa | Varie | Residui MainsailOS (link, patch, branding, variabili inesistenti) |
| [B16](#b16) | Media | Sicurezza | Credenziali di default `pi`/`raspberry` con SSH attivo |
| [B17](#b17) | Media | Build | Immagine base `raspios_lite_arm64_latest` non bloccata |
| [B18](#b18) | Media | Trixie | Componenti G1 non ancora verificati su Trixie (power button, Obico, KlipperScreen fork) |

---

### B01
**Repo esterni non bloccati a versione** — **Non è un bug (decisione 2026-10-05)**

> L'immagine viene buildata dalla pipeline solo ai rilasci: prendere `HEAD` dei repo esterni al momento del rilascio è il comportamento voluto. Resta valido il consiglio di debug: per regressioni tra due versioni dell'immagine, confrontare prima i commit dei repo esterni. Descrizione originale:

Tutti i `git clone` dei moduli `5x`/`6x` prendono `HEAD` del branch di default (Mainsail: `releases/latest`). Rebuildare lo stesso commit G1OS in giorni diversi produce immagini diverse; un commit rotto su klipper4pellet / klipperscreen4pellet / G1-Configs / Moonraker finisce direttamente nell'immagine.

- Dove: `modules/generic/50-klipper4pallet`, `51-moonraker`, `52-mainsail`, `53-crowsnest`, `54-timelapse`, `55-sonar`, `56-klipperscreen4pellet`, `57-Kiauh`, `58-Obico`, `60-Kamp`, `62-PowerButton`, `69-G1Config`.
- Conferma: confronta `git -C ~/<repo> log -1` su due unità flashate da build diverse.
- Fix: aggiungere `-b <tag>` / `git checkout <sha>` almeno per i repo Ginger (klipper4pellet, klipperscreen4pellet, G1-Configs), magari definendo le versioni in `00-config`. Per bug di regressione tra due versioni dell'immagine, **prima** di cercare nel codice G1OS confrontare i commit dei repo esterni.

### B02
**`update_manager KlipperScreen` punta al repo sbagliato** — Alta

`modules/generic/files/moonraker_klipperscreen4pellet.conf:8` → `origin: https://github.com/KlipperScreen/KlipperScreen.git`, ma `~/KlipperScreen` è un clone di `gingeradditive/klipperscreen4pellet`. Moonraker confronta `origin` con il remote reale: il repo viene segnato come **invalid** (aggiornamenti bloccati) oppure, dopo un "recover", verrebbe riallineato all'upstream perdendo le modifiche pellet.

- Conferma: in Mainsail → Machine → Update Manager, KlipperScreen appare "invalid"/"dirty"; `grep -A3 "KlipperScreen" ~/printer_data/logs/moonraker.log`.
- Fix: `origin: https://github.com/gingeradditive/klipperscreen4pellet.git` (+ `primary_branch` corretto). Verificare anche che `scripts/system-dependencies.json` esista nel fork.
- Nota: i commenti nello stesso file ("Uncomment to enable") non corrispondono, la sezione è già attiva.

### B03
**File di config creati da root** — Alta (da confermare)

- `modules/generic/51-moonraker:68` → `cp /files/moonraker.conf …/config/moonraker.conf` eseguito da root ⇒ owner `root:root`.
- `modules/generic/60-Kamp:42` → `cp …/KAMP_Settings.cfg …/config/` da root.

Moonraker gira come `pi`: Mainsail può leggere ma non salvare questi file (errore di salvataggio nell'editor). A meno che `G1-Configs/install.sh` (69) non faccia un `chown -R` su `printer_data` — da verificare in quel repo.

- Conferma: `ls -l ~/printer_data/config/` su un dispositivo appena flashato.
- Fix: `chown "${BASE_USER}:${BASE_USER}"` dopo i `cp`, oppure `sudo -u "${BASE_USER}" cp …`.

### B04
**Path `/home/pi` non gestiti da `postrename`** — Media

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
**Montaggio chiavette USB limitato** — Media

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
**Virgole mancanti nelle regole udev** — Bassa

- `modules/generic/files/070-wifi-powersave.rules:3` `KERNEL=="wlan*" \` → manca `,` prima di `RUN+=`.
- `modules/generic/files/canbus/10-can.rules:1` `KERNEL=="can*"  ATTR{…}` → manca `,`.

udev tollera la virgola mancante ma logga un warning; versioni future potrebbero scartare la regola. Su Trixie (systemd 257) è disponibile `udevadm verify`.
- Conferma: `udevadm verify /etc/udev/rules.d/*.rules`; `iw wlan0 get power_save` deve dire `off`.

### B11
**Resize swap mai eseguito** — ✅ Risolto da upstream (`426a216`): usa `${SWAP_CONF_FILE}` su Bookworm e un drop-in `rpi-swap` su Trixie.

Descrizione originale:

`modules/raspberry/10-config-raspberry:73` testa `${PICONFIG_SWAP_CONF_FILE}` (non definita) invece di `${SWAP_CONF_FILE}` ⇒ il blocco non viene mai eseguito, lo swap resta al default (100 MB). Correggere solo dopo aver valutato l'impatto sulla SD.

### B12
**Pulizia `c_helper.so`** — Bassa

Il fix `ca4b0dd` rimuove `c_helper.so` in build e in `klipper.service` (`ExecStartPre`) cancella il file **solo se vuoto** (`-size 0`). Un file non vuoto ma corrotto (scrittura interrotta da spegnimento durante la prima compilazione) non viene rimosso ⇒ Klipper non parte ("can't connect to klipper").
- Workaround sul campo: `rm ~/klipper/klippy/chelper/c_helper.so && sudo systemctl restart klipper`.
- Nota: `klipper.service` ha `After=network-online.target` senza `Wants=network-online.target` (l'ordinamento non ha effetto).

### B13
**`headless_nm` con password non valida** — Bassa

`files/headless-nm/headless_nm:138`: con password < 8 o > 63 caratteri `wpa_passphrase` fallisce; con `set -e` lo script esce dopo aver già scritto `preconfigured.nmconnection` senza sezione di sicurezza e senza `chmod 0600` (`:212`) ⇒ NetworkManager ignora il file, e `headless_nm.txt` **non** viene cancellato. Anche `HIDDEN` vuoto produce `hidden=` (non valido).
- Conferma: `journalctl -t headless_nm`.

### B14
**CustoPiZer non bloccato** — Bassa

`.github/workflows/build.yml:139` e `release.yml:201` usano `OctoPrint/CustoPiZer@main`: una modifica upstream può rompere la build senza cambi in questo repo. Fissare a un SHA.

### B15
**Residui MainsailOS** — Bassa

- `patches/*.sh`: URL `mainsail-crew/G1OS` inesistenti, pensati per Bullseye. Da rimuovere o riscrivere.
- `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md` (link `mainsail-crew/MainsailOS`), issue template, `rpi_json.icon` (`os.mainsail.xyz`): branding non G1.
- Il README cita CustomPiOS, ma la build usa CustoPiZer.
- Nome file `50-klipper4pallet` (typo "pallet").
- `klipper.service` Description "SV1".
- `config.yml` dichiara in `rpi_json.devices` anche `pi3-64bit` e `pi5-64bit`, ma il target supportato è solo Raspberry Pi 4.

### B16
**Credenziali di default con SSH attivo** — Media (scelta consapevole, 2026-10-05)

Da `426a216` (Trixie non ha più un utente preconfigurato): `BASE_PASSWORD=raspberry` in `00-config`, impostata da `10-config-raspberry` con `userconf`; `userconfig.service` disabilitato; SSH abilitato. Ogni stampante esce con `pi`/`raspberry` raggiungibile in SSH, a meno che l'utente non imposti credenziali in Raspberry Pi Imager (cloud-init).
- Mitigazione possibile: forzare il cambio password al primo login (`chage -d 0 pi`) o impostare una password diversa in `BASE_PASSWORD`.

### B17
**Immagine base non bloccata** — Media

`config.yml` usa `https://downloads.raspberrypi.org/raspios_lite_arm64_latest.torrent` (+ `.sha256`). Ogni nuova release di Raspberry Pi OS cambia la base senza commit in questo repo (stesso problema di [B01](#b01)); se torrent e sha256 vengono aggiornati in momenti diversi, la verifica fallisce.
- Fix: usare l'URL datato `…/images/raspios_lite_arm64-AAAA-MM-GG/…` come si faceva per Bookworm.

### B18
**Componenti G1 da verificare su Trixie** — Media

Verificato in modo statico (2026-10-05): tutti i pacchetti apt dei moduli G1 esistono in Trixie; i requirements di klipper4pellet hanno già pin per Python ≥ 3.12; le dipendenze di klipperscreen4pellet esistono. Restano da verificare sul Pi 4:
- `62-PowerButton`: `listen-for-shutdown.py` usa `RPi.GPIO`, che il modulo non installa (su Trixie `python3-rpi.gpio` esiste ma potrebbe non essere preinstallato); l'installer usa `update-rc.d`/init.d (SysV, deprecato in systemd 257) e lancia il daemon dentro il chroot.
- `58-Obico`: installer esterno, compatibilità con Python 3.13 non verificata.
- `56-klipperscreen4pellet`: requirements con `PyGObject<3.51` (build da sorgente su Python 3.13).
- `G1-Configs/install.sh`: `sudo pip3 install flask` fallisce per PEP 668 (innocuo: Flask arriva da apt in `69-G1Config`).
- Primo boot con utente personalizzato in Raspberry Pi Imager (flusso cloud-init) e senza personalizzazione.
