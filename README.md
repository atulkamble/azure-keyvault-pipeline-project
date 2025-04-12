# azure-keyvault-pipeline-project

Here’s a **full Azure Key Vault project** integrated with **Azure Pipelines** for securely retrieving secrets during deployment. This example demonstrates:

- Creating a Key Vault and storing secrets
- Accessing the Key Vault via a Service Connection in Azure DevOps
- Using the secrets in a pipeline
- Deploying a sample app (Node.js used here, you can replace it as needed)

---

## 🔐 Azure Key Vault + Azure Pipelines – Full Project Setup

---

### 📁 Project Structure (`azure-keyvault-pipeline-demo`)
```
azure-keyvault-pipeline-demo/
│
├── app/
│   ├── index.js
│   └── package.json
│
├── azure-pipelines.yml
└── README.md
```

---

### 1️⃣ Sample Node.js App (`app/index.js`)

```js
const http = require('http');
const port = process.env.PORT || 3000;
const secret = process.env.MY_SECRET || "No secret found!";

http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end(`Secret is: ${secret}`);
}).listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

---

### 2️⃣ package.json

```json
{
  "name": "azure-keyvault-demo",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {}
}
```

---

### 3️⃣ Azure Key Vault Setup (via CLI or Portal)

```bash
az keyvault create --name myKeyVaultDemo --resource-group myResourceGroup --location eastus
az keyvault secret set --vault-name myKeyVaultDemo --name "MY-SECRET" --value "ThisIsASecret"
```

---

### 4️⃣ Azure DevOps – Service Connection

- Create a Service Connection with Azure RM that has **Key Vault access**.
- Enable **"Allow access to all pipelines"**.

---

### 5️⃣ azure-pipelines.yml

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  PORT: 3000

stages:
- stage: Build
  jobs:
  - job: BuildApp
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '16.x'
      displayName: 'Install Node.js'

    - script: |
        cd app
        npm install
      displayName: 'Install Dependencies'

- stage: Deploy
  variables:
    - group: keyvault-variable-group  # Link Key Vault from Library

  jobs:
  - job: DeployApp
    steps:
    - task: AzureKeyVault@2
      inputs:
        azureSubscription: 'Your-Service-Connection-Name'
        KeyVaultName: 'myKeyVaultDemo'
        SecretsFilter: '*'
        RunAsPreJob: true

    - script: |
        echo "Secret from KeyVault is: $(MY-SECRET)"
        cd app
        PORT=$(PORT) MY_SECRET=$(MY-SECRET) node index.js &
        sleep 10
      displayName: 'Run App with Secret'
```

---

### 6️⃣ Azure DevOps Library Variable Group (Optional)
- Go to Pipelines → Library → Add Variable Group → Link to Key Vault.

---

### ✅ Output
When deployed, the console and browser will show:

```
Secret is: ThisIsASecret
```

---

Would you like this project in a GitHub repo format or zip file too?
