# Arenduskeskkonna dokumentatsioon

**Kuupäev:** 2026-10-30  
**Host:** `qortium`  
**Kasutaja:** `iffi`  
**OS:** Debian GNU/Linux 13 (trixie) 13.7, kernel `6.12.113+deb13-cloud-amd64`, `x86_64`  
**Tööruum:** `/home/iffi/dev-workspace`

Käesolev fail dokumenteerib üldise arenduskeskkonna, mis valmistati ette tulevaste
Qortiumi/Qortal projektide jaoks. Qortiumi faile, seadistusi, automaatuuendust,
käivitusskripte, SSH-d, swap'i ega teenuseid ei muudetud.

---

## 1. Paigaldusmeetod ja piirang

Sudo nõudis parooli, mida sessioonis ei olnud. Kokkuleppel kasutajaga paigaldati
kõik tööriistad **kasutajaruumi** (`/home/iffi/...`) ilma `sudo`-ta.

Seetõttu:

- Debiani pakette `python3-pip`, `python3-venv` ja `python-is-python3` **ei
  paigaldatud** (`apt` vajas sudo õigusi).
- Süsteemset Pythonit (`/usr/bin/python3`) ei muudetud. Endiselt kehtib:
  `python3 -m pip` → `No module named pip`.
- Pip ja venv on tehtud kättesaadavaks isoleeritud baasvirtuaalkeskkonna kaudu,
  mis ei puuduta Debiani `externally-managed` süsteemset Pythonit.
- Kui sudo õigused hiljem antakse, võib puuduolevad Debiani paketid paigaldada:
  `sudo apt install python3-pip python3-venv python-is-python3`.

---

## 2. Node.js ja npm

| Tööriist | Versioon | Asukoht |
|---|---|---|
| Node.js | **v24.22.2** (LTS, koodnimi *Krypton*) | `/home/iffi/dev-workspace/tools/node-v24.22.2-linux-x64/bin/node` |
| npm | **11.19.1** | `.../tools/node/bin/npm` |
| npx | 11.19.1 | `.../tools/node/bin/npx` |
| corepack | 0.36.0 | `.../tools/node/bin/corepack` |

**Allikas:** ametlik `https://nodejs.org/dist/v24.22.2/node-v24.22.2-linux-x64.tar.xz`  
**Terviklikkus:** SHA256 `48cc0e21e048790e54c0cf5749c2725005313d652651c40b7a21feb9697aaf75`,
kontrollitud ametliku `SHASUMS256.txt` vastu (`OK`).

**Valiku põhjendus:** Node.js 24 on LTS haru, mida toetatakse kuni 2028-04.
Debian 13 enda `nodejs` pakett on 20.19.2 (Node 20 on juba EOL) ja `npm` 9.2.0,
seega need ei sobiks ajakohaseks arenduskeskkonnaks. Node.js 26 on samuti LTS,
kuid 24 on küpsem ja laiema moodulite ühilduvusega.

**Sümlingid:** `/home/iffi/.local/bin/{node,npm,npx,corepack}` →
`/home/iffi/dev-workspace/tools/node/bin/*`  
PATH-i lisab ühtselt `scripts/env.sh`, mille laadivad nii `~/.profile`
(login-shellid, sh ka mitteinteraktiivsed) kui `~/.bashrc` (interaktiivsed
shellid). Seetõttu on Node.js, npm, Python ja OpenCode leitavad ka tmux-paneedes.

npm registri kontroll: `https://registry.npmjs.org/` vastab, `npm view npm version`
→ `12.2.1` (kataloogis uusim npm; paigaldatud npm on Node 24-ga kaasas tulev 11.19.1).

---

## 3. Python, pip ja venv

| Tööriist | Versioon | Asukoht / kasutus |
|---|---|---|
| Python 3 | **3.13.5** | `/usr/bin/python3` (süsteemne, muutmata) |
| `python` alias | 3.13.5 | `/home/iffi/.local/bin/python` → `/usr/bin/python3` |
| pip | **26.2.2** | `/home/iffi/dev-workspace/tools/python-base/bin/pip` |
| pip (käsud) | 26.2.2 | `/home/iffi/.local/bin/{pip,pip3}` → baas-venv pip |
| virtualenv | **21.15.1** | `/home/iffi/dev-workspace/tools/python-base/bin/virtualenv` |
| `mkvenv` | — | `/home/iffi/.local/bin/mkvenv` → virtualenv wrapper |

