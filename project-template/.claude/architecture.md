# Arquitetura do Projeto

<!--
Este arquivo é lido pelas skills /planejar-tarefa e /auditoria.
Documente decisões técnicas e estrutura para que as skills entendam
o "porquê" das coisas, não apenas o "como".
-->

## Padrões arquiteturais adotados

- **Camadas:** Controller → Service → Repository (quando aplicável) → Model
- **DTOs:** pares de entrada/saída para desacoplar campos internos dos externos (ex: integrações)
- **Jobs:** processamento assíncrono para tudo que leva mais de 2s ou pode falhar com retry
- **Events/Listeners:** para side effects (notificações, logs de auditoria)

## Estrutura de diretórios relevante

```
app/
  Http/
    Controllers/     — apenas recebe request, delega, retorna response
    Requests/        — validação de input
  Services/          — lógica de negócio
  Repositories/      — queries complexas
  DTOs/              — transferência de dados entre camadas
  Jobs/              — processamento assíncrono
  Models/            — Eloquent models com relacionamentos
```

## Módulos e fluxos principais

<!--
Para cada módulo importante, descreva o fluxo principal.
Isso permite que o planner identifique o caminho certo sem perguntar.
-->

### [Módulo A]
Fluxo: [descreva o caminho feliz de ponta a ponta]
Arquivos principais: `app/Services/[Modulo]Service.php`, `app/Jobs/[Modulo]Job.php`

### [Módulo B]
Fluxo: ...

## Infraestrutura K8s

- Dois clusters: `hml` e `prod` no AKS
- Ingress: NGINX com wildcard TLS
- Secrets: Azure Key Vault com CSI Driver
- Workers de fila: deployment separado, fila `[nome]` com X réplicas
- Storage: Azure Files PVC para arquivos temporários, Blob para persistência

## Decisões técnicas e motivos

<!--
Documente decisões não óbvias. "Por que usamos X ao invés de Y?"
Evita que o time questione ou reverta decisões por desconhecimento.
-->

| Decisão | Motivo |
|---------|--------|
| Soft delete em todas as entidades principais | Requisito de auditoria do cliente |
| Fila separada para imports | Evitar que imports longos bloqueiem notificações |

## O que NÃO alterar sem alinhamento

<!--
Partes do sistema que são críticas ou frágeis.
Qualquer mudança aqui deve ser discutida antes.
-->

- Estrutura de migrations de tabelas com >1M registros (risco de lock)
- Contrato de resposta das rotas públicas de API (clientes externos consomem)
