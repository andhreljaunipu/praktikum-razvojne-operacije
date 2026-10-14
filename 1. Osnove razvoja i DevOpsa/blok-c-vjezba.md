# Blok C (80 min): Osnove Linuxa

Cilj: student se samostalno kreće datotečnim sustavom iz terminala, upravlja datotekama i procesima, instalira pakete i spaja se na udaljeni poslužitelj (GCP VM kolegija) preko `ssh` i `sftp`. Sve naredbe izvode se u WSL-u (Ubuntu) iz bloka B.

| Min | Korak |
|---|---|
| 0–12 | 1. Orijentacija: `pwd`, `ls`, `cd` |
| 12–30 | 2. Rad s datotekama: `mkdir`, `cp`, `mv`, `rm`, `rmdir` |
| 30–42 | 3. Procesi: `htop` umjesto Task Managera |
| 42–57 | 4. Upravitelji paketa: `apt` i razlike po distribucijama |
| 57–72 | 5. Udaljeni pristup: `ssh` |
| 72–80 | 6. Prijenos datoteka: `sftp` |

> macOS: sve osim `apt` radi jednako (paketi preko `brew`). Na GCP VM-u kolegija vrijedi sve iz ovog bloka, uključujući `apt`.

## 1. Orijentacija: `pwd`, `ls`, `cd`
Linux nema `C:\`, `D:\`: sve kreće od jednog korijena `/`. Bitni direktoriji:

| Putanja | Sadržaj | Windows analogija |
|---|---|---|
| `/` | korijen datotečnog sustava | `C:\` |
| `/home/<korisnik>` (`~`) | osobni direktorij | `C:\Users\<korisnik>` |
| `/etc` | konfiguracija sustava | registry / `C:\ProgramData` |
| `/usr/bin` | instalirani programi | `C:\Program Files` |
| `/var/log` | logovi | Event Viewer |
| `/tmp` | privremene datoteke | `%TEMP%` |
| `/mnt/c` | Windows disk C: (samo WSL) | |

```bash
pwd                  # print working directory: where am I?
ls                   # list files in the current directory
ls -l                # long format: permissions, owner, size, date
ls -la               # also hidden files (names starting with a dot)
ls -lh /var/log      # human-readable sizes, another directory
cd /etc              # absolute path (starts with /)
cd ..                # one level up
cd -                 # back to the previous directory
cd                   # no argument: go home (same as cd ~)
```
Objasniti: apsolutna (`/home/ana/projekt`) i relativna putanja (`projekt`, `../drugi`), `.` je trenutni direktorij, `..` roditelj. Tab dovršava imena, strelica gore vraća prethodne naredbe, `Ctrl+R` pretražuje povijest.

Čitanje `ls -l`:
```
-rw-r--r-- 1 ana ana 1432 Oct 14 10:00 README.md
drwxr-xr-x 2 ana ana 4096 Oct 14 10:00 docs
```
Prvi znak: `-` datoteka, `d` direktorij. Zatim tri trojke `rwx` (vlasnik, grupa, ostali). Detaljnije (`chmod`, `chown`) kad budemo pisali skripte za VM.

## 2. Rad s datotekama: `mkdir`, `cp`, `mv`, `rm`, `rmdir`
Sve vježbamo u zasebnom direktoriju da ništa ne pokvarimo:
```bash
mkdir -p ~/linux-lab/notes      # -p creates parent directories as needed
cd ~/linux-lab
echo "first line" > notes/todo.txt
echo "second line" >> notes/todo.txt   # >> appends, > overwrites
cat notes/todo.txt
```
Kopiranje i premještanje:
```bash
cp notes/todo.txt todo-backup.txt      # copy a file
cp -r notes notes-backup               # copy a directory (-r = recursive)
mv todo-backup.txt todo-old.txt        # rename (mv within the same directory)
mkdir archive
mv todo-old.txt archive/               # move into a directory
cp -i notes/todo.txt archive/todo-old.txt   # -i asks before overwriting
ls -R                                  # recursive listing
```
Brisanje:
```bash
rm archive/todo-old.txt     # delete a file
rmdir archive               # delete a directory, works ONLY if it is empty
rmdir notes-backup          # fails: "Directory not empty"
rm -r notes-backup          # delete a directory with its contents
rm -i notes/todo.txt        # ask before deleting
```
Upozoriti: Linux **nema koša za smeće** u terminalu. `rm` briše odmah i trajno, a `rm -rf` ne pita ništa. Prije `rm -rf` uvijek `pwd` i `ls`. `sudo rm -rf /` (ili `rm -rf $DIR/` s praznom varijablom) je klasična katastrofa; zato u skriptama koristimo `set -u`.

Kratki zadatak (5 min): u `~/linux-lab` napraviti strukturu ispod, samo naredbama iz ovog koraka, pa prikazati `ls -R`:
```
linux-lab/
├── src/
│   └── main.py
├── docs/
│   └── README.md        (copy of src/main.py renamed)
└── old/                 (created, then removed with rmdir)
```

## 3. Procesi: `htop` umjesto Task Managera
`htop` je terminalska inačica Windows Task Managera: popis procesa, opterećenje CPU-a i memorije, gašenje procesa. Na poslužitelju bez grafičkog sučelja to je glavni alat za pogled "zašto je stroj spor".
```bash
sudo apt install -y htop
htop
```

| Task Manager | htop |
|---|---|
| kartica Processes | glavni popis procesa |
| kartica Performance (CPU, Memory) | trake na vrhu (svaka jezgra zasebno, `Mem`, `Swp`) |
| sortiranje klikom na stupac | `F6` ili klik na zaglavlje (CPU%, MEM%) |
| End task | označiti proces, `F9`, zatim `SIGTERM` (15) ili `SIGKILL` (9) |
| pretraga | `F3` traži, `F4` filtrira |
| | `F5` stablo procesa (tko je koga pokrenuo), `q` izlaz |

Demonstracija: u drugom terminalu namjerno opteretiti jednu jezgru, pa je pronaći i ugasiti u `htop`-u:
```bash
yes > /dev/null &        # endless loop at 100% of one CPU core, & runs it in the background
```
Isto bez `htop`-a (korisno u skriptama i na minimalnim sustavima):
```bash
ps aux | grep yes        # find the process and its PID
kill <PID>               # polite stop (SIGTERM)
kill -9 <PID>            # force stop (SIGKILL), last resort
top                      # older tool, always installed
free -h                  # memory usage
df -h                    # disk usage
```
Objasniti: PID, `SIGTERM` (proces se može uredno zatvoriti) naspram `SIGKILL` (OS ga ubija odmah). `Load average` u `htop`-u: broj procesa koji čekaju CPU; trajno iznad broja jezgri znači preopterećenje (veza na 6. tjedan, hardver).

## 4. Upravitelji paketa: `apt` i razlike po distribucijama
Na Linuxu se softver ne preuzima s web stranica kao `.exe`, nego iz **repozitorija paketa** distribucije. Upravitelj paketa preuzima paket, provjerava potpis, instalira ovisnosti i kasnije ga ažurira.
```bash
sudo apt update                  # refresh the package index (does not install anything)
apt list --upgradable            # what can be upgraded
sudo apt upgrade -y              # upgrade installed packages
apt search tree                  # search
apt show tree                    # package details
sudo apt install -y tree         # install
tree ~/linux-lab
which tree                       # where the binary was installed
sudo apt remove -y tree          # remove (keeps config files)
sudo apt purge -y tree           # remove together with config files
sudo apt autoremove -y           # remove dependencies nobody needs any more
```
`apt` je za ljude (ljepši izlaz, trake napretka), `apt-get` za skripte (stabilan izlaz). U Dockerfileu i CI-ju pišemo `apt-get install -y`. Vlastiti repozitorij (kao `gh` u bloku B) dodaje se u `/etc/apt/sources.list.d/` zajedno s ključem za potpis.

Različite distribucije, različiti upravitelji, isti koncept:

| Distribucija | Format paketa | Upravitelj | Instalacija | Osvježavanje indeksa |
|---|---|---|---|---|
| Debian, Ubuntu (WSL, GCP VM) | `.deb` | `apt` (`dpkg` ispod) | `apt install htop` | `apt update` |
| Fedora, RHEL, Rocky, CentOS | `.rpm` | `dnf` (stariji `yum`) | `dnf install htop` | `dnf check-update` |
| Arch, Manjaro | `.pkg.tar.zst` | `pacman` | `pacman -S htop` | `pacman -Sy` |
| Alpine (česta osnova Docker slika) | `.apk` | `apk` | `apk add htop` | `apk update` |
| openSUSE | `.rpm` | `zypper` | `zypper install htop` | `zypper refresh` |
| macOS | | `brew` | `brew install htop` | `brew update` |
| Windows | | `winget` | `winget install <id>` | |

Kako saznati na kojoj smo distribuciji:
```bash
cat /etc/os-release
uname -r                 # kernel version (in WSL it contains "microsoft")
```
Razlike koje se vide u praksi: imena paketa nisu ista (`python3-venv` na Ubuntuu, na Fedori je venv već dio `python3`; `build-essential` na Ubuntuu je `@development-tools` na Fedori), verzije se razlikuju (Debian stabilan i stariji, Arch najnoviji), a Alpine koristi `musl` umjesto `glibc` pa neki Python paketi traže kompilaciju. Zato se u 4. tjednu za Docker sliku bira osnovna distribucija svjesno.

## 5. Udaljeni pristup: `ssh`
Na GCP VM kolegija (diže ga nositelj) spajamo se ključem, ne lozinkom. Par ključeva: privatni ostaje kod studenta, javni ide na poslužitelj.
```bash
ssh-keygen -t ed25519 -C "<github-korisnik>@praktikum"   # Enter for the default path, set a passphrase
ls -l ~/.ssh                       # id_ed25519 (private, never share) and id_ed25519.pub (public)
cat ~/.ssh/id_ed25519.pub          # send this line to the instructor
```
Kad nositelj doda javni ključ na VM (u GCP-u: metadata projekta ili VM-a, *SSH Keys*):
```bash
ssh <korisnik>@<vm-ip>             # first time: confirm the host fingerprint with "yes"
```
Na VM-u isprobati sve iz koraka 1 do 4 (`pwd`, `ls /home` pokazuje kolege, `htop` pokazuje sve korisnike, `cat /etc/os-release`). Izlaz: `exit` ili `Ctrl+D`.

Da se adresa ne piše svaki put, `~/.ssh/config`:
```bash
cat >> ~/.ssh/config <<'CFG'
Host praktikum-vm
    HostName <vm-ip>
    User <korisnik>
    IdentityFile ~/.ssh/id_ed25519
