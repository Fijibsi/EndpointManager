# VSBELP-Drucker-Analyse

Stand: 2026-10-01 · Durchsucht: GitHub `fijibsi/EndpointManager` (alle Branches, Historie, Archive) und Azure DevOps `ClientAppDeployment/Apps` (alle 144 Repos, Paketinhalte inkl. Treiber-INFs)

## Kernergebnis

**Die Schulen Belp (VSBELP) drucken über Printix — es existiert kein klassisches VSBELP-Drucker-Treiberpaket.**

Das einzige Druck-Paket mit VSBELP-Bezug ist `0469_PrintiX_Client`: Der Tungsten-Printix-Client ist fest an den Cloud-Tenant **`schulen-belp.printix.net`** gebunden (Tenant-ID `3e887af2-9a69-4fb3-8c58-b25d9e119195`). Drucker-Queues und Treiber werden zur Laufzeit aus der Printix-Cloud provisioniert; deshalb gibt es — anders als bei anderen Schulen — keine per-Queue- oder Treiber-Pakete für Belp.

Wichtig zur Abgrenzung: **VS4917 ist NICHT VSBELP.** Die sechs `VS4917-Printer-*`-Pakete gehören zu einem anderen Schul-Mandanten (Belege unten).

## 1. VSBELP-Identität (Beleg)

`0483_Untis-VSBELP` → `Invoke-AppDeployToolkit.ps1`:

> „Lizenzdaten: Kundennummer 48718 / **Schulen Belp** / **3123 Belp**" (Lizenzen SUC-042 / AQR-265 / XLP-515), Datenbank `Files/Database/Belp_2026_10_03 ry.untis`

