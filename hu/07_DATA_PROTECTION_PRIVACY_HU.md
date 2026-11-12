---
name: data-protection-privacy
version: 1.1.0
last_reviewed: 2026-11-03
next_full_audit_due: 2027-02-01
jurisdiction: EU / Hungary
canonical_language_note: "CORE-LANG-001: EU/magyar jogszabályi és adatvédelmi (GDPR/NAIH) fogalmaknál a magyar (HU) szöveg a kanonikus."
---

# 07 – Data Protection & Privacy

## 1. Hatókör, jogi keret és kanonikus nyelv (`PRIV-SCOPE-001`)

Ez a modul (`Tier-1` szelektív doménmodul a `CORE-LOAD-001` szerint) minden olyan projektnél **kötelező**, amely személyes adatot (PII) kezel, vagy olyan technikai adatot (pl. IP-cím, eszközazonosító, munkamenet-telemetria) gyűjt, amely természetes személyhez köthetővé válhat. EU és magyarországi működés esetén az elsődleges jogi keret az **(EU) 2016/679 rendelet (GDPR)**, a kapcsolódó **EDPB** iránymutatások és az alkalmazandó magyar jogszabályok (valamint a **NAIH** gyakorlata).

- **Kanonikus nyelvi szabály (`CORE-LANG-001`):** EU és magyar jogszabályi, illetve adatvédelmi (GDPR/NAIH) fogalmaknál a magyar (`HU`) dokumentumváltozat szövege az irányadó (kanonikus).
- **Jogi elhatárolás (`CORE-DOCGATE-001`):** Ez a modul műszaki és szoftverarchitekturális végrehajtási útmutató, nem minősül jogi tanácsadásnak vagy hivatalos GDPR megfelelőségi tanúsításnak. A jogalap, az adatkezelő/adatfeldolgozó szerepkör, a megőrzési idők és az adatkezelési tájékoztató véglegesítése szükség esetén adatvédelmi tisztviselő (DPO) vagy szakjogász feladata.

---

## 2. Adatleltár és adatfolyam (`PRIV-INV-001`)

Minden személyes adatot kezelő projektben (a `SECURITY_PROFILE.md` részeként vagy külön adatleltárban) rögzíteni kell:
- a kezelt adatmezőket és adatcsoportokat;
- az adatkezelés konkrét célját és GDPR 6. cikk (különleges adatnál 9. cikk) szerinti jogalapját;
- az érintettek kategóriáit (pl. ügyfél, dolgozó, adminisztrátor, gyermek);
- az adat forrását és címzettjeit;
- a Firebase / Google Cloud tárolási helyet és régiót (pl. `europe-west1` / `eur3`);
- az adathoz hozzáférő szerepköröket és szerveroldali Service Accountokat;
- a megőrzési időt (retention) és a tényleges törlési/anonimizálási mechanizmust;
- a külső szolgáltatókhoz (adatfeldolgozókhoz) történő adattovábbítást;
- az alkalmazott biztonsági kontrollokat és hozzáférés-naplózást.

Az adatfolyam-diagramon jelöld a böngészőt/PWA-t, a Firebase Authot, a Firestore-t, a Cloud Storage-ot, a Cloud Functions-t, a külső API-kat, az analitikai szolgáltatásokat és az adminisztrátori eszközöket.

---

## 3. GDPR alapelvek érvényesítése a szoftverarchitektúrában (`PRIV-GDPR-001`)

A GDPR 5. cikkének alapelveit közvetlenül a rendszertervbe és az adatmodellbe kell beépíteni:
1. **Jogszerűség, tisztességes eljárás és átláthatóság** (`Lawfulness, fairness and transparency`).
2. **Célhoz kötöttség** (`Purpose limitation`).
3. **Adattakarékosság / adatminimalizálás** (`Data minimisation`).
4. **Pontosság** (`Accuracy`).
5. **Korlátozott tárolhatóság** (`Storage limitation`).
6. **Integritás és bizalmas jelleg** (`Integrity and confidentiality`).
7. **Elszámoltathatóság** (`Accountability`).

