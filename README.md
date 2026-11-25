# Automação do Instagram - Clínica Veterinária Felina (n8n + Claude)

Três workflows do n8n que usam a API oficial da Meta (Instagram Graph API) e a API do Claude:

| # | Arquivo | O que faz |
|---|---------|-----------|
| 1 | `workflow-1-geracao-conteudo.json` | Toda segunda, 9h: gera 3 rascunhos (legenda + roteiro/slides + sugestão visual) e salva na planilha com status `rascunho` |
| 2 | `workflow-2-postagem.json` | A cada 30 min: publica **uma** postagem com status `aprovado` e marca como `publicado` |
| 3 | `workflow-3-comentarios.json` | A cada 10 min: lê comentários novos, classifica e responde sozinho só elogios e dúvidas simples. O resto vai para a aba `FilaHumana` |

Fluxo de aprovação: você revisa os rascunhos na planilha, cola o link da imagem em `image_url` e muda o status para `aprovado`. Só então o workflow 2 publica.

## 1. Requisitos
- Conta Instagram **profissional** ligada a uma **Página do Facebook**.
- App no [Meta for Developers](https://developers.facebook.com) com as permissões `instagram_basic`, `instagram_content_publish`, `instagram_manage_comments` e `pages_show_list`/`pages_read_engagement`. Em modo desenvolvimento funciona para a sua própria conta. Para uso em contas de terceiros é preciso passar pela revisão do app.
- Token de acesso de longa duração e o `IG_USER_ID` (siga a documentação da Meta; os tokens expiram e precisam ser renovados).
- Chave da API da Anthropic.
- Conta Google (Google Sheets) para a planilha de aprovação.
- Docker instalado (ou n8n Cloud, nesse caso configure as variáveis do `.env` no próprio n8n).

## 2. Instalar
```bash
cp .env.example .env     # preencha os valores
docker compose up -d
```
Abra http://localhost:5678 e crie a conta de administrador. Em servidor (VPS), coloque o n8n atrás de HTTPS (Caddy ou Nginx) e configure `WEBHOOK_URL`.

## 3. Planilha
Crie uma planilha no Google Sheets com duas abas (primeira linha = cabeçalho exato):

**Conteudos**: `id | data_criacao | tema | formato | legenda | roteiro | sugestao_visual | image_url | status | publicado_em`

**FilaHumana**: `comment_id | username | texto | categoria | resposta_sugerida | data | status`

## 4. Importar os workflows
1. No n8n: **Workflows → Import from file** e importe os 3 JSON.
2. Em cada nó do Google Sheets: escolha sua credencial (OAuth do Google) e a planilha, e confira os nomes das abas e colunas.
3. Teste manualmente (**Execute workflow**) antes de ativar.
4. Ative os workflows.

Observações importantes:
- O workflow 3 **não responde a comentários antigos**: na primeira execução ele só registra os comentários existentes. A partir da segunda, responde aos novos.
- O workflow 3 guarda o que já viu na memória do workflow, que só persiste quando ele está **ativo** (não em testes manuais).
- O workflow 2 publica **imagem única** (a imagem precisa estar em um link público JPEG). Reels e carrossel exigem passos extras da Graph API, que podem ser adicionados depois.
- Os JSON foram escritos sem poder testar a importação em uma instância real do n8n. Se algum nó acusar erro de parâmetro depois de importar, ajuste pelo painel do nó. A estrutura e a lógica devem se manter.

## 5. Segurança e boas práticas
- Nunca suba o `.env` para o GitHub (já está no `.gitignore`). Se um token vazar, revogue na hora.
- Mantenha o repositório privado.
- Leia `diretrizes-clinicas.md`: elas definem o que a IA pode e não pode falar. Revise conteúdo e respostas nas primeiras semanas.
- Não use automações com login/senha ou scraping no Instagram. Isso pode bloquear a conta.
