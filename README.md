# Dashboard Renato — Monitoramento OmniChat

Interface web de **uso interno** da **Muvecom** para monitorar conversas, agentes e indicadores operacionais do **OmniChat** (Renato AI / autoatendimento). O painel é uma aplicação **estática** em um único arquivo (`index.html`).

**Produção:** [https://dashrenato.muvecom.com.br/](https://dashrenato.muvecom.com.br/)

## Sobre o projeto

O dashboard consome dados via API proxy da Muvecom (Parse/OmniChat) e apresenta:

- KPIs de qualificação de leads (quente / morno / frio / não transferidas)
- Aderência ao script da IA e demora de resposta (IA vs vendedor)
- Métricas de transferências da IA (sucesso, falha e desdobramento por unidade) — cards e linhas das tabelas são clicáveis para abrir a lista de conversas correspondentes
- Gráfico de negativas e resumo de palavras inadequadas
- Tabelas de agentes e conversas recentes, com filtros por período, operador, status e plataforma
- **Guia rápida por operador** — total de atendimentos, direcionamento IA e falhas de transferência. Cada linha funciona como botão de filtro: clicar seleciona o operador no filtro global e atualiza todo o dashboard. A linha "Todos os operadores" no topo agrega os totais e limpa o filtro.
- **Legenda** dos critérios de temperatura do lead
- Visualização de JSON bruto retornado pela API

## Estrutura

```
renato-main/
├── index.html    # Aplicação completa (HTML, CSS e JavaScript)
└── README.md
```

Scripts de deploy (PM2, nginx, SSL) ficam em `../deploy/`. Ver [deploy/README.md](../deploy/README.md).

## Tecnologias

- **HTML5, CSS3 e JavaScript** (vanilla, sem build)
- **API REST Muvecom** — proxy OmniChat

## Configuração da API

No início do script em `index.html`, o objeto `CONFIG` define os endpoints:

| Variável | Padrão |
|----------|--------|
| `OMNICHAT_API_URL` | `https://api.muvecom.com.br/v1/omnichat-chats?limit=2000` |
| `AGENTS_API_URL` | `https://api.muvecom.com.br/v1/omnichat-agents` |

**Token (opcional):** na URL do dashboard, `?token=SEU_TOKEN` — repassado às chamadas da API.

**CORS:** se “Carregar Dados” falhar no navegador, inclua a origem do dashboard (ex.: `https://dashrenato.muvecom.com.br`) nas origens permitidas da API.

## Como executar localmente

1. Abra `index.html` no navegador, **ou** sirva a pasta com um servidor estático:

   ```bash
   npx --yes serve renato-main -l 8030
   ```

2. Clique em **Carregar Dados OmniChat**.
3. Use os filtros de **período** e **operador** conforme necessário.

## Deep link OmniChat

Links **Ver conversa** usam o `objectId` da conversa (não o ID do cliente):

`https://app.omni.chat/#/home/chat/{chat.objectId}`

## Times regionais

O mapeamento operador → time regional (`E201 Ourinhos`, `E302 Londrina`, etc.) está em `REGIONAL_TEAMS` no `index.html`. Ajuste essa constante quando houver atualização da planilha de vendedores/cidades.

## Filtros automáticos de operadores

Para manter a "Guia rápida por operador", o select global de operador e o cálculo de agentes focados nos vendedores reais, o `index.html` aplica duas regras automáticas:

- **Operadores bloqueados por nome** — operadores cujo nome contenha qualquer um dos termos abaixo são ocultados de toda a interface (botões, tabelas e select). A lista vive na constante `BLOCKED_OPERATOR_NAME_TERMS` (próxima à função `isBotUser` em `index.html`):
  - `Suporte`
  - `Muvecom`
  - `Renan Volpe`
  - `Leandro Gregoleto`

  Comparação é case-insensitive e usa `String.includes`, então variações como "Suporte Geral" ou "MUVECOM" também são filtradas. Para incluir/remover termos, edite essa constante.

- **Operadores sem time** — a "Guia rápida por operador" oculta linhas em que tanto `team` quanto `regionalTeam` estão indefinidos (`'--'`). O filtro é aplicado em `renderProductivityTable`.

## Deploy em produção

Resumo (detalhes em `../deploy/README.md`):

```bash
cd /caminho/para/Renato
bash deploy/setup-pm2-dashrenato.sh
sudo bash deploy/finish-with-sudo.sh   # nginx + Certbot
```

**Atualizar o front:**

1. Substituir `renato-main/index.html`.
2. `pm2 restart dashrenato-ui`

| Ação | Comando |
|------|---------|
| Logs PM2 | `pm2 logs dashrenato-ui` |
| Teste local | `curl -sI http://127.0.0.1:8030/` |

## Aviso legal e confidencialidade

Todo o código-fonte, arquitetura, design e lógica desta interface, assim como os dados trafegados por ela, são **propriedade intelectual exclusiva** da **Muvecom** e destinados a **uso restrito e autorizado**.

- É expressamente **proibida** a cópia, reprodução, engenharia reversa, distribuição ou comercialização deste material, total ou parcial, por terceiros não autorizados.
- As informações manipuladas por este sistema podem conter dados estruturais e comerciais sensíveis.

---

*Desenvolvido e mantido pela equipe da [Muvecom](https://muvecom.com.br).*