A projekt soha ne gyűjtsön „hátha később jó lesz” alapon adatmezőket. Minden egyes Firestore/Storage mezőhöz legyen dokumentált cél, jogalap és megőrzési idő.

---

## 4. Beépített és alapértelmezett adatvédelem – Privacy by Design and by Default (`PRIV-PBD-001`)

A GDPR 25. cikke és az EDPB 4/2019 iránymutatása alapján:
- Alapértelmezetten a legkevesebb személyes adatot kérje és tárolja a rendszer (`CORE-PRIN-011`).
- Az új felhasználói profilok és rekordok alapértelmezetten legyenek privátak, nem nyilvánosak, minimális alapjoggal (`FIREBASE-AUTH-001`).
- Nem alapvető analitikai vagy marketing követőkód (tracker) csak a megfelelő jogalap és (ahol szükséges) előzetes, tevőleges hozzájárulás után aktiválódhat.
- Az adatvédelmi beállítások legyenek egyértelműek, a hozzájárulás pedig bármikor ugyanolyan egyszerűen visszavonható, mint ahogyan megadták.
- Fejlesztés, emulátoros tesztelés és CI/CD során **tilos éles ügyféladatot használni**; használj szintetikus vagy visszafordíthatatlanul anonimizált tesztadatokat!
- Különleges kategóriájú személyes adatot (pl. egészségügyi adat – GDPR 9. cikk) csak kifejezett jogalap, fokozott biztonsági szint (L2/L3 mérlegelés a `CORE-TIER-001` szerint) és megfelelő garanciák mellett kezelj.
- Az adatvédelmi hatásvizsgálat (**DPIA** – GDPR 35. cikk) szükségességét a tényleges kockázati tényezők alapján értékeld a `SECURITY_PROFILE.md`-ben; ne állítsd automatikusan sem azt, hogy minden projekt DPIA-köteles, sem azt, hogy egyik sem az.

---

## 5. Adatbiztonság és naplózási tilalmak (`PRIV-SEC-001`)

- Minden hálózati adatátvitelhez kötelező a TLS/HTTPS; a szolgáltatói nyugalmi titkosítás (encryption at rest) mellett érzékeny tokenekre vagy különleges adatokra alkalmazz alkalmazásszintű borítéktitkosítást (pl. Google Cloud KMS – `FIREBASE-STORAGE-001`).
- A titkosítási kulcsokat Secret Manager / Cloud KMS megoldással kezeld, kulcsrotációval és hozzáférés-naplózással.
- **PII naplózási tilalom (`FIREBASE-IAM-001`):** Személyes adat (név, e-mail, telefonszám, lakcím, egészségügyi megjegyzés), jelszó, session azonosító vagy token **nem kerülhet** alkalmazáslogba (`Cloud Logging`), böngészőkonzolba, crash reportba, analitikai eseménybe vagy AI promptba! Hibák naplózásakor csak belső technikai azonosítót (`uid` – amennyiben biztonsági auditáláshoz szükséges) és szűrt hibakódot (`errorCode`) rögzíts.
- Minden adatexportot és adminisztratív tömeges lekérdezést naplózni kell.
- A biztonsági mentésekhez (backups) való hozzáférés is teljes értékű személyesadat-hozzáférésnek minősül, ezért külön szigorú IAM-korlátozást igényel.

---

## 6. Érintetti jogok, kaszkádolt törlés és PWA korlátok (`PRIV-RIGHTS-001`)

