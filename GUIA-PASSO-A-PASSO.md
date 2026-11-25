# Guia passo a passo (do zero até o Instagram da Anna rodando)

Leia na ordem. Cada fase termina com uma **✅ Confirmação**: só avance quando ela estiver verdadeira. Se algo for diferente do descrito (os sites mudam de tela), pare e mande uma captura de tela para quem está ajudando.

## O que vamos montar
1. **Todo domingo/segunda** a IA cria 3 rascunhos de posts (legenda, roteiro e sugestão de imagem).
2. **Você (ou a Anna) revisa** os rascunhos numa tabela dentro do n8n, cola o link da imagem e muda o status para `aprovado`.
3. A automação **publica no Instagram** os posts aprovados.
4. A automação **lê os comentários**: responde elogios e dúvidas simples; o resto fica numa lista para a Anna responder.

Nada é publicado sem aprovação humana. Só as respostas a elogios e dúvidas simples são automáticas.

## O que é o n8n e o que são as "tabelas" (em vez de planilha)
O **n8n** é um programa que executa automações. Ele tem uma aba chamada **Data tables** (tabelas de dados): funciona como uma planilha simples dentro do próprio n8n. Não precisa de Google, Excel nem conta nenhuma.

## Antes de começar: o que você precisa ter
- [ ] Um computador (Windows ou Mac) para instalar e testar. Para rodar 24 horas, depois veremos a opção de servidor (Fase 12).
- [ ] O Instagram da Anna como conta **Profissional** (Comercial ou Criador de conteúdo). Confira em Instagram → Configurações → "Tipo de conta e ferramentas".
- [ ] Uma conta pessoal do **Facebook** (a Meta exige para criar um app de desenvolvedor). Pode ser a sua.
- [ ] Um **cartão de crédito** (para colocar crédito na API da Anthropic; o gasto neste volume costuma ser baixo, e você define o limite).
- [ ] O celular com o Instagram da Anna logado (a Anna precisará aprovar um convite, se a conta for dela e não sua).
- [ ] Aproximadamente 2 a 3 horas para a primeira configuração.

---

## Fase 1 - Baixar o projeto do GitHub
1. Abra https://github.com/ArtDriessen/instagramAnna e entre na sua conta do GitHub.
2. Clique no botão verde **Code** → **Download ZIP**.
3. Abra o arquivo baixado e **extraia** (Windows: botão direito → "Extrair tudo"; Mac: dois cliques).
4. Mova a pasta para um lugar fácil, por exemplo `Documentos`, e renomeie para `instagramAnna` (sem espaços).
5. Dentro da pasta devem estar: `docker-compose.yml`, `config.exemplo.env`, os 3 arquivos `workflow-*.json`, `README.md` e este guia.

✅ Confirmação: você vê esses arquivos dentro da pasta `instagramAnna`.

## Fase 2 - Chave da API da Anthropic (a "inteligência" que escreve os textos)
1. Acesse https://console.anthropic.com e crie uma conta (ou entre). Atenção: é diferente da assinatura do Claude no site.
2. Vá em **Billing** (Cobrança) e adicione um cartão e um valor de crédito (por exemplo, US$ 10). Ative um **limite de gasto mensal** baixo se houver essa opção.
3. Vá em **API Keys** → **Create Key** → dê o nome `instagram-anna` → **Create**.
4. **Copie a chave agora** (começa com `sk-ant-`). Ela só aparece uma vez. Cole em um bloco de notas provisório (você vai usar na Fase 4).

✅ Confirmação: você tem uma chave começando com `sk-ant-` guardada.

## Fase 3 - Conectar o Instagram (app na Meta)
Esta é a fase mais longa. A Meta muda as telas com frequência; os nomes abaixo vêm da documentação oficial.

**3.1 Conta de desenvolvedor**
1. Acesse https://developers.facebook.com e entre com o Facebook.
2. Se pedir, clique em **Começar / Get Started** e conclua o cadastro de desenvolvedor (confirmação por celular ou e-mail).

