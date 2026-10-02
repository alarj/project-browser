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
versioonihalduse toimingud tehakse eraldi töövoos
```

Rakendus asendab kaugserveris projekti sirvimiseks inimese kohaliku arvuti failibrauserit ning failidesse sisse vaatamiseks kasutatavat lihtsat tekstivaadet. Kõiki lubatud faile peab saama sirvida ja avada nende versioonihalduse seisust sõltumata.

Kataloogipuu peab aitama kasutajal kiiresti leida, millistes kataloogides vajalikud failid asuvad. See võimaldab agentidele antavates promptides viidata täpselt õigetele projektisisestele failidele ja kataloogidele.

Rakendus ei ole IDE ega alternatiivne Git-klient. MVP ei käivita Git-käske ega kuva Git status't, branch'i, commit'e ega failide tracked/untracked olekut. Rakendus ei sõltu teiste samas serveris asuvate rakenduste koodist, andmebaasist ega kasutajamudelist.

## 2. Keel ja märgistik

- Kasutajaliides, veateated, abitekstid, dokumentatsioon ja konfiguratsiooni kommentaarid on eesti keeles.
- Rakenduse enda tekstifailid ja HTTP tekstivastused kasutavad UTF-8 märgistikku.
- HTML-dokumentides määratakse UTF-8 selgesõnaliselt (`<meta charset="utf-8">`) ning tekstivastuste `Content-Type` päises kasutatakse UTF-8 märgistikku.
- Vaadatav projektifail võib olla muus märgistikus. Kui faili ei saa ohutult UTF-8-na lugeda, näidatakse selle kohta selget eestikeelset teadet, mitte vigaselt dekodeeritud sisu.

## 3. Projekti asukoht ja struktuur

- Git-repo asukoht on `/home/ubuntu/projektid/project-browser`.
- Rakendus on eraldi Git-repo.
- Soovituslik struktuur sisaldab vähemalt kaustu `app/`, `templates/`, `static/` ja `config/` ning faile `Dockerfile`, `docker-compose.yml`, `requirements.txt`, `.env.example` ja `.gitignore`.
- Juurkataloogis asuv `.env` ei kuulu Git-reposse. `.env.example` sisaldab ainult seadistusvõtmete nimesid või ohutuid näidisväärtusi.

## 4. Tehnoloogia ja käitamine

- Backend on Pythonil ja FastAPI-l.
- Rakendus töötab eraldi Docker Compose'i projektina.
- Projekti tuvastamiseks kasutatakse esimese taseme kataloogis olevat `.git` kataloogi või Git worktree `.git` faili; rakendus ei loe selle kaudu Git-metainfot ega käivita `git` käsku.
- GitHub API-t ei kasutata.
- Rakendus peab töötama oma alamdomeeni `https://projects.fun-o.eu/` juurel.

## 5. Projektide avastamine

- Hostis on projektide juurkaust `/home/ubuntu/projektid`.
- See ühendatakse konteinerisse ainult lugemiseks: `/home/ubuntu/projektid:/projects:ro`.
- Rakendus avastab projektid automaatselt ainult `/projects` esimese taseme alamkataloogidest. Projekti tunnus on `.git` kataloog või Git worktree `.git` fail.
- `project-browser` on esialgu nimekirjas samamoodi nagu kõik teised projektid.
- Uue Git-repo lisamisel `/home/ubuntu/projektid` alla peab see olema nähtav ilma rakenduse koodi või taaskäivituse muutmata.
- Praegu on projektide juurkaustas `icm0032_ryhm33` ja `project-browser`; mõlemad on vaikimisi privaatsed.

## 6. Read-only põhimõte

### Dockeris

- `/projects` on mountitud valikuga `:ro`.

### Rakenduses

- Puuduvad upload-, muutmis-, kustutamis-, ümbernimetamis- ja teisaldamisendpoint'id.
- Rakendus ei käivita Git-käske ega tee versioonihalduse operatsioone.
- Rakendus kuvab ainult faile ja nende failisüsteemi metainfot.

## 7. Projektide ülevaade

Projektide avaleht näitab iga nähtava projekti kohta vähemalt:

- projekti nime;
- projekti suhtelist asukohta;
- projekti avamise linki.

