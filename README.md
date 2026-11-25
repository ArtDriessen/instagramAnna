# Automação do Instagram da Anna (n8n + Claude)

Médica-veterinária que atende em domicílio cães e gatos, com ênfase em medicina felina.
Três automações no n8n, usando a API oficial do Instagram e a API do Claude. **Não usa planilha**: a aprovação dos posts é feita em tabelas internas do próprio n8n.

| # | Arquivo | O que faz |
|---|---------|-----------|
| 1 | `workflow-1-geracao-conteudo.json` | Toda segunda, 9h: cria 3 rascunhos (legenda, roteiro/slides, sugestão visual) e salva na tabela `conteudos` com status `rascunho` |
| 2 | `workflow-2-postagem.json` | A cada 30 min: publica **um** post com status `aprovado` e imagem preenchida, e marca como `publicado` |
| 3 | `workflow-3-comentarios.json` | A cada 10 min: lê comentários novos; responde sozinho só elogios e dúvidas simples; o resto vai para a tabela `fila_humana` |

**Comece pelo [GUIA-PASSO-A-PASSO.md](GUIA-PASSO-A-PASSO.md).**

Outros arquivos: `docker-compose.yml` (roda o n8n), `config.exemplo.env` (modelo das configurações; copie para `config.env`), `diretrizes-conteudo.md` (o que a IA pode e não pode dizer).

## Limitações atuais
- Publica **imagem única**. Reels e carrossel podem ser acrescentados depois.
- O token do Instagram vale 60 dias e precisa ser renovado (o guia explica).
- Os workflows foram escritos sem poder testar a importação em uma instância real do n8n; o guia diz o que conferir ao importar.
- Nunca suba o `config.env` para o GitHub.