**3.2 Criar o app**
1. No painel (App Dashboard), clique em **Criar app / Create App**.
2. Dê o nome `Instagram Anna` e informe seu e-mail de contato.
3. Em caso de uso (use case), escolha **Other (Outro)** → **Next**.
4. Em tipo de app, escolha **Business** → **Next** → **Create app**.
5. Na lista de produtos, ache **Instagram** e clique em **Set up (Configurar)**. Deve aparecer **API setup with Instagram login**. (Se a tela mostrar "casos de uso", abra **Manage messaging and content on Instagram** e escolha **API setup with Instagram Login**.)

**3.3 Permissões**
1. Na tela da configuração do Instagram, clique em **Add all required permissions** (se existir).
2. Confira que estas três estão presentes: `instagram_business_basic`, `instagram_business_content_publish`, `instagram_business_manage_comments`. Se faltar alguma, adicione.

**3.4 Gerar o token (a "chave" do Instagram)**
- **Caso A: o Instagram é seu** (o mesmo login do Facebook usado no app): na seção **Generate access tokens**, clique em **Add account** → **Continue** → faça login no Instagram na janela que abrir → **Save**. Depois clique em **Generate token** ao lado da conta e **copie o token**.
- **Caso B: o Instagram é da Anna** e é outra pessoa: no painel, vá em **App roles → Roles → Add People**, escolha **Instagram Testers** e digite o @ da Anna. A Anna, no celular, abre o Instagram → **Configurações → Apps e sites → Convites de testador** (o nome pode variar um pouco) e **aceita** o convite. Depois, volte aqui e faça o Caso A (a Anna pode fazer o login na janela, ao vivo, por videochamada).
- O token gerado aqui vale **60 dias**. Guarde no bloco de notas.

**3.5 Descobrir o ID numérico da conta**
1. Monte este endereço trocando SEU_TOKEN pelo token copiado:
   `https://graph.instagram.com/v26.0/me?fields=user_id,username&access_token=SEU_TOKEN`
2. Cole no navegador e dê Enter. Aparece um texto com `user_id` e `username`.
3. Anote o `user_id` (número longo) e confirme que o `username` é o da Anna.

Observações: o app pode ficar em **modo de desenvolvimento**; para uma conta que você possui/gerencia e que foi adicionada ao app, isso basta (acesso "Standard"). Não é preciso enviar o app para revisão da Meta nem configurar webhooks. Se algum passo exigir "política de privacidade", me avise com a captura de tela.

✅ Confirmação: você tem (1) o token, (2) o `user_id`, (3) o `username` da Anna.

## Fase 4 - Preencher o arquivo de configurações
1. Na pasta `instagramAnna`, **copie** o arquivo `config.exemplo.env` e cole na mesma pasta.
2. Renomeie a cópia para **`config.env`**. (Windows: se o Explorer esconder extensões, ative "Exibir → Extensões de nomes de arquivos".)
3. Abra o `config.env` com o Bloco de Notas (Windows) ou TextEdit em modo texto simples (Mac).
4. Preencha **cada linha, sem aspas e sem espaços extras**:
   - `N8N_ENCRYPTION_KEY`: invente uma sequência de 30+ letras e números (pode usar um gerador de senhas). **Guarde uma cópia**: se perder, perde as credenciais salvas no n8n.
   - `ANTHROPIC_API_KEY`: a chave da Fase 2.
   - `IG_ACCESS_TOKEN`: o token da Fase 3.4.
   - `IG_USER_ID`: o número da Fase 3.5.
   - `IG_USERNAME`: o @ da Anna, sem o arroba.
   - `GRAPH_HOST` e `GRAPH_VERSION`: deixe como estão.
   - `INFO_ATENDIMENTO`: uma linha com informações **verdadeiras** para a IA usar nas respostas: WhatsApp, regiões atendidas, horários, se atende cães e gatos. Evite os símbolos # e $.
5. Salve o arquivo.

✅ Confirmação: `config.env` existe, sem textos "COLE-AQUI" restantes e sem aspas.

## Fase 5 - Instalar o Docker e ligar o n8n
O Docker é o programa que roda o n8n de forma isolada no computador.
1. Baixe o **Docker Desktop** em https://www.docker.com/products/docker-desktop e instale.
   - **Windows**: aceite a opção "WSL 2" se aparecer e reinicie o computador quando pedir.
   - **Mac**: escolha a versão certa (chip Apple "M1/M2/M3/M4" ou Intel; veja em  → Sobre este Mac).