## 8. Failide sirvimine ja kuvamine

- Igas projektis saab sirvida kataloogipuud. Kataloogi sisu laaditakse alles selle avamisel; kogu puud ei loeta rekursiivselt korraga.
- Faili ja kataloogi juures kuvatakse projekti juure suhteline tee. Kasutaja saab selle ühe nupuvajutusega kopeerida, et kasutada seda agendile antavas promptis.
- Faili juures kuvatakse nimi, suurus ja muutmisaeg. Kataloogi sisu saab sortida nime, suuruse või muutmisaja järgi ning filtreerida failinime järgi.
- Vaates on "Värskenda" nupp ja viimase värskenduse aeg. Värskendamine loeb avatud kataloogi failisüsteemist uuesti.
- Rasked kataloogid, näiteks `node_modules`, `venv`, `.venv`, `__pycache__` ja `.cache`, ei tohi põhjustada rekursiivset eellaadimist. Neid saab vajaduse korral eraldi avada, kui neid pole turbereegliga peidetud.
- Markdowni (`.md`) puhul on renderdatud vaade ja raw-tekstivaade.
- JSON- (`.json`) ning XML-põhistel (`.xml`, `.xsl`, `.xsd`) failidel on lisaks raw-tekstivaatele vormindatud (pretty-print) vaade, kus struktuur on taandatud ja lihtsasti loetav.
- Tekstivaates toetatakse vähemalt laiendeid `.txt`, `.py`, `.json`, `.yaml`, `.yml`, `.toml`, `.ini`, `.csv`, `.js`, `.ts`, `.html`, `.htm`, `.xml`, `.xsl`, `.xsd`, `.css`, `.sql` ja `.sh` ning laiendita UTF-8 tekstifaile nagu `Dockerfile`, `Makefile`, `LICENSE` ja `README`.
- Tekstivaates on reanumbrid, URL-i reaviited kujul `#L120` ning pika rea murdmise või horisontaalse kerimise kasutajavalik.
- PDF-failid avatakse brauseri native PDF-vaaturis.
- Pildifailid `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp` ja `.svg` kuvatakse brauseris.
- Tundmatute binaarfailide korral näidatakse ainult metadata infot; sisu ei renderdata tekstina.
- Faili sisu ei avata ega saadeta brauserile, kui faili suurus ületab 2 MB. Kasutajale kuvatakse selge eestikeelne teade faili suuruse ja 2 MB piirangu kohta.
- Failide allalaadimise endpoint'i MVP-s ei ole.

## 9. Tundlike failide ja teede kaitse

Rakendus ei kuva vaikimisi järgmisi faile ega katalooge:

- `.git/`
- päris keskkonnafailid `.env` ja `.env.*`; erandina on kõik `*.env*.example` näidisfailid kuvamiseks lubatud;
- `*.key`, `*.pem`, `*.p12`, `*.pfx`
- `id_rsa`, `id_ed25519`
- `credentials*`, `secrets*`, `.npmrc`, `.pypirc`, `.netrc`, `.git-credentials`, `.htpasswd`
- `*.tfstate`, `*.tfvars`, `*.kdbx`, `*.sqlite`

Failinime kontrollis rakendatakse `*.env*.example` lubamisreeglit enne tundlike keskkonnafailide keelureeglit. Näidisfailid ei tohi sisaldada päris saladusi.

Lokaalne `config/projects.yaml` toetab projektipõhist `hidden_paths` glob-mustrite nimekirja, mille abil saab peita projekti eripäraseid tundlikke või ebaolulisi teid. Seda loendit ei lisata Git-reposse.

Kõik kasutajalt tulevad teed normaliseeritakse ja kontrollitakse. Path traversal ei tohi võimaldada väljuda valitud projekti kataloogist ega lugeda muid hosti faile. Symlink'e ei järgita ega avata; avatava tee tegelik asukoht peab jääma valitud projekti juure sisse.

Markdown renderdatakse sanitiseeritult. HTML-faile kuvatakse ainult tekstivaates. SVG-faile serveeritakse pildina, mitte usaldatud HTML-dokumendina. Rakendus saadab vähemalt päised `Content-Security-Policy` ja `X-Content-Type-Options: nosniff`.

