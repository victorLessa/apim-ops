# Publicação APIOps para Azure APIM

## Premissas

- `apimartifacts/` é a fonte da verdade; não há extração do APIM nesta esteira.
- A configuração é publicada primeiro em DES e, em push para `main`, depois em PRD. O Environment `prd` deve exigir aprovação.
- O fluxo usa OIDC com `azure/login` e a release oficial fixa APIOps Publisher v7.0.4.
- O template contém um exemplo de backend `pedidos`; ajuste o nome para corresponder ao artefato existente.

## Pré-requisitos

1. Criar os GitHub Environments `des` e `prd`. Configurar required reviewers em `prd`.
2. Em cada Environment, cadastrar como **Variables**: `AZURE_SUBSCRIPTION_ID`, `AZURE_RESOURCE_GROUP_NAME`, `API_MANAGEMENT_SERVICE_NAME`, `AZURE_CLIENT_ID`, `AZURE_TENANT_ID` e `BACKEND_PEDIDOS_URL`.
3. Configurar a federação OIDC do GitHub no Microsoft Entra ID e conceder à identidade permissões de publicação nos APIMs DES e PRD. O fluxo usa Azure CLI authentication do `azure/login` por meio do `DefaultAzureCredential` do Publisher.
4. Ajustar `configuration/configuration.template.yaml` para os nomes e overrides reais, mantendo os placeholders `${VAR}` definidos no workflow.

## Uso

- Merge/push em `main` com mudanças em `apimartifacts/**` ou `configuration/**` publica em DES e, após sucesso e aprovação do Environment, em PRD.
- Em **Actions > Publicar APIM > Run workflow**, escolher `des` ou `prd` e o modo. `full` (padrão) publica tudo; `incremental` publica o commit selecionado. Se apenas variáveis de ambiente mudaram, executar manualmente com `full`.
- Valores sensíveis de Named Values devem preferencialmente ser referências a Azure Key Vault; não os armazene nas variáveis do GitHub.

## Atenção a exclusões

Com `COMMIT_ID`, o Publisher remove do APIM os artefatos excluídos no commit. Revise cuidadosamente exclusões antes da aprovação de PRD. Para excluir uma API, remova sua pasta completa em `apimartifacts/`: apagar apenas o arquivo de especificação não remove a API, enquanto apagar o arquivo de informações pode removê-la.

## Ajustes pendentes

- TODO: substituir `pedidos`/`BACKEND_PEDIDOS_URL` pelo nome e pela variável do backend realmente existente.
- TODO: cadastrar as variáveis separadamente em `des` e `prd` e validar o fluxo OIDC/RBAC com os IDs e nomes reais dos recursos Azure.
- TODO: revisar a versão fixada do Publisher (v7.0.4) antes de atualizações futuras.