**Baasvirtuaalkeskkond:** `/home/iffi/dev-workspace/tools/python-base`
(loodud `python3 -m venv --without-pip`, pip bootstrap'itud ametliku
`https://bootstrap.pypa.io/get-pip.py` abil, seejärel paigaldatud `virtualenv`).

**Oluline:** globaalne `pip`/`pip3` osutab baas-venv pip-ile, mitte süsteemsele
Pythonile. Projektide sõltuvused tuleb paigaldada **projekti enda
virtuaalkeskkonda**, mitte baas-venv-i ega süsteemsesse Pythonisse.
Väljaspool virtuaalkeskkonda `python -m pip` tahtlikult ei tööta, sest
süsteemsel Pythonil pip puudub; aktiveeritud venv-is töötab see tavapäraselt.

### Virtuaalkeskkonna loomine

```bash
source ~/dev-workspace/scripts/env.sh
mkvenv ~/dev-workspace/projects/minuprojekt/.venv
source ~/dev-workspace/projects/minuprojekt/.venv/bin/activate
pip install <pakett>
deactivate
```

Kontrollitud: `mkvenv /tmp/pyprojtest` lõi venv-i pip 26.2.2-ga ning
`pip install six` paigaldas paketi edukalt (`six 1.17.0`). Süsteemne
`python3 -m pip` jäi puutumata.

---

## 4. Git

| Näitaja | Väärtus |
|---|---|
| Versioon | **2.47.3** |
| Binaar | `/usr/bin/git` |
| Globaalne config | Puudub |
| `user.name` | **Seadistamata** (nõutud, teadlikult) |
| `user.email` | **Seadistamata** (nõutud, teadlikult) |
| `dev-workspace` Git-repo | Ei ole algatatud |

Git on paigaldatud ja töötab, kuid commit'imiseks tuleb identiteet hiljem
kasutaja kinnitusega seadistada:

```bash
git config --global user.name "Eesnimi Perekonnanimi"
git config --global user.email "sinu@email.tld"
git config --global init.defaultBranch main
```

Neid käske **ei käivitatud** käesoleva töö käigus.

---

## 5. Tööruumi struktuur

```
/home/iffi/dev-workspace/
├── projects/   # projektide kood
├── tools/      # paigaldatud tööriistad (Node.js, python-base)
├── scripts/    # abiskriptid (env.sh)
├── logs/       # logid
├── backups/    # varukoopiad
└── docs/       # dokumentatsioon (see fail)
```

### Keskkonna aktiveerimine skriptides

```bash
source ~/dev-workspace/scripts/env.sh
```

Skript lisab PATH-i (idempotentselt, duplikaate vältides):

1. `~/.local/bin` — Node/npm/npx/corepack, python, pip, pip3, mkvenv
2. `~/dev-workspace/tools/node/bin` — Node.js LTS binaarid
3. `~/.opencode/bin` — `opencode` binaar

`~/.profile` ja `~/.bashrc` laadivad selle skripti automaatselt, seega
tavapärases login- või interaktiivses shellis pole käsitsi source'imist vaja.
Mitteinteraktiivsetes skriptides tee siiski:

```bash
source ~/dev-workspace/scripts/env.sh
```

---

## 6. Shellid, tmux ja OpenCode

### Kontrollitulemused (2026-10-30)

| Keskkond | node | npm | python | pip | opencode |
|---|---|---|---|---|---|
| `bash -lc` (login, mitteinteraktiivne) | OK | OK | OK | OK | OK |
| `bash -ic` (interaktiivne) | OK | OK | OK | OK | OK |
| `tmux new-session` (vaikimisi shell) | OK | OK | OK | OK | OK |

Lisaks:

