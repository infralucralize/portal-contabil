# Portal Contábil — Lançamentos Contábeis

App de lançamentos contábeis (plano de contas, empresas, bancos, importação/conciliação
bancária, classificação automática, exportação Domínio) com login Microsoft e dados
persistidos numa planilha Excel no SharePoint (via Microsoft Graph API).

## Arquitetura

- **Hospedagem:** GitHub Pages, arquivo único `index.html`.
- **Login:** MSAL.js (popup, delegado), reaproveitando o app registration já usado pelo
  `central-triagem` (clientId `0876f783-54bf-4149-8ce3-4d72d4b5af94`).
- **Dados:** planilha `PortalContabil.xlsx` na biblioteca "Operacional" do site raiz do
  SharePoint (`lucralize.sharepoint.com`), pasta
  `PI - PROCESSOS E INFRA/AUTOMAÇÕES CLAUDE/portal-contabil/`. Cada aba é uma Tabela do
  Excel (Empresas, PlanoContas, Bancos, Lancamentos, Regras), manipulada via
  `/workbook/tables/{nome}/rows` do Graph.
- **Extratos importados:** arquivo original (OFX/Excel/PDF) é enviado para a subpasta
  `Extratos Importados` na mesma pasta, para histórico/auditoria.

## Configuração necessária no Azure AD

Adicionar a URL do GitHub Pages deste repositório como Redirect URI (plataforma SPA) no
app registration `0876f783-54bf-4149-8ce3-4d72d4b5af94`. Nenhuma permissão nova é
necessária — o app já tem `Sites.ReadWrite.All` e `User.Read` delegados.

## Limitações conhecidas (v1)

- Excluir uma empresa cascade-deleta bancos/lançamentos/regras dela no Excel um a um —
  pode demorar se houver muito histórico.
- `DB.movimentos` (log bruto do que foi importado) só existe na memória da sessão, não é
  persistido — é reconstruível a partir do campo `Origem` dos lançamentos.
- Sem tratamento de concorrência: duas pessoas editando ao mesmo tempo podem sobrescrever
  uma a outra (mesma limitação que os outros portais baseados em SharePoint/Excel).
