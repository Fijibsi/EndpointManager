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

Die Cloud-Umgebung hat standardmässig nur GitHub-Zugriff. Für Azure DevOps braucht die Session einen **Personal Access Token (PAT)** als Umgebungs-Secret:

- Variablenname: `AZURE_DEVOPS_PAT`
- Empfohlene Scopes: mindestens **Code (Read)**; für Uploads/Pushes **Code (Read & Write)**. Der PAT muss für beide Organisationen gelten (bei der PAT-Erstellung „All accessible organizations" wählen) oder es braucht je einen PAT pro Organisation.
- Verwendung:
  - REST API: `curl -u ":$AZURE_DEVOPS_PAT" "https://dev.azure.com/{org}/{project}/_apis/git/repositories?api-version=7.1"`
  - Git-Clone: `git clone "https://pat:$AZURE_DEVOPS_PAT@dev.azure.com/{org}/{project}/_git/{repo}"`
