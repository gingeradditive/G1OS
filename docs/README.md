# Documentazione tecnica G1OS

Documentazione interna per capire come è costruita l'immagine G1OS e per individuare/correggere bug.

| Documento | Contenuto |
|---|---|
| [architecture.md](architecture.md) | Pipeline di build (GitHub Actions → CustoPiZer → moduli), ordine di esecuzione, layout del sistema sul dispositivo (utenti, path, servizi, porte) |
| [modules.md](modules.md) | Riferimento modulo per modulo: cosa fa, file che installa, dipendenze esterne |
| [known-issues.md](known-issues.md) | Catalogo dei bug e dei punti fragili trovati nell'analisi del codice, con file/riga e fix proposto |
| [debugging.md](debugging.md) | Come riprodurre e diagnosticare problemi: sul dispositivo, nella build CI, in locale |

## In due righe

G1OS è un fork di [MainsailOS](https://github.com/mainsail-crew/MainsailOS) (remote `upstream`). Il repository **non contiene il codice applicativo** (Klipper, Moonraker, KlipperScreen, G1-Configs…): contiene solo gli **script bash che, in fase di build, vengono eseguiti in chroot dentro l'immagine Raspberry Pi OS** per clonare/installare quei progetti e configurare il sistema. Il prodotto finale è un file `.img.xz` da flashare su SD.

Quindi un bug "di G1OS" ricade quasi sempre in una di queste categorie:

1. **Script di build sbagliato** → l'immagine nasce con file/permessi/servizi errati (vedi `modules/`).
2. **File di configurazione statico sbagliato** (`modules/*/files/`) → servizio configurato male.
3. **Script di runtime** che girano sul dispositivo (`postrename`, `headless_nm`, unit systemd, regole udev).
4. **Dipendenza esterna non bloccata a una versione**: ogni build clona il `HEAD` dei repo esterni, quindi due build dello stesso commit G1OS possono produrre immagini diverse.
5. **Bug nel progetto esterno** (klipper4pellet, klipperscreen4pellet, G1-Configs…) → va corretto in quel repo, non qui.

## Stato del repository (al 2026-10-05)

- Versione: `2.0.10` (file `VERSION`), branch principale `develop`.
- Unico target attivo in `config.yml`: **Raspberry Pi OS Bookworm Lite arm64**. L'hardware supportato è **solo Raspberry Pi 4** (`config.yml` elenca ancora anche Pi 3 e Pi 5 nei metadati per RPi Imager). I target armhf e Orange Pi/Armbian sono commentati: i moduli `modules/armbian/` e `modules/special/` esistono ma **non vengono usati** nella build attuale.
- Il README e `CONTRIBUTING.md` sono ancora in gran parte quelli di MainsailOS (link, Discord, menzione di CustomPiOS: in realtà la build usa **CustoPiZer** dal 2025-05, commit `26eee8e`).
