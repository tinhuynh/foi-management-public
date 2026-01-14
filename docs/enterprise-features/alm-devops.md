# ALM & DevOps Discipline — Azure DevOps CI/CD (Dev → Test)

This project implements a controlled ALM process using Azure DevOps for exporting managed solutions from DEV and deploying them into TEST with environment-specific configuration. The goal is to demonstrate enterprise-grade governance, repeatability, and reliable solution promotion.

---

## Architecture Overview

```mermaid
flowchart LR
    DEV[DEV (Unmanaged)]
    ADO[Ado Build Pipeline]
    RELEASE[ADO Release Pipeline]
    TEST[Test (Managed)]

    DEV -->|Export Managed Solution| ADO
    ADO -->|Publish Artifacts| RELEASE
    RELEASE -->|Import Managed Solution| TEST
```

---

## Build Pipeline (CI)

The CI pipeline performs:

- Export of **managed** solution from DEV  
- Version stamping for controlled releases  
- Automatic generation of `deploymentSettings.json`  
- Packaging artifacts into the `drop` folder  
- Publishing artifacts via Power Platform Build Tools  

**Build output structure:**

```
drop/
 ├── FOI_Management_Managed.zip
 └── deploymentSettings.json
```

---

## Release Pipeline (CD)

The CD pipeline:

- Retrieves the `drop` artifact  
- Authenticates using a service principal  
- Imports the managed solution into TEST  
- Applies environment variables  
- Applies connection references  
- Logs all deployment operations for auditing  

---

## Environment Strategy

| Environment | Purpose | Customisations |
|------------|---------|----------------|
| **DEV**    | Development & experimentation | Unmanaged |
| **TEST**   | QA, validation, stakeholder review | Managed only |
| **PROD** *(future)* | Production | Managed only |

TEST is locked down as a managed-only environment to ensure deployment consistency and prevent accidental changes.

---

## Deployment Settings (Environment Separation)

Example structure:

```json
{
  "EnvironmentVariables": [
    {
      "SchemaName": "new_EnvironmentSpecificVariable",
        "Value": "https://test.sharepoint.com/sites/foi",
        "DefaultValue": "https://test.sharepoint.com/sites/foi",
        "Name": {
          "Default": "FOI_SharePointSiteURL",
          "ByLcid": {
            "1033": "FOI_SharePointSiteURL"
          }
        },
        "TypeId": 100000000,
        "IsRequired": false
    }
  ],
  "ConnectionReferences": [
    {
      "LogicalName": "shared_sharepointonline_connection",
      "ConnectionId": "<TEST-CONNECTION-ID>",
      "ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline"
    }
  ]
}
```

This enables environment-specific configuration without modifying the solution.

---

## Security & Governance Controls

- Service principal authentication  
- Least-privilege access model  
- Deployments into TEST are restricted to CI/CD pipelines, ensuring controlled promotion and traceability

---

## Screenshots

![Build Success](../../assets/screenshots/35-build-success.png)
*Build Pipeline (DEV) - CI run completed with managed solution export and artifact packaging.*
<br><br>

![Release Success](../../assets/screenshots/36-release-success.png)
*Release Pipeline (TEST) - Managed solution imported successfully via CD.*
<br><br>

![Service Connection](../../assets/screenshots/37-service-connection.png)
*Secure service principal used by Azure DevOps to access the DEV environment.*
<br><br>

![Artifact Structure](../../assets/screenshots/38-artifact-structure.png)
*Managed solution and deployment settings packaged for release.*