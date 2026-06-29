# CorretorPro — Repositório público

> Plataforma de gestão estratégica de carteira para corretores de plano de saúde e seguro.
> Controle clientes, contratos, comissões e renovações em um único lugar — com previsibilidade real de receita.

---

> [Acesse a versão beta da plataforma clicando aqui](corretorpro.den42.tech)

>*Contas criadas nesta versão estão sujeitas a serem deletadas a qualquer momento.*

---

## Stack

| Camada    | Tecnologia                                  |
|-----------|---------------------------------------------|
| Backend   | ASP.NET Core 8 Web API                      |
| Frontend  | Blazor WebAssembly (.NET 8)                 |
| Banco     | PostgreSQL 16                               |
| ORM       | Entity Framework Core 8                     |
| Auth      | ASP.NET Core Identity + JWT + Refresh Token |
| Logs      | Serilog (Console + File)                    |
| Container | Docker + Docker Compose                     |

---

## Arquitetura

```
CorretorPro.sln
├── src/
│   ├── CorretorPro.Domain          # Entidades, enums, interfaces, regras de negócio
│   ├── CorretorPro.Application     # Use cases, DTOs, services, Result<T>, planos
│   ├── CorretorPro.Infrastructure  # EF Core, Identity, Repos, MultiTenancy, Plans
│   ├── CorretorPro.API             # Controllers v1, JWT middleware, Export, Admin
│   └── CorretorPro.Web             # Blazor WASM, NavMenu, páginas, serviços
└── tests/
    ├── CorretorPro.Domain.Tests
    └── CorretorPro.Application.Tests
```

---

## Funcionalidades

### Fase 1 — MVP
- Multi-tenancy com isolamento por slug ou header `X-Tenant-Id`
- Gestão de Clientes (CRUD, ativação/desativação, auditoria)
- Gestão de Contratos (criação, renovação, cancelamento)
- Gestão de Produtos e Regras de Comissão
- Dashboard com KPIs: receita mensal, comissão estimada, alertas de vencimento

### Fase 2 — Autenticação avançada
- Refresh Tokens com rotação automática (30 dias)
- Logout com revogação server-side
- Gestão de Colaboradores com roles (Admin / Corretor / Assistente)
- Auditoria: `CreatedBy` / `UpdatedBy` em todas as entidades
- API versionada em `/api/v1/`

### Fase 3 — Operacional
- Alertas de renovação no Dashboard
- Simulador de comissão interativo
- Exportação CSV de clientes e contratos (com autenticação)

### Fase 4 — Planos e Admin
- Estrutura de planos configurável via `.env` (limites de clientes e colaboradores)
- Bloqueio efetivo ao atingir limite (HTTP 402)
- Painel Admin: informações do tenant, plano ativo e barras de uso

### Fase 5 — Refinamento de Interface
- Design System v5.0 com dark mode automático (`prefers-color-scheme`)
- Sidebar redesenhada: avatar + logout na base, colapsável em tablet, hamburguer em mobile
- Toast notifications centralizadas
- Skeleton screens nas listas
- Paginação em Clientes e Contratos
- Layout totalmente responsivo (mobile / tablet / desktop)

---

## Deploy rápido

### Pré-requisitos
- Docker + Docker Compose
- .NET SDK 8.x (apenas para o `make setup`)

### 1. Configurar variáveis de ambiente

```bash
cp .env.example .env
nano .env  # Edite senhas, JWT_SECRET_KEY e planos
```

Variáveis obrigatórias:
```env
POSTGRES_PASSWORD=senha_forte_aqui
JWT_SECRET_KEY=chave_com_minimo_32_caracteres!!
TENANT_DEFAULT_PLAN=basico
```

### 2. Subir os containers

```bash
make setup   # Build + testes + docker up (primeira vez)
make up      # Rebuild e start (deploys subsequentes)
make down    # Parar containers
make logs    # Ver logs em tempo real
```

### 3. Registrar o primeiro tenant

```bash
curl -X POST http://localhost:8080/api/v1/tenants/register \
  -H 'Content-Type: application/json' \
  -d '{
    "nomeEmpresa": "Minha Corretora",
    "slug": "minha-corretora",
    "adminEmail": "admin@corretora.com",
    "adminNome": "Admin",
    "adminSenha": "SenhaForte123!"
  }'
```

### 4. Acessar

| URL | Descrição |
|-----|-----------|
| `http://localhost:5050` | Frontend (Blazor) |
| `http://localhost:8080/swagger` | API docs |
| `http://localhost:8080/health` | Health check |

---

## Configuração de Planos

Os planos são configurados no `.env`. Valor `0` = ilimitado.

```env
TENANT_DEFAULT_PLAN=basico

Planos__basico__Nome=Básico
Planos__basico__MaxClientes=50
Planos__basico__MaxColaboradores=2

Planos__pro__Nome=Pro
Planos__pro__MaxClientes=200
Planos__pro__MaxColaboradores=5

Planos__enterprise__Nome=Enterprise
Planos__enterprise__MaxClientes=0
Planos__enterprise__MaxColaboradores=0
```

---

## Roadmap

| Fase | Status | Descrição |
|------|--------|-----------|
| 0 | ✅ | Fundação: multi-tenant, Identity, JWT, Docker |
| 1 | ✅ | MVP: Clientes, Contratos, Produtos, Dashboard |
| 2 | ✅ | Refresh Tokens, Colaboradores, Auditoria, API v1 |
| 3 | ✅ | Alertas, Simulador, Exportação CSV |
| 4 | ✅ | Planos configuráveis, Painel Admin |
| 5 | ✅ | Refinamento de interface, Dark Mode, Responsividade |
| 5b | 🔜 | Comissões flexíveis (múltiplos modelos de cálculo) |
| 6 | — | Integração de pagamento |

---

## Estrutura de Autenticação

```
POST /api/v1/auth/login    → { token, refreshToken, user }
POST /api/v1/auth/refresh  → { token, refreshToken }  (rotação automática)
POST /api/v1/auth/logout   → 204 (revoga refresh token)
GET  /api/v1/auth/me       → dados do usuário autenticado
```

Todas as requisições autenticadas precisam do header:
```
Authorization: Bearer {token}
X-Tenant-Id: {slug-da-corretora}
```

---

## Licença

Proprietário — todos os direitos reservados.
