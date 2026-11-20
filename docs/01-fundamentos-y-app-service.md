# Lección 1: Fundamentos de Azure y despliegue en App Service

> **Objetivo:** entender los conceptos base de Azure y publicar una API .NET 10 en la nube con costo **$0**.
> **Resultado:** API en vivo en `https://app-cloudshop-francisco.azurewebsites.net/weatherforecast`

---

## 1. Conceptos aprendidos

### ¿Qué es la nube?
Usar servidores de otro (Microsoft, Amazon, Google) por internet y **pagar solo lo que se usa**.
Ventajas: pago por uso, escalabilidad, alta disponibilidad, alcance global y servicios administrados.

### Modelos de servicio
| Modelo | Tú manejas | Azure maneja | Ejemplo |
|---|---|---|---|
| **IaaS** | SO, runtime, app, datos | Hardware, red | Virtual Machines |
| **PaaS** | Solo código y datos | Todo lo demás | App Service, Azure SQL, Functions |
| **SaaS** | Nada, solo lo usas | Todo | Outlook, Teams |
| **Serverless** | Solo código | Todo; pagas por ejecución | Azure Functions |

> Truco: **¿hay código tuyo?** Entonces no es SaaS. Código + servidor de Azure = **PaaS**.

### Jerarquía de Azure
```
Tenant (Microsoft Entra ID)   → organización y usuarios
 └── Subscription             → facturación y límites
      └── Resource Group      → carpeta de un proyecto/ambiente
           └── Resources      → App Service, SQL, Key Vault...
```
- Los permisos (**RBAC**: Owner, Contributor, Reader) se heredan hacia abajo.
- **Un Resource Group por ambiente** (`dev`, `qa`, `prod`): separa permisos, costos y ciclo de vida.
- Un usuario puede pertenecer a **varios tenants**, cada uno con sus propias reglas (ej. MFA).

### Geografía
`Geografía → Región (ej. West US 2) → Zonas de disponibilidad → Datacenters`
Los precios, los servicios disponibles y **las cuotas** cambian por región.

### Seguridad
- **MFA** es obligatorio para entrar al portal y a la CLI de Azure.
- **Responsabilidad compartida:** Azure protege la infraestructura; yo protejo mi código, datos, accesos y secretos.
- **Key Vault** guarda secretos, llaves y certificados.
- **Managed Identity** le da a la app una identidad propia para acceder a otros servicios **sin contraseñas**.

### App Service Plan vs Web App
- **App Service Plan** = el servidor (región, SO, tamaño/SKU, instancias). **Es lo que se paga.**
- **Web App** = la aplicación que corre dentro del plan. Varias apps pueden compartir un plan.

| SKU | Uso | Notas |
|---|---|---|
| F1 (Free) | Aprender, demos | $0, 60 min CPU/día, se duerme |
| B1 (Basic) | Dev/test | Dominio propio, Always On |
| S1 (Standard) | Producción | + Deployment slots, autoescalado |
| P1v3 (Premium) | Producción seria | Más potencia |

- **Scale up** = servidor más grande. **Scale out** = más instancias (requiere app *stateless*).

### Cuotas vs límites
- **Cuota:** tope por suscripción, región y recurso; **se puede pedir aumento** (portal → *Cuotas*).
- **Límite duro:** no se puede cambiar.

---

## 2. Lo que se construyó

| Recurso | Nombre | Detalle |
|---|---|---|
| Resource Group | `rg-cloudshop-dev` | East US |
| App Service Plan | `asp-cloudshop-dev` | F1, Linux, **West US 2** |
| Web App | `app-cloudshop-francisco` | .NET 10 (LTS) |
| Repositorio | `FranciscoSoftware/cloudshop-azure` | Público en GitHub |

Convención de nombres (Cloud Adoption Framework): `rg-`, `asp-`, `app-`, `sql-`, `kv-`, `func-` + proyecto + ambiente.

---

## 3. Comandos usados

```powershell
# Herramientas
winget install -e --id Microsoft.AzureCLI
winget install -e --id GitHub.cli

# Login (con MFA, eligiendo la cuenta en una ventana de incógnito)
az login --use-device-code
az account list --all --refresh --output table
az account show --output table

# Resource Group
az group create --name rg-cloudshop-dev --location eastus

# Proyecto .NET
dotnet new gitignore
dotnet new sln -n CloudShop
dotnet new webapi -n CloudShop.Api -o src/CloudShop.Api -f net10.0
dotnet sln add src/CloudShop.Api

# GitHub
git init -b main
git add .
git commit -m "Lección 1: API base CloudShop en .NET 10"
gh repo create cloudshop-azure --public --source . --push

# Despliegue (az webapp up está deprecado)
az appservice plan create --name asp-cloudshop-dev --resource-group rg-cloudshop-dev --sku F1 --is-linux --location westus2
az webapp create --name app-cloudshop-francisco --resource-group rg-cloudshop-dev --plan asp-cloudshop-dev --runtime "DOTNETCORE:10.0"
dotnet publish src/CloudShop.Api -c Release -o $env:TEMP\cloudshop-publish
Compress-Archive -Path "$env:TEMP\cloudshop-publish\*" -DestinationPath "$env:TEMP\cloudshop.zip" -Force
az webapp deploy --name app-cloudshop-francisco --resource-group rg-cloudshop-dev --src-path "$env:TEMP\cloudshop.zip" --type zip

# Borrar todo (cuando ya no se necesite)
az group delete --name rg-cloudshop-dev --yes --no-wait
```

---

## 4. Problemas encontrados y cómo se resolvieron

| Problema | Causa | Solución |
|---|---|---|
| `AADSTS50076 ... must use multi-factor authentication` | El tenant exige MFA | Activar MFA (Microsoft Authenticator) |
| `AADSTS50020 ... does not exist in tenant` | El navegador usó otra cuenta Microsoft | `az login --use-device-code` + ventana de incógnito |
| `N/A (tenant level account)` / *No subscriptions found* | La cuenta aún no tenía suscripción | Crear la suscripción Pay-As-You-Go en el portal |
| No se pudo crear una nueva cuenta gratuita | La prueba gratis es una sola por persona/tarjeta/teléfono | Usar Pay-As-You-Go con SKUs gratuitos |
| `'az webapp up' is deprecated` | Comando retirado en la CLI 2.93 | `appservice plan create` + `webapp create` + `webapp deploy` |
| `Current Limit (F1 App Service instances): 0` | Suscripción nueva con cuota 0 en East US | Crear el plan en otra región (West US 2) |
| `Token 'in' inesperado` en PowerShell | Dos comandos pegados en una línea | Pegar un comando a la vez (Esc limpia la línea) |

---

## 5. Control de costos
- Presupuesto mensual de **$5 USD** con alertas al 50% y 100% (avisa, **no bloquea**).
- Todo en este momento cuesta **$0** (plan F1).
- Evitar: VMs, AKS, API Management Developer/Standard, Application Gateway, Front Door, NAT/VPN Gateway.
- Al terminar prácticas que cuesten: `az group delete`.

---

## 6. Preguntas de entrevista repasadas
1. ¿Diferencia entre App Service Plan y Web App?
2. ¿Por qué un Resource Group por ambiente?
3. ¿Qué problema resuelve Managed Identity?
4. ¿IaaS vs PaaS vs SaaS?
5. ¿Scale up vs scale out?
6. ¿Cuota vs límite duro?

---

## Siguiente: Lección 2
**Clean Architecture** + **Azure SQL** (oferta gratuita) + **Entity Framework Core** + **Key Vault** + **Managed Identity**.
