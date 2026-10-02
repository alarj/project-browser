# Project Browseri nõuded

## 1. Eesmärk ja ulatus

Project Browser on Ubuntu serveris töötav iseseisev veebirakendus, mille abil saab brauseris **ainult lugemiseks** vaadata serveris olevaid projektikatalooge ja nende faile.

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

Rakendus asendab kaugserveris projekti sirvimiseks inimese kohaliku arvuti failibrauserit ning failidesse sisse vaatamiseks kasutatavat lihtsat tekstivaadet. Kõiki lubatud faile peab saama sirvida ja avada sõltumata sellest, kas Git neid jälgib, need on untracked või Gitist ignoreeritud.

Kataloogipuu peab aitama kasutajal kiiresti leida, millistes kataloogides vajalikud failid asuvad. See võimaldab agentidele antavates promptides viidata täpselt õigetele projektisisestele failidele ja kataloogidele.

Rakendus ei ole IDE ega alternatiivne Git-klient. Git status, aktiivne branch ja viimase commiti info on projekti ülevaates ainult lisainfo. Rakendus ei sõltu teiste samas serveris asuvate rakenduste koodist, andmebaasist ega kasutajamudelist.

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
- JSON- (`.json`) ning XML-põhistel (`.xml`, `.xsl`, `.xsd`) failidel on lisaks raw-tekstivaatele vormindatud (pretty-print) vaade, kus struktuur on taandatud ja lihtsasti loetav.
- Tekstivaates toetatakse vähemalt laiendeid `.txt`, `.py`, `.json`, `.yaml`, `.yml`, `.toml`, `.ini`, `.csv`, `.js`, `.ts`, `.html`, `.htm`, `.xml`, `.xsl`, `.xsd`, `.css`, `.sql` ja `.sh`.
- PDF-failid avatakse brauseri native PDF-vaaturis.
- Pildifailid `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp` ja `.svg` kuvatakse brauseris.
- Tundmatute binaarfailide korral näidatakse ainult metadata infot; sisu ei renderdata tekstina.
- Faili sisu ei avata ega saadeta brauserile, kui faili suurus ületab 2 MB. Kasutajale kuvatakse selge eestikeelne teade faili suuruse ja 2 MB piirangu kohta.

## 9. Tundlike failide ja teede kaitse

Rakendus ei kuva vaikimisi järgmisi faile ega katalooge:

- `.git/`
- päris keskkonnafailid `.env` ja `.env.*`; erandina on kõik `*.env.example` näidisfailid kuvamiseks lubatud;
- `*.key`, `*.pem`, `*.p12`, `*.pfx`
- `id_rsa`, `id_ed25519`
- `credentials*`, `secrets*`

Failinime kontrollis rakendatakse `*.env.example` lubamisreeglit enne tundlike keskkonnafailide keelureeglit. Näidisfailid ei tohi sisaldada päris saladusi.

Kõik kasutajalt tulevad teed normaliseeritakse ja kontrollitakse. Path traversal ei tohi võimaldada väljuda valitud projekti kataloogist ega lugeda muid hosti faile.

## 10. Autentimine ja sessioonid

- Ligipääs toimub Google Identity Servicesi ID-tokeni põhise sisselogimisega.
- Ei kasutata kasutajanime/parooli, Basic Authi, `fun_o` kasutajabaasi ega `fun_o` sessioone.
- Project Browser kasutab sama `GOOGLE_CLIENT_ID` väärtust nagu `fun_o`, kuid see väärtus paikneb Project Browseri enda `.env` failis.
- Project Browseril on oma `SESSION_SECRET` ja oma küpsise nimi (näiteks `project_browser_session`); neid ei jagata `fun_o`ga.
- Google'i ID-tokeni ehtsust ja selle audience'i kontrollitakse igal sisselogimisel.
- `.env.example` sisaldab vähemalt võtmeid `GOOGLE_CLIENT_ID`, `SESSION_SECRET` ja `SESSION_COOKIE_NAME`.

## 11. Projektipõhine autoriseerimine MVP-s