## 10. Autentimine ja sessioonid

- Ligipääs toimub Google Identity Servicesi ID-tokeni põhise sisselogimisega.
- Ei kasutata kasutajanime/parooli, Basic Authi, `fun_o` kasutajabaasi ega `fun_o` sessioone.
- Project Browser kasutab sama `GOOGLE_CLIENT_ID` väärtust nagu `fun_o`, kuid see väärtus paikneb Project Browseri enda `.env` failis.
- Project Browseril on oma `SESSION_SECRET` ja oma küpsise nimi (näiteks `project_browser_session`); neid ei jagata `fun_o`ga.
- Google'i ID-tokeni ehtsust, väljastajat (`iss`), audience'it (`aud`), aegumist (`exp`) ja kinnitatud e-posti (`email_verified`) kontrollitakse igal sisselogimisel.
- Sessioonil on määratud aegumine ning kasutaja saab välja logida. Sessiooni muutvad päringud kasutavad CSRF-kaitset või samaväärset päritolu kontrolli ning sisselogimist piiratakse mõistliku rate limit'iga.
- `.env.example` sisaldab vähemalt võtmeid `GOOGLE_CLIENT_ID`, `SESSION_SECRET` ja `SESSION_COOKIE_NAME`.

## 11. Projektipõhine autoriseerimine MVP-s

- ACL on esimeses versioonis failipõhine: `config/projects.yaml` on lokaalne, Gitist välja jäetud konfiguratsioonifail.
- Git-repos on ainult isikuandmeteta näidisfail `config/projects.example.yaml`.
- Vaikimisi on autentimine nõutud ning seadistamata uus repo on tavakasutajale nähtamatu.
- Admin näeb kõiki projekte. Tegelikud adminide ja kasutajate e-posti aadressid paiknevad ainult lokaalses ACL-failis, mitte Git-repos.
- Tavakasutaja näeb ainult projekte, mille `allowed_users` nimekirjas on tema Google'i e-posti aadress.
- Kõik projektid on MVP-s privaatsed; avaliku projekti võimalust ei ole.
- Autoriseerimist kontrollitakse igal projektiga seotud päringul, sealhulgas käsitsi sisestatud projekti- ja failiaadressil. Projekti peitmisest kasutajaliideses üksi ei piisa. Õiguseta kasutajale vastatakse `404`-ga.
- Kui ACL-fail on puudu, vigane või loetamatu, keelatakse ligipääs kõigile projektidele (fail closed).
- Rakendus logib turbeauditi jaoks eduka ja ebaõnnestunud sisselogimise, autoriseerimisest keeldumise ning projekti- ja failivaatamise aja, kasutaja, projekti ja suhtelise failitee. Faili sisu ei logita.

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
- Autoriseerimiskood kasutab ACL-allika liidest: MVP-s annab selle YAML-fail, hiljem Oracle/ORDS. Ülejäänud rakenduse autoriseerimisloogika ei sõltu andmete hoiukohast.
- Project Browser kasutab selleks olemasolevat Oracle Cloudi andmebaasi ja Oracle REST Data Servicesit (ORDS), kuid ei kasuta teiste rakenduste skeeme, tabeleid, pakette ega ORDS endpoint'e.
- Oracle'is luuakse Project Browserile eraldi skeemid õiguste andmete ja ORDS API jaoks. Skeemide täpsed nimed määratakse juurutamisel.
- FastAPI suhtleb Oracle'iga ainult ORDS-i HTTPS API kaudu; rakendus ei ava otsest Oracle'i draiveriühendust.
- Project Browseri lokaalne `.env` sisaldab oma `ORDS_BASE_URL`, `ORDS_USERNAME` ja `ORDS_PASSWORD` väärtusi. Neid ei lisata Git-reposse ega jagata teiste rakendustega.
- Püsiv õiguste mudel sisaldab vähemalt kasutajaid, rolle või adminiõigusi, automaatselt avastatud projekte ning kasutaja-projekti ligipääsu seoseid.
- Õigused seotakse Google'i püsiva kasutajatunnusega `sub`; e-posti aadress on abiteave, mitte õiguste põhitunnus.
- Päris kasutaja- ja adminandmed ei paikne Git-repos. Tulevases andmebaasimudelis hoitakse neid ainult Project Browseri eraldi skeemis.