2. Abra o Docker Desktop e espere o ícone indicar que está rodando ("Engine running").
3. Abra um terminal **dentro da pasta**:
   - **Windows**: abra a pasta `instagramAnna` no Explorer, clique na barra de endereço (onde aparece o caminho), digite `cmd` e Enter.
   - **Mac**: no Finder, clique com o botão direito na pasta → **Serviços → Novo Terminal na Pasta** (se não aparecer, abra o Terminal e digite `cd ` seguido de arrastar a pasta para a janela, depois Enter).
4. Digite e dê Enter: `docker compose up -d`
5. Aguarde baixar (alguns minutos na primeira vez). No fim aparece "Started".
6. Abra o navegador em **http://localhost:5678**.
7. Na primeira vez o n8n pede para **criar a conta de administrador** (e-mail, nome, senha). Crie e anote a senha.

✅ Confirmação: você vê o painel do n8n.

## Fase 6 - Criar as duas tabelas (no lugar da planilha)
Os nomes e colunas precisam ser **exatamente** estes (tudo minúsculo, tipo **String**/texto).

1. No n8n, no menu da esquerda, abra **Overview** e procure a aba **Data tables**. Clique em **Create data table** (ou botão semelhante).
2. Nome: `conteudos`. Adicione estas colunas, todas do tipo texto (String):
   `codigo`, `data_criacao`, `tema`, `formato`, `legenda`, `roteiro`, `sugestao_visual`, `image_url`, `status`, `publicado_em`
3. Crie a segunda tabela: `fila_humana`, com colunas (todas texto):
   `comment_id`, `username`, `texto`, `categoria`, `resposta_sugerida`, `data`, `status`

Se a sua versão do n8n não tiver "Data tables" na tela, atualize o n8n (`docker compose pull` e depois `docker compose up -d`) ou me avise.

✅ Confirmação: as duas tabelas existem com todas as colunas.

## Fase 7 - Importar os 3 workflows
1. No n8n, clique em **Create workflow** (ou o botão "+") para abrir um workflow vazio.
2. Menu "⋯" no canto superior direito → **Import from file** → escolha `workflow-1-geracao-conteudo.json`.
3. **Confira o nó de tabela** ("Salvar na tabela conteudos"): clique nele e veja se a tabela `conteudos` está selecionada e as colunas estão mapeadas. Se estiver em branco, selecione a tabela na lista. (Esse é o ponto mais provável de ajuste: não pude testar a importação numa instância real.)
4. Salve (Ctrl+S / Cmd+S). Repita para os arquivos 2 e 3:
   - **Workflow 2**: nós "Buscar post aprovado" e "Marcar como publicado" usam `conteudos`.
   - **Workflow 3**: o nó "Enviar para fila_humana" usa `fila_humana`.
5. Não ative nada ainda.

✅ Confirmação: 3 workflows salvos, com as tabelas selecionadas nos nós correspondentes.

## Fase 8 - Testar a geração de conteúdo
1. Abra o workflow 1 e clique em **Execute workflow**.
2. Espere os nós ficarem verdes. Se algum ficar vermelho, clique nele e leia o erro (veja "Problemas comuns").
3. Vá em **Data tables → conteudos**: devem existir 3 linhas com status `rascunho`.
4. Leia as legendas. **Revise com a Anna**: tom, correção técnica, adequação às normas do CRMV. Se quiser, edite a `legenda` direto na tabela.

✅ Confirmação: 3 rascunhos bons na tabela.

## Fase 9 - Imagens (o Instagram exige um link público)
O Instagram só publica uma imagem se ela estiver em um **link público** na internet, em formato **JPEG** (.jpg).
Uma opção sem criar contas novas:
1. No GitHub, crie um repositório **público** chamado `instagramAnna-midia`.
2. Crie a arte no Canva (ou outro) e exporte como **JPG**, com nome sem espaços nem acentos (ex.: `gato-hidratacao.jpg`). Proporção entre 4:5 (retrato) e 1,91:1; até 8 MB.
3. No repositório: **Add file → Upload files** → envie → **Commit changes**.
4. O link será: `https://raw.githubusercontent.com/ArtDriessen/instagramAnna-midia/main/gato-hidratacao.jpg` (confira abrindo no navegador: a imagem deve aparecer).
5. Atenção: o repositório é público. Só suba artes que serão publicadas mesmo, nunca fotos de pacientes sem autorização.
Se o Instagram recusar esse link, use um serviço de hospedagem de imagens (por exemplo, Cloudinary, plano gratuito) e me avise.

