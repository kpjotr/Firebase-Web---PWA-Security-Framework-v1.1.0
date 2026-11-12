======================================================================
            FSF SKILL (v1.1.0) – NAPI HASZNÁLATI PUSKA
======================================================================

NEM KELL FEJBŐL TUDNOD A KÓDOKAT (CORE-*, FIREBASE-* stb.)!
Az AI-nak csak ezt a 3 alapinfót add meg saját szavaiddal:

1. MI A TÉT? (Kockázati szint)
   - L1 (Saját/belső kis app): Nincs ügyfél-adat, nincs fizetés/bérlet.
   - L2 (Ügyfél PWA / Üzleti app): Van ügyfél, bérlet, foglalás, PII.
   - L3 (Kritikus): Pénzügyi vagy egészségügyi/szabályozott rendszer.

2. KI REGISZTRÁLHAT? (Onboarding)
   - A modell: Csak én (admin) hozok létre fiókot a konzolban/backendben.
   - B modell: Zárt, csak meghívóval vagy előzetes listával (allowlist).
   - C modell: Nyilvános regisztráció bárkinek (ügyfél PWA).

3. MI A FELADAT?
   - Új projekt / Új funkció tervezése / Meglévő kód auditálása.

----------------------------------------------------------------------
        4 KÉSZ MONDATSABLON (CSAK ÍRD ÁT A [SZÖGLETES] RÉSZT!)
----------------------------------------------------------------------

[1] ÚJ PROJEKT INDÍTÁSAKOR (Alapozás + SECURITY_PROFILE.md):
"Új Firebase projektet indítok: [miről szól röviden].
Kockázat: [L1 saját / L2 ügyfél PWA], regisztráció: [A: admin / B: meghívós / C: nyilvános].
Készítsd el az FSF skill szerint a SECURITY_PROFILE.md-t és az alapokat!"

[2] ÚJ FUNKCIÓ FEJLESZTÉSEKOR (Két ügynökös terv + VS Code prompt):
"A [Projekt neve] appban ([L1 / L2]) szeretném megcsinálni a [funkció leírása, pl. bérletlevonás / napló mentés] részt.
Az FSF skill és a két ügynökös munkafolyamat szerint készítsd el a biztonsági tervet, a handoffot és az angol VS Code kódoló promptot!"

[3] MEGLÉVŐ PROJEKT VAGY FÁJLOK AUDITÁLÁSAKOR (Security Review):
"Auditszemmel nézd át a csatolt [firestore.rules / Cloud Functions / frontend] fájlokat az FSF skill [L1 / L2] szintje alapján!
Keresd meg a jogosultsági, validációs és költség-kockázatokat, és adj javítási tervet + angol VS Code javító promptot!"

[4] KÓDOLÁS UTÁNI ELLENŐRZÉSEKOR (Amikor a VS Code végzett):
"A VS Code elkészült a feladattal. Itt a módosított kód / git diff és a teszt eredménye.
Ellenőrizd le független Reviewer szerepben az FSF skill szerint, hogy minden biztonsági szabály teljesült-e!"

----------------------------------------------------------------------
TIPP: Ha valamit elfelejtesz megadni, az FSF skill szabályzata miatt
az AI úgyis visszakérdez a hiányzó adatra, mielőtt kódot generálna!
======================================================================