## 13. Docker, võrk ja reverse proxy

- Compose'i teenus ei ava hostis uut porti; kasutatakse ainult `expose: 8000`, mitte `ports` seadistust.
- Project Browser ühendub eraldi proxy-võrku, millega on liidetud ainult Project Browser ja reverse proxy. See ei ühendu üldise `funo_net` võrguga.
- Avalik HTTPS reverse proxy on konteiner `funo_nginx`; selle Nginxi konfiguratsioon asub praegu `/home/ubuntu/fun_o/nginx/default.conf`.
- Project Browser on avalikult ligipääsetav ainult HTTPS-i kaudu aadressil `https://projects.fun-o.eu/`.
- Nginxile lisatakse `projects.fun-o.eu` HTTPS-serveriplokk, mis proxib Project Browseri konteinerisse. Nginxi lisaseadistus hoitakse võimaluse korral eraldi include-failis.
- Port 80, Let's Encrypti ACME challenge ja olemasolev HTTPS-redirect jäävad puutumata.
- `projects.fun-o.eu` DNS CNAME on seadistatud viitama nimele `fun-o.eu` ning lahendub samale serverile. Praegune Let's Encrypti sertifikaat katab siiski ainult `fun-o.eu` ja `www.fun-o.eu`; enne Project Browseri avaldamist tuleb sertifikaati laiendada või väljastada uus sertifikaat, mis katab ka `projects.fun-o.eu`.
- Serveris juba töötavate rakenduste käitamist ei muudeta rohkem, kui Project Browseri Nginxi reverse proxy ja eraldi proxy-võrgu ühendamiseks vältimatult vajalik.
- Konteiner töötab mitte-root kasutajana, read-only juurfailisüsteemiga, minimaalsete Linuxi õigustega (`no-new-privileges`) ning ainult vajalikku ajutist kirjutusruumi pakkuva `tmpfs`-iga.
- Rakendusel on healthcheck ning struktureeritud logimine. Taaskäivitamisel peab rakendus taastuma ilma käsitsi andmete parandamiseta.

## 14. MVP piirid

Esimene töötav versioon sisaldab Docker-konteinerit, FastAPI backend'i, automaatset projektiavastust, failide ja kataloogide sirvimist, otsingut, sorteerimist, teede kopeerimist, toetusvorme, tundlike failide kaitset, Google'i sisselogimist, projektipõhist ACL-i, adminiõigust, read-only failisüsteemi, eraldi proxy-võrku ning `projects.fun-o.eu` reverse proxy tuge.

Esimeses versioonis ei tehta Git commit'i, push'i, pull'i, branch'i vahetamist ega muid Git-käske; samuti ei tehta failide muutmist, üleslaadimist ega kustutamist, Git diff'i, failide allalaadimist, GitHub API integratsiooni, andmebaasi, avalikke projekte ega keerukat kasutajahaldusliidest.

## 15. MVP vastuvõtukriteeriumid

- Autoriseeritud kasutaja saab avada lubatud projekti, sirvida selle katalooge laisalt ning kopeerida faili või kataloogi projekti suhtelise tee.
- Lubatud failide sirvimine ei sõltu Git-käskude, Git status'e ega commit'ide olemasolust.
- Failinime filter, sorteerimine ja "Värskenda" toimivad ka Gitist ignoreeritud failide korral.
- Markdowni, JSON-i ja XML-põhiste failide kokkulepitud vaated töötavad kuni 2 MB failidel; üle piiri faili sisu ei saadeta brauserile.
- Tundlik fail, `hidden_paths`-iga peidetud fail, path traversal ja symlink'i kaudu sihitud fail ei ole nähtavad ega avatavad.
- Õiguseta või autentimata kasutaja ei saa projekti ega selle faili avada, ka käsitsi koostatud URL-iga; vastus on `404`.
- ACL-faili tõrke korral ei ole ükski projekt tavakasutajale ega adminile ligipääsetav.
- Rakendus on HTTPS-i kaudu kättesaadav aadressil `https://projects.fun-o.eu/` ning ei ava hostis uut porti.
