# Project Browseri nõuded

## 1. Eesmärk ja ulatus

Project Browser on Ubuntu serveris töötav iseseisev veebirakendus, mille abil saab brauseris enne Git push'i **ainult lugemiseks** vaadata serveris olevaid projektikatalooge, nende faile ja Git-metainfot.

Rakenduse eesmärk on toetada töövoogu:

```text
Codex muudab serveris faile
        ↓
Project Browser võimaldab inimesel tulemuse üle vaadata
        ↓
kui tulemus sobib
        ↓
git commit ja push tehakse eraldi töövoos
```

Rakendus ei ole IDE ega Git-klient. See ei sõltu teiste samas serveris asuvate rakenduste koodist, andmebaasist ega kasutajamudelist.

## 2. Keel ja märgistik

- Kasutajaliides, veateated, abitekstid, dokumentatsioon ja konfiguratsiooni kommentaarid on eesti keeles.
- Kõik projekti tekstifailid ning HTTP vastused kasutavad UTF-8 märgistikku.
- HTML-dokumentides määratakse UTF-8 selgesõnaliselt (`<meta charset="utf-8">`) ning tekstivastuste `Content-Type` päises kasutatakse UTF-8 märgistikku.

## 3. Projekti asukoht ja struktuur

- Git-repo asukoht on `/home/ubuntu/projektid/project-browser`.
- Rakendus on eraldi Git-repo.
- Soovituslik struktuur sisaldab vähemalt kaustu `app/`, `templates/`, `static/` ja `config/` ning faile `Dockerfile`, `docker-compose.yml`, `requirements.txt`, `.env.example` ja `.gitignore`.
- Juurkataloogis asuv `.env` ei kuulu Git-reposse. `.env.example` sisaldab ainult seadistusvõtmete nimesid või ohutuid näidisväärtusi.

## 4. Tehnoloogia ja käitamine

- Backend on Pythonil ja FastAPI-l.
- Rakendus töötab eraldi Docker Compose'i projektina.
- Git-andmeid loetakse kohalikest Git-repodest; GitHub API-t ei kasutata.
- Git-info lugemiseks võib kasutada süsteemi `git` käsku või sobivat Pythoni teeki. Lahendus peab olema lihtne ja läbipaistev.
- Rakendus peab töötama URL-i prefiksi `/projects/` taga.

## 5. Projektide avastamine

- Hostis on projektide juurkaust `/home/ubuntu/projektid`.
- See ühendatakse konteinerisse ainult lugemiseks: `/home/ubuntu/projektid:/projects:ro`.
- Rakendus avastab Git-repod automaatselt ainult `/projects` esimese taseme alamkataloogidest. Repo tunnus on `.git` kataloog.
- `project-browser` on esialgu nimekirjas samamoodi nagu kõik teised projektid.
- Uue Git-repo lisamisel `/home/ubuntu/projektid` alla peab see olema nähtav ilma rakenduse koodi või taaskäivituse muutmata.
- Praegu on projektide juurkaustas `icm0032_ryhm33` ja `project-browser`; mõlemad on vaikimisi privaatsed.

## 6. Read-only põhimõte

### Dockeris

- `/projects` on mountitud valikuga `:ro`.

### Rakenduses

- Puuduvad upload-, muutmis-, kustutamis-, ümbernimetamis- ja teisaldamisendpoint'id.
- Puuduvad Git write-operatsioonid: rakendus ei tee commit'i, checkout'i, merge'i, pull'i ega push'i.
- Rakendus kuvab ainult faile ja Git-metainfot.

## 7. Projektide ülevaade

Projektide avaleht näitab iga nähtava repo kohta vähemalt:

- projekti nime;
- aktiivset branch'i;
- Git status'e kokkuvõtet;
- viimast commit'i (lühike räsi ja sõnum);
- viimase commiti aega.

Git status'es eristatakse vähemalt olekuid `modified`, `added`, `deleted`, `renamed` ja `untracked`.

## 8. Failide sirvimine ja kuvamine

- Igas projektis saab sirvida kataloogipuud.
- Faili juures kuvatakse nimi, suurus, Git status ning kas fail on tracked või untracked.
- Markdowni (`.md`) puhul on renderdatud vaade ja raw-tekstivaade.
- Tekstivaates toetatakse vähemalt laiendeid `.txt`, `.py`, `.json`, `.yaml`, `.yml`, `.toml`, `.ini`, `.csv`, `.js`, `.ts`, `.html`, `.css`, `.sql` ja `.sh`.
- PDF-failid avatakse brauseri native PDF-vaaturis.
- Pildifailid `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp` ja `.svg` kuvatakse brauseris.
- Tundmatute binaarfailide korral näidatakse ainult metadata infot; sisu ei renderdata tekstina.
- Faili sisu ei avata ega saadeta brauserile, kui faili suurus ületab 2 MB. Kasutajale kuvatakse selge eestikeelne teade faili suuruse ja 2 MB piirangu kohta.

