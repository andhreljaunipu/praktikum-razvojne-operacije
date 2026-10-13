# Tjedan 1: Osnovno o razvoju aplikacijskog softvera i razvojnih operacija

Termin: 180 min = blok A (80 min) + pauza (20 min) + blok B (80 min).
Ishodi učenja koje tjedan pokriva: 1 (prepoznati koncepte i alate), 2 (razlikovati okruženja, uvodno).

| Datoteka | Sadržaj |
|---|---|
| [blok-a-predavanje.md](blok-a-predavanje.md) | Uvod u kolegij, SDLC, DevOps, CI, hardver i produkcijski sustavi |
| [blok-b-vjezba.md](blok-b-vjezba.md) | Vježba: WSL, git, gh CLI, Python + venv, prvi push |
| [check_env.sh](check_env.sh) | Skripta kojom student i nositelj provjeravaju je li okruženje ispravno |
| [primjer-release-please/](primjer-release-please/) | Primjer workflowa i datoteka za release-please (kopirati u korijen repozitorija) |

## GitHub konfiguracija
- Poslati studentima s Windowsom uputu da uključe virtualizaciju u BIOS-u i, ako mogu, unaprijed pokrenu `wsl --install` (preuzimanje traje).
- U postavkama organizacije (Settings → Actions → General) dopustiti GitHub Actionsu stvaranje pull requestova, inače release-please ne može otvoriti release PR.
- Naziv repozitorija: `praktikum-<github-korisnik>`.

## Predaja (praktični zadatak 1)
Osobni repozitorij `praktikum-<github-korisnik>` u organizaciji, s `README.md`, `.gitignore`, `requirements.txt`, `check_env.sh`, izlazom provjere (`check_env.txt`), te `.github/workflows/release_please.yml` i `VERSION.txt`. Na kraju bloka postoji izdanje `v0.1.0`.
