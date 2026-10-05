# Debugging

## 1. Primo passo: capire dove sta il bug

| Sintomo | Dove guardare prima |
|---|---|
| La build CI fallisce | Log della job `build` → gruppo `Running /…/scripts/<modulo> in chroot` (ogni modulo è un gruppo collassabile; con `set -x` si vede il comando fallito) |
| Problema presente su tutte le unità appena flashate | Moduli in `modules/` e file in `modules/*/files/` |
| Problema comparso tra due versioni dell'immagine senza cambi in G1OS | Repo esterni non bloccati ([known-issues B01](known-issues.md#b01)): confronta i commit |
| Solo con utente diverso da `pi` | `postrename` ([B04](known-issues.md#b04), [B05](known-issues.md#b05)) |
| Wi-Fi non si configura | `headless_nm` |
| Klipper / KlipperScreen / G1-Configs si comportano male a runtime | Quasi sempre bug nel repo esterno (klipper4pellet, klipperscreen4pellet, G1-Configs) |

## 2. Sul dispositivo

Accesso: `ssh pi@g1os.local` (o IP dal router).

```bash
# Versione immagine
cat /etc/g1os-release

# Stato servizi
systemctl --failed
systemctl status klipper moonraker nginx crowsnest sonar KlipperScreen moonraker-obico headless_nm splashscreen

# Log applicativi
tail -n 200 ~/printer_data/logs/klippy.log
tail -n 200 ~/printer_data/logs/moonraker.log
journalctl -u klipper -u moonraker -b --no-pager | tail -n 200

# Commit effettivi dei repo esterni inclusi nell'immagine
for d in klipper moonraker KlipperScreen G1-Configs mainsail-config crowsnest sonar \
         moonraker-timelapse moonraker-obico Klipper-Adaptive-Meshing-Purging kiauh pi-power-button; do
  printf '%-36s ' "$d"; git -C ~/$d log -1 --format='%h %ad %s' --date=short 2>/dev/null || echo '-'
done

# Permessi config (B03)
ls -l ~/printer_data/config/

# Pacchetti in hold (B09)
apt-mark showhold

# Seriale MCU (UART PL011)
ls -l /dev/serial0 /dev/ttyAMA0 /dev/serial/by-id/ 2>/dev/null
grep -E 'enable_uart|disable-bt|i2c|spi' /boot/firmware/config.txt
cat /boot/firmware/cmdline.txt

# Regole udev (B10)
udevadm verify /etc/udev/rules.d/*.rules   # disponibile da systemd 253 (Trixie ha la 257)
iw dev wlan0 get power_save

# Wi-Fi headless (B13)
journalctl -t headless_nm --no-pager
ls -l /etc/NetworkManager/system-connections/

# Chiavette USB (B08)
journalctl -u 'usbstick-handler@*' -b --no-pager; mount | grep /media

# Rename utente (B04/B05) - cloud-init
systemctl status mainsailos-prerename mainsailos-postrename
cat /var/log/mainsailos-postrename.log; ls /var/lib/mainsailos/
cloud-init status --long
grep -rl '/home/pi' /etc/systemd/system ~/printer_data 2>/dev/null
```

Klipper non si connette ("can't connect to klipper"): vedi [B12](known-issues.md#b12), poi `klippy.log`. Verificare che `~/printer_data/config/printer.cfg` esista (lo fornisce G1-Configs, non questo repo).

## 3. Nella build

- Ogni push su `develop` e ogni PR che tocca `modules/**`, `config.yml` o `build.yml` lancia `build.yml`; si può lanciare anche a mano (*Actions → Build Images → Run workflow*) su un branch.
- L'immagine prodotta è scaricabile come artifact della run (`<data>-G1OS-<target>-<versione>.img-artifacts`).
- L'immagine base è `raspios_lite_arm64_latest`: se una build si rompe senza cambi nel repo, verificare se Raspberry Pi ha pubblicato una nuova immagine.
- Tempi: download torrent + build arm64 ≈ 1 ora; per iterare velocemente su un modulo conviene prima provarlo a mano su un Pi già flashato (vedi §4).

Errori tipici in build:

| Errore | Causa probabile |
|---|---|
| `E: Unable to locate package …` | pacchetto rinominato/rimosso in Debian; i moduli non usano versioni fisse |
| fallimento in `install.sh` / `make install` di un repo esterno | cambiamento upstream (B01); l'installer richiede interattività o avvia servizi |
| `sha256sum: WARNING: 1 computed checksum did NOT match` | l'URL in `config.yml` è stato aggiornato senza aggiornare l'hash o viceversa |
| `No space left on device` | aumentare `IMAGE_ENLARGEROOT` in `config.yml` |
| comandi `systemctl start` / servizi che "non partono" | normale in chroot: `policy-rc.d` blocca l'avvio; usare solo `enable` |

## 4. Provare una modifica senza rifare l'immagine

Gli script dei moduli si possono adattare per girare su un Pi già flashato (come root), purché si sostituiscano le dipendenze da CustoPiZer:

```bash
# sul Pi, dalla root del repo copiato
sudo mkdir -p /files && sudo cp -r modules/generic/files/* modules/raspberry/files/* /files/
cat <<'EOF' | sudo tee /common.sh
install_cleanup_trap() { :; }
systemctl_if_exists() { systemctl "$@"; }
echo_green() { echo "$@"; }
EOF
sudo BOOT_PATH=/boot/firmware bash modules/generic/<modulo>
```

Attenzione: molti moduli **non sono idempotenti** (`git clone` in una directory esistente fallisce, `ln -s` su un link esistente fallisce, `cat >> moonraker.conf` duplica le sezioni). Lavorare su una SD di test.

## 5. Build locale (Linux, Docker)

CustoPiZer è disponibile come immagine Docker (vedi README di <https://github.com/OctoPrint/CustoPiZer>). Riprodurre i passi 3-7 di `build.yml`:

```bash
mkdir -p build scripts
# mettere l'immagine base decompressa in build/input.img
cp -r modules/generic/* modules/raspberry/* scripts/ && rm -f scripts/*.disabled
printf 'EDITBASE_ARCH=arm64\nEDITBASE_IMAGE_ENLARGEROOT=7000\nEDITBASE_MOUNT_PROC=1\nEDITBASE_MOUNT_SYS=1\n' > config.local
docker run --rm --privileged \
  -v "$PWD/build":/CustoPiZer/workspace \
  -v "$PWD/scripts":/CustoPiZer/workspace/scripts \
  -v "$PWD/config.local":/CustoPiZer/config.local \
  -e RELEASE_TAG="$(cat VERSION)" \
  ghcr.io/octoprint/custopizer:latest
```

Su host x86 serve `qemu-user-static`/binfmt per eseguire il chroot arm64 (molto più lento del runner `ubuntu-22.04-arm`). Su macOS conviene usare la CI.

## 6. Checklist per una PR di fix

1. Branch da `develop` aggiornato (`git pull`: le release aggiungono commit automatici).
2. Commit in stile Conventional Commits (`fix(modulo): …`): `git-cliff` li usa per il CHANGELOG.
3. Se il fix riguarda un modulo, verificare che resti compatibile con `set -xe` (nessun comando che può fallire senza `|| true`).
4. Se il fix riguarda un path utente, verificare anche il caso utente rinominato (`postrename`).
5. Lasciare girare `build.yml` sulla PR e testare l'artifact su hardware reale.
6. Aggiornare [known-issues.md](known-issues.md) (rimuovere/aggiornare la voce).
