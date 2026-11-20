# CLAUDE.md

## Qué es este proyecto
**CloudShop** es un proyecto de aprendizaje y portafolio. Un desarrollador .NET mid-senior de Colombia está aprendiendo **Azure** paso a paso con Claude como tutor.

**Meta principal: ser empleable como desarrollador .NET senior** (Colombia y remoto). La certificación es secundaria: la **AZ-204 se retiró el 31/07/2026**; su reemplazo es la **AI-200**, opcional y para después de tener el portafolio sólido.

El proyecto es una API de e-commerce que **crece lección por lección**. Cada lección agrega una habilidad que evalúan en entrevistas senior y queda documentada en `docs/`.

## Enfoque "senior"
- Lo que diferencia a un senior es **justificar decisiones y sus trade-offs**, no conocer más servicios. Siempre explicar *por qué* se elige algo y qué se sacrifica.
- Cada lección deja en el repo:
  - `docs/NN-tema.md`: conceptos, recursos, comandos, problemas y soluciones, y **preguntas de entrevista** con respuestas modelo.
  - Un **ADR** en `docs/adr/NNNN-titulo.md` (contexto, decisión, alternativas, consecuencias) por cada decisión de arquitectura relevante.
- Código con calidad de producción: Clean Architecture, pruebas, manejo de errores, logs estructurados, sin secretos.
- Incluir cuando aplique: términos y frases en **inglés** para entrevistas remotas, y ejercicios de **diseño de sistemas**.

## Cómo trabajar en este repo
- **Idioma:** responder y documentar en **español**.
- **Modo tutor:** explicar el *por qué* de cada paso, con analogías simples y una nota "concepto de entrevista" cuando aplique. El usuario ejecuta los comandos para aprender; no hacerlo todo por él, salvo verificaciones de solo lectura.
- **Comandos:** un comando por bloque de código (`powershell`), sin mezclar varios en la misma línea. El usuario trabaja en Windows con PowerShell.
- **Ritmo:** avanzar un paso a la vez y pedir confirmación ("¿qué te salió?") antes de seguir.
- **Costo ~$0:** usar solo SKUs gratuitos o de centavos (App Service F1, Functions Consumption, oferta gratuita de Azure SQL, Container Apps con cupo gratis, Static Web Apps Free, Service Bus Basic). Advertir antes de cualquier recurso que cobre. Evitar VMs, AKS en Azure, API Management Developer/Standard, Application Gateway, Front Door y NAT/VPN Gateway.
- **Seguridad:** nunca escribir contraseñas, cadenas de conexión, keys, tenant IDs ni IDs de suscripción en archivos del repo (es **público**). Usar Key Vault, Managed Identity y OIDC para GitHub Actions.
- **Git:** no hacer commit ni push sin que el usuario lo pida. Un commit por lección, con mensaje `Lección N: ...`.

## Stack
- .NET 10 (LTS), ASP.NET Core Web API (minimal APIs), solución `CloudShop.slnx`
- Azure CLI 2.93+ (`az webapp up` está **deprecado**: desplegar con `appservice plan create` + `webapp create` + `dotnet publish` + zip + `az webapp deploy`)
- GitHub: repositorio público `FranciscoSoftware/cloudshop-azure`, GitHub CLI (`gh`)

## Estructura
```
cloudshop-azure/
├── CLAUDE.md
├── CloudShop.slnx
├── src/
│   └── CloudShop.Api/        ← API .NET 10 (se dividirá en capas en la Lección 2)
├── tests/                    ← (Lección 3) xUnit + Testcontainers
├── docs/                     ← un resumen por lección
│   └── adr/                  ← decisiones de arquitectura
├── infra/                    ← (Lección 9) Bicep
└── .github/workflows/        ← (Lección 4) CI/CD
```

## Recursos en Azure (ambiente dev)
| Recurso | Nombre | Notas |
|---|---|---|
| Resource Group | `rg-cloudshop-dev` | East US |
| App Service Plan | `asp-cloudshop-dev` | F1, Linux, West US 2 (East US tenía cuota F1 = 0) |
| Web App | `app-cloudshop-francisco` | `https://app-cloudshop-francisco.azurewebsites.net` |

Convención de nombres: `rg-`, `asp-`, `app-`, `sql-`, `kv-`, `func-`, `sb-` + `cloudshop` + ambiente.

## Comandos frecuentes
```powershell
# Ejecutar en local
dotnet run --project src/CloudShop.Api

# Login a Azure (la cuenta exige MFA)
az login --use-device-code

# Desplegar
dotnet publish src/CloudShop.Api -c Release -o $env:TEMP\cloudshop-publish
Compress-Archive -Path "$env:TEMP\cloudshop-publish\*" -DestinationPath "$env:TEMP\cloudshop.zip" -Force
az webapp deploy --name app-cloudshop-francisco --resource-group rg-cloudshop-dev --src-path "$env:TEMP\cloudshop.zip" --type zip
```

## Ruta de aprendizaje
| # | Lección | Demuestra | Estado |
|---|---|---|---|
| 1 | Fundamentos + API en **App Service** | Cloud básico | ✅ `docs/01-fundamentos-y-app-service.md` |
| 2 | **Clean Architecture** + **Azure SQL** + EF Core + **Key Vault** + **Managed Identity** | Arquitectura, datos, seguridad | ⬜ Siguiente |
| 3 | **Pruebas**: xUnit + integración con Testcontainers | Calidad | ⬜ |
| 4 | **CI/CD** con GitHub Actions + OIDC (sin secretos) | DevOps | ⬜ |
| 5 | **Service Bus** + **Azure Functions**: pedidos asíncronos, idempotencia, dead-letter | Sistemas distribuidos | ⬜ |
| 6 | **Observabilidad**: OpenTelemetry + Application Insights + KQL | Producción | ⬜ |
| 7 | **Seguridad de API**: JWT con Entra ID, roles, rate limiting | Seguridad | ⬜ |
| 8 | **Resiliencia y rendimiento**: Polly, caché con Redis, benchmarks | Nivel senior | ⬜ |
| 9 | **Contenedores**: Docker + Container Apps + infraestructura en **Bicep** | Cloud moderno | ⬜ |
| 10 | **IA**: Azure OpenAI + búsqueda vectorial (RAG) | Diferenciador | ⬜ |

Al terminar cada lección: crear `docs/NN-tema.md`, los ADR que correspondan, y actualizar esta tabla.

Servicios que cuestan (PostgreSQL, Redis, AKS): practicarlos en local con Docker y en Azure solo temporalmente, borrando el Resource Group al terminar.

## Notas del entorno
- El repo está dentro de OneDrive: si aparecen errores de "archivo en uso" en `bin/` u `obj/`, pausar la sincronización.
- Hay un presupuesto de $5 USD/mes con alertas en la suscripción.
