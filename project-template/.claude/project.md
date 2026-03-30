# [Nome do Projeto]

## Stack

- **Backend:** Laravel X, PHP 8.X
- **Frontend:** React 18, TypeScript, Vite
- **Admin:** Filament v3
- **Infra:** AKS (Azure), namespaces `[projeto]-hml` / `[projeto]-prod`
- **Banco:** Azure MySQL 8.0 (ou PostgreSQL X)
- **Fila:** Redis + Laravel Horizon
- **Storage:** Azure Blob Storage
- **CI/CD:** GitLab CI

## Repositórios

- Backend: `git@gitlab.com:[org]/[projeto]-api.git`
- Frontend: `git@gitlab.com:[org]/[projeto]-web.git`

## Ambientes

- Local: `http://localhost:8000`
- Homologação: `https://hml.[projeto].com.br`
- Produção: `https://[projeto].com.br`

## Módulos principais

<!--
Liste os módulos de negócio do sistema. Isso ajuda as skills a identificar
arquivos relevantes e riscos quando o usuário menciona um módulo pelo nome.
-->

- **[Módulo A]:** descrição curta do que faz
- **[Módulo B]:** descrição curta do que faz

## Integrações externas

<!--
Liste APIs e serviços externos. Importante para auditoria de fallback e
para o planner identificar dependências.
-->

- `[Nome da API]`: finalidade, endpoint base

## Contexto de negócio

<!--
1-3 parágrafos explicando o que o sistema faz e para quem.
Quem são os usuários? Qual o problema que resolve?
-->
