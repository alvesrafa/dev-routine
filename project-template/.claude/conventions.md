# Convenções do Projeto

<!--
Este arquivo é lido pelas skills /code-review e /planejar-tarefa.
Documente aqui os padrões que o time decidiu seguir.
Seja específico — convenções vagas não ajudam na revisão.
-->

## PHP / Laravel

- Controllers apenas delegam para Services, sem lógica de negócio direta
- Services contêm lógica de negócio, sem acesso direto ao banco (usar Repositories quando query for complexa)
- DTOs para transferência entre camadas e integração com APIs externas
- Jobs sempre implementam `ShouldBeUnique` quando idempotência for necessária
- Migrations devem ter `down()` funcional
- Nunca usar `DB::statement` com DDL sem verificar ambiente
- Form Requests obrigatórios para validação de endpoints públicos
- Soft deletes em entidades principais (nunca hard delete de dados de negócio)

## TypeScript / React

- Componentes funcionais com hooks, sem class components
- Props tipadas com `interface`, nunca `any` ou `object`
- Axios centralizado em `services/api.ts`, sem instâncias avulsas
- Estado global via [Zustand / Context / Redux — preencher]
- Nenhum `console.log` em código commitado

## Git

- Branches: `feature/`, `fix/`, `hotfix/`, `chore/`
- Commits em português, modo imperativo: "Adiciona validação de CPF"
- PR obrigatório para `main` e `develop`, push direto proibido
- PR deve ter descrição mínima: o que muda e como testar

## Nomenclatura

- Classes: PascalCase
- Métodos e variáveis: camelCase (TS) / snake_case (PHP)
- Tabelas e colunas do banco: snake_case
- Rotas de API: kebab-case, plural para recursos (`/api/beneficiarios`)

## O que não fazer

<!--
Liste antipadrões específicos que já causaram problema neste projeto.
Exemplos concretos ajudam mais que regras abstratas.
-->

- Não usar `$request->all()` diretamente — usar `$request->validated()` após Form Request
- Não retornar dados sensíveis em respostas de erro (stack trace em produção)
- Não commitar arquivos `.env`