- `scripts/env.sh` on idempotentne — korduv source'imine ei tekita PATH-i duplikaate.
- Login-shellis olid varem `node`/`npm`/`python`/`pip` leitavad, kuid `opencode`
  **ei olnud**, sest `~/.profile` laadis `~/.bashrc`-i, mis väljus
  mitteinteraktiivses režiimis enne PATH-i seadistamist. Parandatud: `~/.profile`
  laadib nüüd `scripts/env.sh` otse.
- `~/.local/bin` duplikaat PATH-is (tekkis `.profile` ja `.bashrc` koostoimel) on
  kõrvaldatud.
- tmux 3.5a uues sessioonis oli PATH puhas:
  `/home/iffi/.local/bin:/home/iffi/dev-workspace/tools/node/bin:/home/iffi/.opencode/bin:...`

### Püsiva tmux-sessiooni käivitamine

```bash
# Loo nimeline sessioon arendustööruumis
tmux new-session -s dev -c ~/dev-workspace

# Sessioonist lahku (protsessid jäävad tööle): Ctrl-b, seejärel d
# Sessioonide nimekiri
tmux ls

# Naase sessiooni
tmux attach-session -t dev

# Sessiooni lõpetamine
tmux kill-session -t dev
```

Sessiooni sees saab OpenCode'i käivitada foreground-protsessina:

```bash
cd ~/dev-workspace
opencode
```

OpenCode'i ei käivitatud taustal ega loodud systemd-teenust. Kui tmux ise
käivitatakse keskkonnast, kus PATH on ebatäielik, saab paneedis käivitada
`source ~/dev-workspace/scripts/env.sh` või alustada sessiooni login-shelliga:

```bash
tmux new-session -s dev -c ~/dev-workspace 'exec bash -l'
```

---

## 7. Kontrollitud tööriistad

| Kontroll | Tulemus |
|---|---|
| `node --version` | `v24.22.2` |
| `npm --version` | `11.19.1` |
| `npx --version` | `11.19.1` |
| `corepack --version` | `0.36.0` |
| `npm view npm version` | `12.2.1` (register kättesaadav) |
| `python --version` | `Python 3.13.5` |
| `python3 --version` | `Python 3.13.5` |
| `pip --version` | `pip 26.2.2` |
| `mkvenv` + `pip install six` | OK (`six 1.17.0`) |
| `git --version` | `2.47.3` |
| Git identiteet | Teadlikult seadistamata |
| `bash -lc` + `bash -ic` | Kõik vajalikud käsud leitavad |
| `tmux new-session` | Kõik vajalikud käsud leitavad, PATH puhas |
| Qortiumi protsess | Töötab (PID 13615), muutmata |
| Swap | 0 B, muutmata |
| systemd-teenus / taustal OpenCode | Ei loodud / ei käivitatud |

---

## 8. Muudatused kokkuvõtvalt

1. Loodud `/home/iffi/.local/bin` sümlingid ja wrapperid:
   `node`, `npm`, `npx`, `corepack`, `python`, `pip`, `pip3`, `mkvenv`.
2. Paigaldatud Node.js `v24.22.2` LTS koos npm/npx/corepack-iga
   `/home/iffi/dev-workspace/tools/` alla (ametlik tarball, SHA256 kontrollitud).
3. Loodud Pythoni baas-venv `tools/python-base` pip 26.2.2 ja virtualenv 21.15.1-ga.
4. `scripts/env.sh` ühtlustatud PATH-i allikaks: `~/.local/bin`,
   `tools/node/bin`, `~/.opencode/bin` (idempotentne).
5. `~/.bashrc` ja `~/.profile` laadivad nüüd `scripts/env.sh`; parandatud
   `opencode` puudumine mitteinteraktiivses login-shellis ja PATH-i duplikaat.
6. Loodud `dev-workspace/{projects,tools,scripts,logs,backups,docs}`.
7. Loodud `scripts/env.sh` ja käesolev dokumentatsioon.

**Ei muudetud:** Qortiumi failid/seadised/automaatuuendus/käivitusskriptid,
Qortiumi protsess, SSH seadistus, swap, teenused. API võtmeid ega saladusi ei
loetud ega väljastatud.