A rendszernek – az adott jogalap és a jogszabályi megőrzési kivételek függvényében – támogatnia kell a hozzáféréshez, helyesbítéshez, törléshez („elfeledtetéshez”), az adatkezelés korlátozásához, az adathordozhatósághoz és a tiltakozáshoz való jog gyakorlását:
- **Kaszkádolt törlés vs. Jogi megőrzés:** Fióktörléskor vizsgáld meg az összes adattárolót: Firebase Auth, Firestore kollekciók, Cloud Storage fájlok, külső e-mail/marketing szolgáltatók és cache-ek. Azonban a jogszabály (pl. számviteli törvény) által kötelezően megőrzendő tranzakciós/számlázási rekordokat különítsd el, korlátozd a hozzáférésüket, és csak a nem kötelező személyes profiladatokat töröld/anonimizáld.
- **Backup és törlés összehangolása:** Az élő adatbázisból való törlés nem jelenti az archivált biztonsági mentések (backups / PITR) azonnali fizikai felülírását. Dokumentáld a backup megőrzési idejét (retention) és azt a folyamatot, amely egy esetleges disaster recovery (restore) után újra érvényesíti a közben beérkezett törlési kérelmeket.
- **Anonimizálás vs. álnevesítés:** Az álnevesített (pseudonymized, pl. `uid`-hoz vagy hash-hez kötött) adat továbbra is személyes adat (GDPR hatálya alá tartozik)! Anonimizálásról csak akkor beszélhetünk, ha az érintett visszaazonosítása ésszerűen semmilyen eszközzel nem lehetséges.
- **PWA cache korlátok (`FIREBASE-PWA-001`):** A kliensoldali `clearIndexedDbPersistence(db)` és a cache-takarítás (`REF-PWA-LOGOUT-001`) **nem forenzikus adattörlés**. Megosztott eszközök és érzékeny PII esetén a tartós kliensoldali cache helyett a `memoryLocalCache()` használandó.

---

## 7. Adatmegőrzés (Retention) és a Firestore TTL szabálya (`PRIV-RET-001`)

Minden adatcsoporthoz készíts megőrzési (retention) táblázatot a `SECURITY_PROFILE.md`-ben:
- aktív felhasználói fiók és profiladatok;
- inaktív fiókok (automatikus értesítési és törlési ciklus);
- tranzakciós, bérlet-, foglalási és számviteli adatok (`passLedger`, számlák);
- biztonsági és auditnaplók (`Cloud Logging`);
- biztonsági mentések (backups / exportok);
- marketing és analitikai adatok;
- ügyfélszolgálati (support) kommunikáció.

**Normatív Firestore TTL tilalom (`FIREBASE-IDEMP-001`):** A Firestore TTL (Time-to-Live) automatikus törlési funkció technikai takarítóeszköz (pl. `_idempotency` és `_rateLimits` kollekciókhoz), nem önálló adatmegőrzési politika! **TILOS (`MUST NOT`)** Firestore TTL-t beállítani olyan üzleti, pénzügyi vagy jogilag megőrzendő naplóra (`passLedger`, számlázási rekordok), amelynek megőrzését jogszabály vagy üzleti elszámoltathatóság írja elő.

---

## 8. Analitika, cookie-k és kliensoldali követők (`PRIV-COOKIE-001`)

- A cookie-k, a `localStorage`, az SDK-azonosítók, a QR-kódos/marketing funnelek és az analitikai események használatát teljes adatfolyamként vizsgáld, ne pusztán „cookie banner” kérdésként.
- Azonosítsd minden használt tárolóelem célját, élettartamát és azt, hogy történik-e adattovábbítás harmadik félhez.
- Hozzájárulás (consent) alkalmazásakor annak előzetesnek, szabadnak, konkrétnak, tájékozottnak, egyértelműnek és bármikor visszavonhatónak kell lennie.
- **Valótlan állítások tilalma:** Soha ne állítsd egy adatkezelési tájékoztatóban vagy dokumentációban, hogy az oldal „semmilyen személyes vagy technikai adatot nem kezel”, ha a Firebase Hosting, a Cloud Functions, a Cloud Logging, az App Check (reCAPTCHA) vagy bármilyen külső erőforrás IP-címet és kérés-metaadatokat dolgoz fel!
- A „cookie-mentes” (cookie-free) saját analitika sem automatikusan jogalap-mentes; a ténylegesen tárolt és továbbított adatokat (pl. hash-elt azonosítók, forrásparaméterek) a GDPR 6. cikke (pl. jogos érdek vs. hozzájárulás) alapján kell értékelni.

---

## 9. Adatvédelmi incidenskezelés (`PRIV-INC-001`)

