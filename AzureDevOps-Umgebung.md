# Azure DevOps Umgebungen (Paketierung)

Kurzreferenz, damit nicht jedes Mal erklärt werden muss, welche DevOps-Organisation wofür da ist.

## Übersicht

| Organisation / Projekt | URL | Status | Zweck |
|---|---|---|---|
| **ClientAppDeployment / Apps** | https://dev.azure.com/ClientAppDeployment/Apps | **NEU / aktiv** | Aktuelle Ablage für die Paketierung. **Alle neuen App-Pakete werden hier hochgeladen.** |
| **FBAI / eduAPP** | https://dev.azure.com/FBAI/eduAPP | **ALT / Archiv** | Alte Paketierungs-Ablage. Nichts Neues mehr hochladen — aber praktisch als Referenz, weil dort z. T. noch Logs und Historie (z. B. Pipeline-Runs, alte Pakete) erhalten sind. |

## Regeln

- **App-Paket hochladen / neue Paketierung** → immer **ClientAppDeployment**.
- **Nachschauen, wie etwas früher gemacht wurde** (Logs, alte Pakete, Historie) → ggf. in **FBAI/eduAPP** suchen.
- **PCDS / Nextcloud-Tenant-Themen** sind ein separates Thema und gehören **nicht** in ClientAppDeployment.

## Zugriff aus Claude-Code-Cloud-Sessions

Die Cloud-Umgebung hat standardmässig nur GitHub-Zugriff, kein Azure-DevOps-Credential. Zwei Wege:

1. **Device-Code-Login (interaktiv, bevorzugt):** Claude startet den Microsoft-Device-Code-Flow (Client-ID der Azure CLI, Scope `499b84ac-1321-427f-aa17-267ca6975798/.default` = Azure-DevOps-Ressource), zeigt URL (https://microsoft.com/devicelogin) + Code an, der Benutzer loggt sich im Browser ein. Das Token gilt dann für die Session (~1 h, mit Refresh-Token verlängerbar).
2. **PAT als Umgebungs-Secret (dauerhaft):** Variable `AZURE_DEVOPS_PAT` in den Umgebungseinstellungen hinterlegen (Scope: Code Read bzw. Read & Write, „All accessible organizations").

Verwendung (Token oder PAT):
- REST API: `curl -H "Authorization: Bearer $TOKEN" "https://dev.azure.com/{org}/{project}/_apis/git/repositories?api-version=7.1"` (bei PAT: `curl -u ":$PAT" ...`)
- Git-Clone: `git clone "https://pat:$PAT@dev.azure.com/{org}/{project}/_git/{repo}"`
