# 🚗 Aplicação de Aluguel de Carros Cloud Native com Azure

Este guia mostra como criar uma aplicação totalmente **cloud native** usando apenas o **portal do Azure**, sem necessidade de CLI. O projeto utiliza:

- Azure Functions  
- Key Vault  
- Application Insights  
- Container Apps  
- Service Bus  
- Logic App  
- PostgreSQL Flexible Server  

---

## ✅ Etapas

### 1. 🔹 Criar o Resource Group

1. Acesse [portal.azure.com](https://portal.azure.com)
2. Menu lateral > **Resource groups** > **+ Create**
3. Configure:
   - **Subscription**
   - **Resource Group Name**: `car-rental-rg`
   - **Region**: `East US`
4. Clique em **Review + create** > **Create**

---

### 2. 🔹 Criar o Container Registry (ACR)

1. Pesquise **"Container Registry"** > **+ Create**
2. Configure:
   - Resource Group: `car-rental-rg`
   - Name: `carrentalregistry`
   - Region: mesma do RG
   - SKU: `Basic`
3. Clique em **Review + create** > **Create**

---

### 3. 🔹 Criar o Azure Container Apps Environment

1. Pesquise **"Container Apps"** > **+ Create**
2. Clique em **"Create new" Environment**
   - Nome: `carrental-env`
   - Região: mesma do projeto
3. Finalize o ambiente e prossiga para o próximo passo

---

### 4. 🔹 Criar os Container Apps (Frontend e Backend)

Para cada app (backend e frontend):

1. Pesquise **"Container Apps"** > **+ Create**
2. Configure:
   - Resource Group: `car-rental-rg`
   - App name: `carrental-backend` (ou `carrental-frontend`)
   - Environment: `carrental-env`
   - Ingress:
     - Backend: porta 8000
     - Frontend: porta 80
3. Container:
   - Source: **Azure Container Registry**
   - Imagem: `carrental-backend:latest`
4. Adicione variáveis de ambiente:
   - `DATABASE_URL`
   - `SERVICEBUS_CONNECTION_STRING`
5. **Review + create** > **Create**

---

### 5. 🔹 Criar o PostgreSQL Flexible Server

1. Pesquise **"Azure Database for PostgreSQL Flexible Server"** > **+ Create**
2. Configure:
   - Resource Group: `car-rental-rg`
   - Name: `carrental-db`
   - Admin: `vitor`
   - Senha segura
   - Versão: 13
   - Plano: `Basic`
3. Após criado:
   - Vá em **Networking** > **Allow public access** (para testes)
   - Copie a connection string

---

### 6. 🔹 Criar o Azure Key Vault

1. Pesquise **"Key Vault"** > **+ Create**
2. Configure:
   - Resource Group: `car-rental-rg`
   - Name: `carrental-kv`
   - Região: mesma
3. Após criado:
   - Vá em **Secrets** > **+ Generate/Import**
     - `PostgresConnectionString`
     - `ServiceBusConnectionString`
4. Em **Access policies**, adicione acesso para seus Container Apps e Function App

---

### 7. 🔹 Criar o Azure Service Bus

1. Pesquise **"Service Bus"** > **+ Create**
2. Configure:
   - Resource Group: `car-rental-rg`
   - Namespace: `carrental-sb`
   - Tier: `Standard`
3. Após criado:
   - Vá em **Queues** > **+ Queue**: `rental-requests`
   - Pegue a connection string em **Shared access policies**

---

### 8. 🔹 Criar o Function App

1. Pesquise **"Function App"** > **+ Create**
2. Configure:
   - Resource Group: `car-rental-rg`
   - Name: `carrental-functions`
   - Runtime: Python ou Node.js
   - Storage Account: novo
   - Application Insights: ativado
3. Após criação:
   - Vá em **Functions** > **+ Add**
   - Template: **Service Bus Queue trigger**
     - Fila: `rental-requests`
     - Connection string do Service Bus
4. Implemente a lógica de reserva, notificações etc.

---

### 9. 🔹 Criar o Application Insights

> Se você ativou durante o Function App, já está ok.

Caso contrário:

1. Pesquise **"Application Insights"** > **+ Create**
2. Configure:
   - Resource Group: `car-rental-rg`
   - Nome: `carrental-insights`
   - Tipo: Web
3. Conecte à Function App e/ou Container App via Instrumentation Key

---

### 10. 🔹 Criar Logic App (ex: envio de e-mails)

1. Pesquise **"Logic Apps"** > **+ Create**
2. Configure:
   - Resource Group: `car-rental-rg`
   - Nome: `process-reservation`
3. Após criado:
   - No Designer:
     - Gatilho: **When a message is received in a Service Bus Queue**
     - Fila: `rental-requests`
     - Ações:
       - Enviar e-mail com status da reserva
       - Enviar mensagem para Teams/Telegram

---

## 📌 Extras

- 🎯 **Melhorar segurança**: Usar Managed Identity nos apps
- 📦 **Dockerfile**: para backend e frontend com push para o ACR
- 🔄 **CICD**: usar GitHub Actions com deploy automático para Container Apps

---

## 🧭 Próximos passos

- Desenhar arquitetura (diagrama)
- Escrever testes automatizados
- Criar dashboard com Power BI ou Grafana usando Application Insights

---