- Legyen dokumentált folyamat a biztonsági és adatvédelmi incidensek felismerésére, izolálására és eszkalálására.
- Incidens esetén őrizd meg a forenzikus bizonyítékokat és naplókat (`Cloud Logging`), de minimalizáld a kompromittált személyes adatok további másolását.
- Értékeld, hogy történt-e a GDPR 4. cikk 12. pontja szerinti adatvédelmi incidens (bizalmasság, sértetlenség vagy rendelkezésre állás sérülése), hány érintettet és milyen adatkategóriákat érint, és milyen kockázatot jelent a természetes személyek jogaira és szabadságaira.
- A felügyeleti hatósági (Magyarországon a **NAIH** felé a tudomásszerzéstől számított **72 órán belüli** – GDPR 33. cikk) bejelentési és a magas kockázat esetén szükséges érintetti tájékoztatási (GDPR 34. cikk) kötelezettséget a konkrét tényállás és emberi/jogi döntés alapján teljesítsd; az AI-ágens önállóan nem küldhet hatósági bejelentést!
- Incidens után végezz gyökérok-elemzést (root cause analysis), és frissítsd a biztonsági kontrollokat.

---

## 10. Külső adatfeldolgozók és AI-szolgáltatók (`PRIV-PROC-001`)

- Minden külső szolgáltatónál (Google Cloud/Firebase, e-mail küldő, fizetési szolgáltató, tárhely/backup szerver, AI/LLM szolgáltató) ellenőrizd az adatfeldolgozói szerepet, az adatfeldolgozói szerződést (DPA), az alfeldolgozókat, az adatközpont régióját (EU/EGT), az esetleges harmadik országbeli adattovábbítási garanciákat és a megőrzési időt.
- AI / LLM szolgáltatásba kizárólag a feladathoz feltétlenül szükséges, minimalizált adat küldhető (`API-LLM-001`).
- Ellenőrizd az AI-szolgáltató adatfelhasználási (model training tiltása!), megőrzési és adatrezidencia (data residency) feltételeit.
- A promptokban és feltöltött fájlokban lévő személyes adatokat lehetőség szerint szűrd (redaction) vagy álnevesítsd.

---

## 11. Adatvédelmi ellenőrzőlista (`PRIV-CHK-001`)

- [ ] adatleltár és adatfolyam-diagram elkészült a `SECURITY_PROFILE.md`-ben (`PRIV-INV-001`);
- [ ] minden adatcsoporthoz rögzített cél, jogalap és megőrzési idő (retention) tartozik (`PRIV-GDPR-001`, `PRIV-RET-001`);
- [ ] adatminimalizálás és privát alapértelmezések érvényesítve (`PRIV-PBD-001`);
- [ ] a Firestore TTL nincs üzleti/számviteli `ledger` kollekcióra beállítva (`PRIV-RET-001`);
- [ ] az adatfeldolgozók, régiók és adattovábbítások áttekintve (`PRIV-PROC-001`);
- [ ] az érintetti jogok és a kaszkádolt törlési + backup folyamat összehangolva (`PRIV-RIGHTS-001`);
- [ ] a logok, az analitika és az AI promptok PII-szűrése ellenőrizve (`PRIV-SEC-001`);
- [ ] a DPIA szükségessége mérlegelve (`PRIV-PBD-001`);
- [ ] az incidenskezelési és NAIH eszkalációs folyamat ismert (`PRIV-INC-001`);
- [ ] az adatkezelési tájékoztató és a consent flow emberileg/jogilag ellenőrizve (`PRIV-COOKIE-001`).

---

## 12. Hivatalos források (`PRIV-REF-001`)

A pontos verzióstátuszt és ellenőrzési dátumokat (`last_verified`) lásd a `10_SOURCE_REGISTER.md` modulban:
- GDPR (EU) 2016/679: https://eur-lex.europa.eu/eli/reg/2016/679/oj
- EDPB Guidelines 4/2019 on Article 25 (Data Protection by Design and by Default): https://www.edpb.europa.eu/our-work-tools/our-documents/guidelines/guidelines-42019-article-25-data-protection-design-and_en
- EDPB 2026 Summary of Data Protection by Design and by Default: https://www.edpb.europa.eu/system/files/2026-02/edpb-summary-gdpr-data-protection-design-default_en.pdf
- ENISA: https://www.enisa.europa.eu/
- Nemzeti Adatvédelmi és Információszabadság Hatóság (NAIH): https://www.naih.hu/