- ACL on esimeses versioonis failipõhine: `config/projects.yaml` on lokaalne, Gitist välja jäetud konfiguratsioonifail.
- Git-repos on ainult isikuandmeteta näidisfail `config/projects.example.yaml`.
- Vaikimisi on autentimine nõutud ning seadistamata uus repo on tavakasutajale nähtamatu.
- Admin näeb kõiki projekte. Tegelikud adminide ja kasutajate e-posti aadressid paiknevad ainult lokaalses ACL-failis, mitte Git-repos.
- Tavakasutaja näeb ainult projekte, mille `allowed_users` nimekirjas on tema Google'i e-posti aadress.
- Projekt võib olla avalik, kui `authentication_required: false`; praegu ei ole ükski projekt avalik.
- Autoriseerimist kontrollitakse igal projektiga seotud päringul, sealhulgas käsitsi sisestatud projekti- ja failiaadressil. Projekti peitmisest kasutajaliideses üksi ei piisa.

Näidiskonfiguratsioon:

```yaml
defaults:
  authentication_required: true

admins:
  - admin@miskidomeen.ee

projects:
  icm0032_ryhm33:
    allowed_users:
      - kasutaja1@miskimuudomeen.ee
  project-browser:
    allowed_users:
      - kasutaja2@miskimuudomeen.ee
```

## 12. Tulevane kasutajahaldus ja Oracle'i integratsioon

- YAML-põhine ACL on ainult MVP lahendus. Kasutajahaldusliidese lisamisel asendatakse see Project Browseri enda püsiva õiguste andmebaasiga.
- Project Browser kasutab selleks olemasolevat Oracle Cloudi andmebaasi ja Oracle REST Data Servicesit (ORDS), kuid ei kasuta teiste rakenduste skeeme, tabeleid, pakette ega ORDS endpoint'e.
- Oracle'is luuakse Project Browserile eraldi skeemid õiguste andmete ja ORDS API jaoks. Skeemide täpsed nimed määratakse juurutamisel.
- FastAPI suhtleb Oracle'iga ainult ORDS-i HTTPS API kaudu; rakendus ei ava otsest Oracle'i draiveriühendust.
- Project Browseri lokaalne `.env` sisaldab oma `ORDS_BASE_URL`, `ORDS_USERNAME` ja `ORDS_PASSWORD` väärtusi. Neid ei lisata Git-reposse ega jagata teiste rakendustega.
- Püsiv õiguste mudel sisaldab vähemalt kasutajaid, rolle või adminiõigusi, automaatselt avastatud projekte ning kasutaja-projekti ligipääsu seoseid.
- Õigused seotakse Google'i püsiva kasutajatunnusega `sub`; e-posti aadress on abiteave, mitte õiguste põhitunnus.
- Päris kasutaja- ja adminandmed ei paikne Git-repos. Tulevases andmebaasimudelis hoitakse neid ainult Project Browseri eraldi skeemis.

## 13. Docker, võrk ja reverse proxy

- Compose'i teenus ei ava hostis uut porti; kasutatakse ainult `expose: 8000`, mitte `ports` seadistust.
- Teenus ühendub olemasoleva välise Docker-võrguga `funo_net`.
- Avalik HTTPS reverse proxy on konteiner `funo_nginx`; selle Nginxi konfiguratsioon asub praegu `/home/ubuntu/fun_o/nginx/default.conf`.
- Project Browser on avalikult ligipääsetav ainult HTTPS-i kaudu aadressil `https://fun-o.eu/projects/`.
- Nginxi HTTPS-serveriplokki lisatakse `/projects/` asukohareegel, mis proxib Project Browseri konteinerisse.
- Port 80, Let's Encrypti ACME challenge ja olemasolev HTTPS-redirect jäävad puutumata.
- `fun_o`, `Double_Check_AI` ja Ollama olemasolevat käitamist ei muudeta rohkem, kui `/projects/` reverse proxy lisamiseks vältimatult vajalik.

## 14. MVP piirid

Esimene töötav versioon sisaldab Docker-konteinerit, FastAPI backend'i, automaatset repo-avastust, projekti ülevaadet, Git infot, failide sirvimist ja toetusvorme, tundlike failide kaitset, Google'i sisselogimist, projektipõhist ACL-i, adminiõigust, read-only failisüsteemi, `funo_net` integratsiooni ning `/projects/` reverse proxy tuge.

Esimeses versioonis ei tehta Git commit'i, push'i, pull'i, branch'i vahetamist, failide muutmist, üleslaadimist ega kustutamist, Git diff'i, GitHub API integratsiooni, andmebaasi ega keerukat kasutajahaldusliidest.
