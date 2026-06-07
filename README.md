# Jämförelse av systemsäkerhet: Ubuntu 22.04 LTS & CIS Benchmarks

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)
[![OS: Ubuntu](https://img.shields.io/badge/OS-Ubuntu%2022.04%20LTS-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Standard: CIS Benchmarks](https://img.shields.io/badge/Standard-CIS%20Level%201%20%26%202-005b9f)](https://www.cisecurity.org/)

**Utvärdering av säkerhetsvinster och påverkan vid systemhärdning.** Detta repository innehåller dokumentation, mätdata och utvärderingsrapporter från mitt examensarbete inom programmet **IT-säkerhetsspecialist** på TUC Yrkeshögskola. Projektet utforskar den kritiska balansen mellan strikt systemsäkerhet och operativ tillgänglighet i en Linux-miljö.

> 📖 **Läs Hela Uppsatsen (PDF):** [Jämförelse av systemsäkerhet_Examensarbete_ITS24STO.pdf](dokument/Jämförelse_av_systemsäkerhet_Examensarbete_ITS24STO.pdf)

---

## 📑 Innehåll
- [Beskrivning](#-beskrivning)
- [Metod](#️-metod)
- [Säkerhetsmätvärden & Resultat](#-säkerhetsmätvärden--resultat)
- [Påverkan & konsekvenser](#️-påverkan--konsekvenser)
- [Mina Slutsatser](#-mina-slutsatser)
- [OpenSCAP Rapporter](#-openscap-rapporter)

---

## 📝 Beskrivning
En standardinstallation av Linux prioriterar initial funktionalitet framför maximal säkerhet, vilket lämnar onödiga attackytor öppna för angripare. Syftet med detta projekt var att implementera *best practices* från Center for Internet Security (CIS) Benchmarks för Ubuntu 22.04 LTS och kvantitativt mäta säkerhetsvinsterna jämfört med en "vanilla" installation. 

Vidare dokumenterar projektet den tekniska och operativa konflikter som dessa åtgärder orsakar i den dagliga administrationen och driften.

## 🛠️ Metod
Utvärderingen genomfördes i en isolerad, virtualiserad miljö med två identiska servrar för att säkerställa hög reliabilitet:

* **Server A (Baseline):** Standardinstallation av Ubuntu 22.04 LTS Server.
* **Server B (Hardened):** Ubuntu 22.04 LTS Server härdad både manuellt och via automatiserade skript enligt CIS Benchmarks.

**Använda granskningsverktyg:**
* **OpenSCAP (`oscap`):** Användes för automatiserad konfigurationsgranskning mot den officiella XML-profilen `ssg-ubuntu2204-ds.xml`.
* **Lynis:** Användes som ett kompletterande verktyg för att validera härdningsindexet.


## 📊 Säkerhetsmätvärden & Resultat
Härdningsprocessen minskade serverns attackyta drastiskt. Skanningarna före och efter härdningen genererade följande resultat:

| Mätvärde | Server A (Baseline) | Server B (Härdad) | Skillnad |
| :--- | :--- | :--- | :--- |
| **OpenSCAP Score (%)** | 67,97 % | 92,27 % | **+ 24,3 %** |
| **Godkända regler (Pass)** | 228 | 350 | **+ 122** |
| **Underkända regler (Fail)** | 115 | 7 | **- 108** |
| **Lynis Hardening Index** | 62 | 84 | **+ 22** |
| **Administrativ tid** | ~15 min (installation) | ~150 min (härdning) | **+ 135 min** |

## ⚠️ Påverkan & Konsekvenser
Även om OpenSCAP-poängen ökade med över 24 procent, påverkade härdningen omedelbart systemets tillgänglighet och administration. De mest kritiska observationerna var:

1.  **Blockerad DHCP-trafik:** Implementeringen av CIS-regel 3.4.2.1, som konfigurerar brandväggen UFW med policyn "Default Deny Outgoing", blockerade serverns förmåga att förnya sin IP-adress via DHCP vid omstart. En manuell regel för att tillåta UDP-trafik på port 67/68 krävdes för att återställa nätverksfunktionen.

2.  **Bruten logghantering (SCP-problem):** Automatiserade åtgärdsskript begränsade filrättigheterna så strikt att export av `root`-ägda loggfiler via Secure Copy Protocol (SCP) blockerades. Ändring av filägarskap med `chown` krävdes för att överhuvudtaget kunna extrahera audit-rapporterna.

3.  **Ökad risk för SSH-utlåsning:** Systemet härdades genom att begränsa `MaxAuthTries` till 4 försök och sätta LogLevel till `VERBOSE`. Detta resulterade i en utökad loggmängd och en påtagligt högre risk för automatisk utlåsning av legitima administratörer.

4.  **Tidskrävande administration:** Medan en standardinstallation tog 15 minuter, krävde felsökningen och den manuella härdningen av endast 8 CIS-regler hela 90 minuters aktiv arbetstid. Totalt krävde processen 150 minuter.


## 💡 Mina Slutsatser
* **Härdning är ingen checklista:** Att köra automatiserade skript blint i produktion utan kännedom om beroenden kan leda till driftstopp (DoS), vilket exemplet med den blockerade DHCP-tjänsten visade.

* **CIS Level 1 vs Level 2:** Resultaten bekräftar ramverkets egen vägledning – Level 1 är en utmärkt driftvänlig baslinje. Level 2 medför dock allvarlig administrativ friktion och bör reserveras exklusivt för system med exceptionella säkerhetskrav.

* **Least Privilege vs Hanterbarhet:** Överlappande skyddslager försvårar legitima administrativa uppgifter. Försvaret blir rent tekniskt starkare, men i praktiken betydligt mer svår att hantera.

* **Automation är kritisk för skalbarhet:** Den massiva tidsåtgången 150 min per server, bevisar att manuell härdning är ineffektivt i en miljö med flera servrar. Verktyg för konfigurationshantering, såsom Ansible, rekommenderas för att framtidssäkra och underhålla CIS-efterlevnad.

## 🔗 OpenSCAP Rapporter
För att utforska granskningsdatan i detalj, klicka på länkarna nedan för att rendera de genererade HTML-rapporterna direkt i webbläsaren:

* 📊 **[Visa Baseline rapport (Server A)](https://althany.github.io/Server-Hardening-Analysis/rapporter/Server_A-report_baseline.html)**
* 🧰 **[Visa Manuell härdnings rapport (Server B)](https://althany.github.io/Server-Hardening-Analysis/rapporter/Server_B-report_hardened_manuall.html)**
* 🛡️ **[Visa Auto härdnings rapport (Server B)](https://althany.github.io/Server-Hardening-Analysis/rapporter/Server_B-report_auto_hardened.html)**

---
**Författare:** Abdullah Althany  
**Program:** IT-säkerhetsspecialist, TUC Yrkeshögskola  
**Datum:** Januari - Maj 2026