CFG
chmod 600 ~/.ssh/config
ssh praktikum-vm
ssh praktikum-vm "uptime && df -h /"    # run a single command remotely and come back
```
DevSecOps trenutak: privatni ključ nikad u git, chat ili e-mail; ako procuri, javni ključ se briše s poslužitelja i generira novi par. Prava moraju biti stroga (`chmod 700 ~/.ssh`, `chmod 600 ~/.ssh/id_ed25519`), inače `ssh` odbija ključ.

## 6. Prijenos datoteka: `sftp`
`sftp` koristi istu SSH vezu i iste ključeve, pa ne treba dodatna konfiguracija.
```bash
cd ~/linux-lab
sftp praktikum-vm
```
U `sftp>` sesiji naredbe s `l` odnose se na lokalno računalo:
```
sftp> pwd                  # remote directory
sftp> lpwd                 # local directory
sftp> ls                   # remote listing
sftp> lls                  # local listing
sftp> mkdir linux-lab
sftp> cd linux-lab
sftp> put src/main.py      # upload: local -> remote
sftp> put -r docs          # upload a whole directory
sftp> get /etc/os-release  # download: remote -> local
sftp> bye
```
Provjera:
```bash
ls -l os-release
ssh praktikum-vm "ls -R ~/linux-lab"
```
Za jednu datoteku u skripti zgodniji je `scp` (`scp src/main.py praktikum-vm:~/linux-lab/`), a za sinkronizaciju direktorija `rsync`. Kasnije (8. i 11. tjedan) studenti na isti način na VM šalju skripte koje im postavljaju razvojno okruženje.

## Česti problemi
| Simptom | Rješenje |
|---|---|
| `rmdir: failed to remove 'x': Directory not empty` | Direktorij nije prazan; `ls -la x` (skrivene datoteke) ili `rm -r x` |
| `Permission denied` pri `apt install` | Nedostaje `sudo` |
| `E: Unable to locate package ...` | Prvo `sudo apt update`; provjeriti ime paketa za tu distribuciju |
| `Could not get lock /var/lib/dpkg/lock-frontend` | Drugi `apt` već radi (često automatsko ažuriranje); pričekati, ne brisati lock |
| `htop: command not found` | `sudo apt install -y htop` |
| `Permission denied (publickey)` | Javni ključ nije dodan na VM, krivi korisnik ili prava na `~/.ssh` preširoka |
| `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED` | VM je ponovno stvoren; provjeriti s nositeljem, pa `ssh-keygen -R <vm-ip>` |
| `ssh: connect to host ... port 22: Connection timed out` | VM ugašen ili GCP firewall ne dopušta port 22 s te adrese |
| `UNPROTECTED PRIVATE KEY FILE` | `chmod 600 ~/.ssh/id_ed25519` |