## 9. Tundlike failide ja teede kaitse

Rakendus ei kuva vaikimisi järgmisi faile ega katalooge:

- `.git/`
- `.env` ja `.env.*`
- `*.key`, `*.pem`, `*.p12`, `*.pfx`
- `id_rsa`, `id_ed25519`
- `credentials*`, `secrets*`

Kõik kasutajalt tulevad teed normaliseeritakse ja kontrollitakse. Path traversal ei tohi võimaldada väljuda valitud projekti kataloogist ega lugeda muid hosti faile.

## 10. Autentimine ja sessioonid

- Ligipääs toimub Google Identity Servicesi ID-tokeni põhise sisselogimisega.
- Ei kasutata kasutajanime/parooli, Basic Authi, `fun_o` kasutajabaasi ega `fun_o` sessioone.
- Project Browser kasutab sama `GOOGLE_CLIENT_ID` väärtust nagu `fun_o`, kuid see väärtus paikneb Project Browseri enda `.env` failis.
- Project Browseril on oma `SESSION_SECRET` ja oma küpsise nimi (näiteks `project_browser_session`); neid ei jagata `fun_o`ga.
- Google'i ID-tokeni ehtsust ja selle audience'i kontrollitakse igal sisselogimisel.
- `.env.example` sisaldab vähemalt võtmeid `GOOGLE_CLIENT_ID`, `SESSION_SECRET` ja `SESSION_COOKIE_NAME`.

## 11. Projektipõhine autoriseerimine

- ACL on esimeses versioonis failipõhine, näiteks `config/projects.yaml`.
- Vaikimisi on autentimine nõutud ning seadistamata uus repo on tavakasutajale nähtamatu.
- Admin näeb kõiki projekte. Esialgne admin on `alar.joeste@gmail.com`.
- Tavakasutaja näeb ainult projekte, mille `allowed_users` nimekirjas on tema Google'i e-posti aadress.
- Projekt võib olla avalik, kui `authentication_required: false`; praegu ei ole ükski projekt avalik.
- Autoriseerimist kontrollitakse igal projektiga seotud päringul, sealhulgas käsitsi sisestatud projekti- ja failiaadressil. Projekti peitmisest kasutajaliideses üksi ei piisa.

Näidiskonfiguratsioon:

```yaml
defaults:
  authentication_required: true

admins:
  - alar.joeste@gmail.com

projects:
  icm0032_ryhm33:
    allowed_users: []
  project-browser:
    allowed_users: []
```

## 12. Docker, võrk ja reverse proxy

- Compose'i teenus ei ava hostis uut porti; kasutatakse ainult `expose: 8000`, mitte `ports` seadistust.
- Teenus ühendub olemasoleva välise Docker-võrguga `funo_net`.
- Avalik HTTPS reverse proxy on konteiner `funo_nginx`; selle Nginxi konfiguratsioon asub praegu `/home/ubuntu/fun_o/nginx/default.conf`.
- Project Browser on avalikult ligipääsetav ainult HTTPS-i kaudu aadressil `https://fun-o.eu/projects/`.
- Nginxi HTTPS-serveriplokki lisatakse `/projects/` asukohareegel, mis proxib Project Browseri konteinerisse.
- Port 80, Let's Encrypti ACME challenge ja olemasolev HTTPS-redirect jäävad puutumata.
- `fun_o`, `Double_Check_AI` ja Ollama olemasolevat käitamist ei muudeta rohkem, kui `/projects/` reverse proxy lisamiseks vältimatult vajalik.

## 13. MVP piirid

Esimene töötav versioon sisaldab Docker-konteinerit, FastAPI backend'i, automaatset repo-avastust, projekti ülevaadet, Git infot, failide sirvimist ja toetusvorme, tundlike failide kaitset, Google'i sisselogimist, projektipõhist ACL-i, adminiõigust, read-only failisüsteemi, `funo_net` integratsiooni ning `/projects/` reverse proxy tuge.

Esimeses versioonis ei tehta Git commit'i, push'i, pull'i, branch'i vahetamist, failide muutmist, üleslaadimist ega kustutamist, GitHub API integratsiooni, andmebaasi ega keerukat kasutajahaldusliidest.