## Fase 10 - Testar a postagem (isto publica de verdade!)
1. Escolha **um** rascunho bom e, em **Data tables → conteudos**, preencha `image_url` com o link da Fase 9 e mude `status` para `aprovado`.
2. Abra o workflow 2 e clique em **Execute workflow**.
3. Se tudo der certo, o post aparece no Instagram da Anna em instantes e o status vira `publicado`.
4. Se for só teste, apague o post depois pelo próprio Instagram.

✅ Confirmação: o post apareceu no perfil e a tabela mostra `publicado`.

## Fase 11 - Ativar as automações e testar os comentários
1. Ative o workflow **2** (chave "Active" no canto superior direito) e o workflow **1**.
2. Ative o workflow **3**. Na **primeira execução** ele só "memoriza" os comentários que já existem (para não responder comentários antigos). Espere cerca de 10 minutos.
3. Com **outro perfil**, comente num post: `Vocês atendem gatos?` e outro: `Meu gato está vomitando, o que dou?`.
4. Em até ~10 minutos: a primeira deve receber resposta automática; a segunda **não** (vai para **Data tables → fila_humana**, e a Anna responde manualmente no Instagram).
5. Veja em **Executions** o histórico de cada execução.

Rotina semanal (5 minutos): abrir `conteudos`, revisar rascunhos, colar `image_url`, mudar para `aprovado`; abrir `fila_humana` e responder no Instagram o que ficou pendente (depois mude o `status` para `respondido`).

✅ Confirmação: resposta automática no comentário simples e pendência registrada no comentário clínico.

## Fase 12 - Deixar rodando 24 horas
Enquanto o computador estiver desligado, as automações não rodam. Opções:
- **Deixar o computador ligado** (com Docker Desktop aberto). Simples, mas frágil.
- **Servidor (VPS)**: um computador alugado na internet que fica ligado sempre (existem planos baratos e alguns provedores têm instalação pronta do n8n). Quando você chegar nesta fase, me diga o provedor escolhido e eu monto o passo a passo específico. Observação: o n8n Cloud (versão paga pelo próprio n8n) **não** serve sem adaptação, porque estes workflows leem as chaves do arquivo de configuração.

## Manutenção
- **Renovar o token do Instagram a cada ~50 dias** (vale 60; coloque um lembrete no calendário). Cole no navegador, trocando SEU_TOKEN:
  `https://graph.instagram.com/refresh_access_token?grant_type=ig_refresh_token&access_token=SEU_TOKEN`
  A resposta traz um `access_token` novo. Cole no `config.env` (linha `IG_ACCESS_TOKEN`) e, no terminal da pasta, rode `docker compose up -d --force-recreate`. Se o token expirar de vez, gere outro na Fase 3.4.
- Para reiniciar o n8n com novas configurações, **não basta "restart"**: use `docker compose up -d --force-recreate`.
- Para parar: `docker compose down`. Seus dados ficam guardados.
- Faça backup do `config.env` em local seguro (nunca no GitHub).

## Problemas comuns
| Sintoma | Provável causa / o que fazer |
|---|---|
| Erro "access to env vars denied" nos nós | O `docker-compose.yml` precisa ter `N8N_BLOCK_ENV_ACCESS_IN_NODE=false`; rode `docker compose up -d --force-recreate` |
| Nó de tabela sem tabela selecionada ou com erro de coluna | Abra o nó, escolha a tabela na lista e confira os nomes das colunas |
| Erro 401 / "Invalid API key" na Anthropic | Chave errada ou sem crédito na conta da API |
| Erro 190 / "token" / 400 do Instagram | Token expirado ou colado errado; gere outro (Fase 3.4) |
| "Media ID is not available" ao publicar | O Instagram ainda processava a imagem; aumente a espera no nó "Aguardar processamento" |
| Instagram recusa a imagem | Link não é público, não é JPEG ou proporção inválida |
| O workflow 3 não responde nada no teste manual | Normal: ele só funciona ativado (a memória não persiste em testes manuais) |
| Tela do Meta diferente deste guia | Mande uma captura de tela; as telas mudam |
