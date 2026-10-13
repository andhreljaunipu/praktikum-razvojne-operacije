# Blok A (80 min): Uvod, SDLC, DevOps, CI i produkcijski sustavi

## Raspored
| Min | Cjelina | Oblik |
|---|---|---|
| 0–10 | Uvod u kolegij | Prezentacija (postojeći izvedbeni plan, slajdovi 1–12) |
| 10–25 | SDLC | Predavanje + pitanje grupi |
| 25–42 | DevOps i presjek sa SDLC-om | Predavanje |
| 42–57 | CI workflowi | Predavanje + živi prikaz workflowa |
| 57–75 | Hardver i produkcijski sustavi | Predavanje + rasprava |
| 75–80 | Provjera razumijevanja | 5 kratkih pitanja |

## 1. Uvod u kolegij (10 min)
- Cilj i 5 ishoda učenja, 12 tjedana × 2 bloka.
- Semestralni projekt: Python skripte → Ollama + Open WebUI u Dockeru → DevOps lokalno i na zajedničkom GCP VM-u.
- Vrednovanje: praktični zadaci 40 %, individualne i grupne zadaće 30 %, prezentacija 30 %.
- Organizacija: GitHub organizacija kolegija, osobni repozitorij sada, timski kasnije.

## 2. SDLC: životni ciklus razvoja softvera (15 min)
- Faze: planiranje → analiza zahtjeva → dizajn → implementacija → testiranje → isporuka → održavanje.
- Modeli: vodopadni, iterativni/inkrementalni, agilni (Scrum, najava teme 12).
- Ključna misao: tradicionalno je razvoj predavao kod operacijama "preko zida"; isporuka je bila rijetka, rizična i ručna.
- **Pitanje grupi:** koliko često mislite da Google ili Netflix isporučuju promjene u produkciju? (odgovor: tisuće puta dnevno)

## 3. DevOps i presjek sa SDLC-om (17 min)
- DevOps = kultura + prakse + alati koji spajaju razvoj (Dev) i operacije (Ops).
- CALMS: Culture, Automation, Lean, Measurement, Sharing.
- Beskonačna petlja: **Plan → Code → Build → Test → Release → Deploy → Operate → Monitor**; preslikati je na faze SDLC-a (implementacija = Code/Build, testiranje = Test, isporuka = Release/Deploy, održavanje = Operate/Monitor).
- Razlika: SDLC opisuje *što* se radi, DevOps *kako* se to radi brzo, često i automatizirano, uz povratnu informaciju iz produkcije.
- DORA metrike (knjiga *Accelerate*): učestalost isporuke, vrijeme od commita do produkcije, stopa neuspjelih promjena, vrijeme oporavka.
- DevSecOps: sigurnost "lijevo" u petlji (provjere u kodu i CI-u, ne tek pred produkciju).
- Preslikavanje na kolegij: svaka tema 2–12 je jedna točka u petlji (pokazati tablicu iz [prijedlog-strukture.md](../prijedlog-strukture.md)).

## 4. CI workflowi (15 min)
- Kontinuirana integracija: svaki push automatski se gradi i testira; problemi se otkrivaju u minutama, ne tjednima.
- CI vs Continuous Delivery (uvijek spremno za isporuku, ručni gumb) vs Continuous Deployment (automatski u produkciju).
- Pojmovi: trigger (događaj), runner (računalo koje izvršava), job, step, artefakt, status provjere na PR-u.
- Živi prikaz: workflow u nekom javnom repozitoriju (zelene i crvene kvačice na PR-u), zatim ovaj minimalni primjer koji ćemo zapisati u naše repozitorije kao `.github/workflows/release_please.yml`:

```yaml
name: release-please

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - name: Dump GitHub context
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
        run: echo "$GITHUB_CONTEXT"

      - name: Release Please
        uses: googleapis/release-please-action@v4
        with:
          config-file: release-please-config.json
          manifest-file: .release-please-manifest.json
```

Uz workflow u korijen repozitorija idu tri datoteke:

`VERSION.txt`
```
0.0.1
```

`release-please-config.json`
```json
{
  "packages": {
    ".": {
      "release-type": "simple",
      "version-file": "VERSION.txt"
    }
  }
}
```

`.release-please-manifest.json`
```json
{
  ".": "0.0.1"
}
```

- Što pokazati na primjeru:
  - **Trigger** je `push` na `main`, **runner** je `ubuntu-latest`, job ima dva **stepa**.
  - Prvi korak ispisuje cijeli `github` kontekst (tko je pushao, koji commit, koja grana); tako studenti vide koje podatke workflow ima na raspolaganju.
  - Drugi korak čita commitove od zadnjeg izdanja. Commit `feat:` ili `fix:` otvara "release PR" koji podiže verziju u `VERSION.txt` i piše `CHANGELOG.md`. Spajanjem tog PR-a nastaju git tag i GitHub Release. Commit `chore:` sam po sebi ne otvara izdanje.
  - `permissions` daje workflowu najmanje ovlasti koje treba (DevSecOps: princip najmanjih ovlasti).
- Ovo je ujedno najava tema 9 (CI/CD) i 10 (verzioniranje i konvencionalne poruke).
- Naglasiti: runner je "tuđe računalo u oblaku" (most prema sljedećoj cjelini). Self-hosted runner je opcija kad treba vlastiti hardver, npr. GPU.

## 5. Hardver i produkcijski sustavi (18 min)
- Što je produkcijski sustav: stvarni korisnici, dostupnost, nadgledanje, sigurnosne kopije, sigurnost.
- **Self-hosted (on-premise):** vlastiti poslužitelji; potpuna kontrola, kapitalni trošak (CapEx), sami održavamo hardver, mrežu, struju, hlađenje; sporo skaliranje.
- **Cloud-hosted:** najam kapaciteta (OpEx), skaliranje u minutama, plaćanje po potrošnji; model podijeljene odgovornosti.
- IaaS / PaaS / SaaS na primjeru: VM na GCP-u (IaaS), Cloud Run (PaaS), ChatGPT (SaaS).
- Hibrid i edge kao srednji put.
- Naš slučaj: LLM sustav lokalno (laptop, besplatno, ograničen RAM/GPU) vs zajednički GCP VM (dijeljen, plaća se po satu, dostupan svima). Najava tema 6–8.
- **Rasprava (5 min):** startup s 3 programera i bolnički informacijski sustav: gdje bi tko hostao i zašto?

## 6. Provjera razumijevanja (5 min)
1. Koje su faze DevOps petlje i koja odgovara fazi održavanja u SDLC-u?
2. Razlika između continuous delivery i continuous deployment?
3. Što je runner u CI workflowu?
4. Navedi jednu prednost self-hosted i jednu cloud-hosted infrastrukture.
5. Koja je DORA metrika "vrijeme od commita do produkcije"?