VSBELP = Schulen Belp (3123 Belp, Kanton Bern), paketiert unter dem Dach **EDUBERN** (AppVendor `EDUBERN`, Autoren `@edubern.ch` / `@aa.edubern-cloud.ch`, PSADT-Template-Config: CompanyName „Bildungs- und Kulturdirektion des Kantons Bern").

## 2. Das VSBELP-Druck-Paket: `0469_PrintiX_Client`

| Aspekt | Befund |
|---|---|
| Payload | `CLIENT_{schulen-belp.printix.net}_{3e887af2-…}_x64.MSI` (~151 MiB) — Tenant-MSI, Umbenennen verboten (bricht Tenant-Registrierung) |
| Client | Tungsten Printix Client **2025.4.0.103** (MSI-Wrapper 11.0.53.0 um Inno-Setup 6.7.0, Build 2026-01, signiert) |
| Wrapper | PSADT 4.1.7, `AppVendor='Printix'`, `AppVersion='latest'`, Autor-Datum 2026-02-06 |
| Install | `Start-ADTMsiProcess /QN REBOOT=ReallySuppress`; keine `Add-Printer*`-/`pnputil`-Aufrufe — Queues/Treiber kommen aus der Cloud |
| Uninstall | hartkodiert `C:\Program Files\printix.net\Printix Client\unins000.exe /VERYSILENT` (Inno, nicht msiexec) |
| Detection | `Test-Path 'C:\Program Files\printix.net\Printix Client\PrintixClient.exe'` |
| Treiber im Paket | keine (keine .inf/.cat im ganzen Repo — by design bei Printix) |

### Risiken / Verbesserungen (PrintiX-Paket)

1. **Uninstall-Inkonsistenz**: Installation per MSI, Deinstallation per Inno-`unins000.exe` → kann verwaisten ARP-/MSI-Eintrag hinterlassen. Besser: `Uninstall-ADTApplication` über den ProductCode `{F0D257C7-442A-405C-9CD6-B4A477C180B5}`.
2. **Hartkodierte Pfade** in Uninstall und Detection — brechen bei Pfadänderung durch Printix.
3. **`AppVersion='latest'`** statt `2025.4.0.103` — erschwert Inventar und Supersedence.
4. Tote Variablen `$MsiFileName`/`$UninstallExe` (definiert, nie verwendet); `AppLang='EN'` bei deutschen Prompts.
5. Keine Credentials/IPs im Paket; MSI signiert; Client-Build aktuell (2026-01).

## 3. Weitere VSBELP-Funde (kein Treiberpaket, aber Belp-Bezug)

| Repo | Fund |
|---|---|
| `0467_Predata_winMedio` | Mandanten-Ordner `Files/Mandanten/VSBELP/MandantInfo.xml` — Bibliothekssoftware (Mandanten „Biblio Neumatt", „Biblio Mühlematt", „Materialverwaltung"); dazu Quittungsdrucker-Vorlagen (Hersteller-Standard, kein eigener Treiber) |
| `0452/0453/0454_Affinity_*` | Ordner `Files/Schulen Belp/Affinity2.defaults` — Belp-spezifische App-Defaults, kein Drucker |
| `0483_Untis-VSBELP` | Untis 2026 für Schulen Belp (Stundenplan, kein Druckbezug) |
| `0047_UniFLOW_Client*` | **Negativ-Beleg**: Hostname-Weiche kennt `aid*, bernedu*, wp*, vru*, nbe*, bbz*, bemed*, bfb*, bff*, bwz*, bfsl*, gymo*, gymb*, vs4917*, bze*` — **kein `belp*`/`vsbelp*`** → Belp nutzt kein uniFLOW |

## 4. Abgrenzung: die anderen Drucker-Pakete in ClientAppDeployment/Apps

Vollständige Drucker-Paket-Familie der Organisation (14 Repos, alle inhaltlich durchsucht, keine VSBELP-Referenz außer Printix):

| Schule/Mandant | Repos | Technik |
|---|---|---|
| **VSBELP (Belp)** | `0469_PrintiX_Client` | Printix-Cloud, Tenant `schulen-belp.printix.net` |
| **VS4917** (andere Schule!) | `0515_VS4917-Driver` + `0516`–`0521_VS4917-Printer-*` | Canon GPlus UFR II V3.40 (CNLB0M.INF, DriverVer 12/24/2025) + 6 Raum-Queues: Ratszimmer 172.16.3.11, Büro-Lehrerzimmer .12, Bibliothek .13, Unterstufe-Gang .14, Schulleitung .16, Tagesschule 192.168.50.105 (RAW 9100) |
| **VSTB** | `0033_VSTB-PrinterDrivers` | Toshiba e-STUDIO Universal 2 (7.204.4408.29, 2019) + Generic XL (2.13.1.0, 2016), reines `pnputil`-Staging |
| **BEMED** | `0485/0486_BEMED_Printer-*-PMF-G15-01` | OKI Universal PCL 5 (1.8.1.0, 07/2024) + Queue PMF-G15-01 → 10.163.136.32:9100 |
| **Multi-Schule** | `0047_UniFLOW_Client(_Converted_PSADT-V4.1.7)`, `0385_MEAP` | uniFLOW SmartClient 2025.4.x, 10 Tenant-MSIs (`<code>.eu.uniflowonline.com`), Canon Generic Plus UFR II 3.31 im MSI-CAB; `0385_MEAP` = getarntes CloudPrint-MEAP-Paket (DSUStarter.exe) |

### Warum VS4917 ≠ VSBELP (Kritiker-Prüfung)

- Die Portfolio-Konvention nutzt für Belp das Kürzel `VSBELP` (siehe `0483_Untis-VSBELP`); die Druckerpakete heißen aber `VS4917-*` und referenzieren durchgehend „Mandant VS4917" / „an der Schule VS4917" (READMEs, Intune-Gruppen `C-AD-VS4917-FA-…`).
- `Belp` kommt in keiner Datei der 7 VS4917-Repos vor; `4917` kommt nirgends im Untis-VSBELP-Repo vor. Belp hat PLZ 3123, nicht 4917.
- Gemeinsame Autoren (Valmir Rusiti, Nicolas Berger) belegen nur dieselbe Paketierungs-Organisation EDUBERN, nicht dieselbe Schule.

## 5. Suchabdeckung und verbleibende Lücken

**Durchsucht**: GitHub-Repo `fijibsi/EndpointManager` komplett (Worktree, alle 3 Branches, gesamte Historie inkl. Pickaxe, alle Archive entpackt — dort nur `System.Printing.dll` aus .NET-3.5-FOD-CABs, kein Treiberpaket) · Azure DevOps `ClientAppDeployment/Apps`: alle 144 Repos (62 leer), 14 Drucker-/Kontext-Repos inhaltlich tief (Skripte, Configs, INFs, MSI-Innereien), übrige 68 per vollständigem Pfad-Sweep mit Verifikation der Treffer.

**Nicht prüfbar** (Lücken):

1. **FBAI/eduAPP** (Alt-Archiv-Organisation): HTTP 401 — das hinterlegte PAT deckt die FBAI-Organisation nicht ab. Falls dort historische Belp-Druckerpakete liegen, sind sie unsichtbar. → PAT mit „All accessible organizations" bzw. FBAI-Zugriff hinterlegen.
2. **Autoritative Mandanten-Zuordnung** (SharePoint-Liste „App-Portfolio_alle AttributeNew", maßgebliches Excel) ist aus den Repos nicht erreichbar — der VS4917≠VSBELP-Schluss basiert auf Repo-Evidenz (sehr konsistent, aber kein 100 %-Ausschluss).
3. **Azure-DevOps-Volltextsuche** (`almsearch.dev.azure.com`) wird vom Proxy blockiert (403) — Pfad-Sweep + gezielte Inhaltsprüfung als Ersatz.
4. `GitHub_EDUBERN_Org_Archive` und `0290_Klett-…` (je 2,6 GB) liefern im Default-Branch fast keine Pfade (Inhalt vermutlich in Packfiles/anderen Branches) — nicht durchsuchbar per items-API.

## 6. Empfehlungen

1. **Printix-Paket härten** (Punkte in Abschnitt 2): Uninstall über ProductCode, Versionsnummer statt `latest`, tote Variablen entfernen.
2. Falls eine vollständige Belp-Druckerlandschaft dokumentiert werden soll: Queues/Drucker direkt im **Printix-Admin-Portal** (`schulen-belp.printix.net`) auslesen — die Repos enthalten diese Informationen prinzipbedingt nicht.
3. FBAI-Zugriff für künftige Archiv-Recherchen ergänzen (siehe Lücke 1).
