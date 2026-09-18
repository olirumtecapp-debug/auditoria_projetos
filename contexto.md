# CONTEXTO DO ECOSSISTEMA

**Leia este documento inteiro antes de trabalhar em qualquer projeto.**

Atualizado em: 2026-09-17

---

## Regra número 1 — este documento

Tudo o que for feito de novo tem de ser atualizado **aqui**. Este é o único lugar que explica o que existe, o que foi feito e como se trabalha. Se você mudar algo e não registrar aqui, o processo se perde.

---

## 1. Mapa do ecossistema

| Projeto | Pasta local | Repositório (GitHub) | Produção |
|---|---|---|---|
| Escola da Fé | `D:\EscolaDaFe` | `escola-da-fe` | `escoladafe.creativeam.com.br` |
| Catecismo | `D:\Catecismo` | `catecismo-catolico` | `catecismo.creativeam.com.br` |
| ELEVATE (Inglês) | `D:\InglesPassoAPasso` (trabalho) · `D:\ProjetosGITHUB\ingles-passo-a-passo` (git) | `ingles-passo-a-passo` | `elevate.creativeam.com.br` |
| CodeLogic PRO | `D:\ProjetosGITHUB\codelogic-pro` | `codelogic-pro` | `codelogic.creativeam.com.br` |
| CreativeAM (portfólio) | `D:\CreativeAM` | (ainda sem repositório remoto) | a publicar |
| Auditoria + Processos | `D:\ProjetosGITHUB\auditoria_projetos` | `auditoria_projetos` | `auditoria-projetos.vercel.app` |

Conta GitHub: `olirumtecapp-debug`. Conta Vercel: `creativeam-pij`.

**Backend:** Firestore (projeto `expedicao-brasil`) acessado por API REST, sem segredo no código. As funções ficam em `api/` nos projetos (Vercel Functions).

---

## 2. Protocolo obrigatório de trabalho

1. **Backup local antes de alterar** qualquer arquivo (cópia com data/hora, ou zip).
2. **Não mexer no que já funciona** sem autorização expressa.
3. **Commit + push** no GitHub do projeto.
4. **Deploy e validação no ar**: testar em produção e entregar o link testado.
5. **Não usar `push --force`** sem autorização.
6. **Nunca gravar segredo** em arquivo do repositório: token e senha só em variável de ambiente da Vercel.

**O git não está no PATH.** Use o caminho completo:

```
C:\Program Files\Verdent\resources\app.asar.unpacked\node_modules\dugite\git\cmd\git.exe
```

---

## 3. Padrões do ecossistema

### 3.1 Acesso do aluno (aplicado nos 4 projetos)

- Cadastro com **nome e e-mail**, sem senha e sem login social.
- **PIN opcional** (4 a 6 dígitos), guardado só como hash no servidor.
- **Código de recuperação** gerado no cadastro, mostrado uma única vez.
- **Recuperação por e-mail**: o aluno pede o código, o servidor gera um novo e envia (Resend).
- **Canal de ajuda** na tela de entrada: "não consigo entrar — falar com a coordenação".
- **Redefinir acesso** no painel: remove o PIN e gera um código novo para entregar ao aluno.
- Quem **concede** acesso é a coordenação: nunca o próprio aluno.
- O PIN **jamais** pode ser gravado dentro do estado do aluno (isso já travou um aluno real).

### 3.2 Caixa postal (aplicada nos 4 projetos)

- **Comunicados vêm do servidor** — aviso guardado no navegador só aparece naquele aparelho.
- **Aluno**: vê o estado e a resposta, arquiva e exclui da própria caixa.
- Excluir pelo aluno é **ocultar para ele** (campo `ocultadaParaAluno`): a coordenação mantém o registro.
- **Coordenação**: filtros (ativas, novas, respondidas, arquivadas), responder, arquivar e excluir, mais o gerenciador de comunicados publicados.
- Um endpoint único de atualização com `action` (evita o limite de 12 funções da Vercel).

### 3.3 Publicação, domínio e cache

- **Tailwind hospedado no próprio site** (`assets/vendor/tailwind.js`). Nunca depender de CDN externo: foi o que deixou o layout quebrado no celular.
- `vercel.json`: rewrite que **exclui `/api/`**; site com páginas separadas usa `handle: filesystem` antes do catch-all.
- Cada projeto tem seu subdomínio `.creativeam.com.br` com CNAME para `cname.vercel-dns.com`.
- Plano gratuito da Vercel: **no máximo 12 funções por deploy**. Se passar, o deploy falha e o site fica na versão anterior.
- O deployer precisa estar ligado ao repositório (projeto sem integração com o GitHub não publica com push).

### 3.4 E-mail (Resend)

- Domínio verificado: `contato.creativeam.com.br`. Remetente: `CreativeAM <contato@contato.creativeam.com.br>` com resposta para `contato@creativeam.com.br`.
- Variáveis na Vercel: `RESEND_API_KEY`, `EMAIL_REMETENTE`, `EMAIL_RESPOSTA`.
- Sem essas variáveis o envio fica desligado e as telas caem no canal manual — sem quebrar.

### 3.5 Pagamento PIX / Asaas (webhook)

- **POST** `/api/asaas-webhook` recebe do Asaas e aceita apenas `PAYMENT_RECEIVED`, `PAYMENT_CONFIRMED` e `PAYMENT_RECEIVED_IN_CASH`.
- **GET** `/api/asaas-webhook?value=VALOR&since=TIMESTAMP` é a consulta que o frontend faz a cada 5 segundos enquanto o modal do PIX está aberto; responde `{approved:true|false}`.
- Ao cadastrar no painel do Asaas, a URL é sempre `https://<domínio do projeto>/api/asaas-webhook`:

| Projeto | URL para cadastrar no Asaas |
|---|---|
| Escola da Fé | `https://escoladafe.creativeam.com.br/api/asaas-webhook` |
| Catecismo | `https://catecismo.creativeam.com.br/api/asaas-webhook` |
| ELEVATE | `https://elevate.creativeam.com.br/api/asaas-webhook` |
| CodeLogic | `https://codelogic.creativeam.com.br/api/asaas-webhook` |

- **Onde a aprovação é guardada (atualizado em 17/09/2026):**

| Projeto | Aprovação | Concede acesso automaticamente |
|---|---|---|
| Escola da Fé | Firestore | **sim — marca o aluno como apoiador no servidor** |
| ELEVATE | Firestore | **sim — grava o VIP no servidor para o e-mail que pagou** |
| Catecismo | Firestore | não (doação) |
| CodeLogic | Firestore | **sim — grava o VIP no cadastro do aluno que pagou** |

- Antes a aprovação ficava na **memória do processo**: em serverless cada chamada pode cair em outra instância e o site nunca via o pagamento (o clássico "paguei e não liberou"). Hoje os quatro gravam no Firestore e liberam o acesso **no próprio webhook**, sem depender do navegador do aluno estar aberto.

- **Teste rápido** (deve responder `approved:false`, nunca 404): abrir a URL de GET com `?value=50` no navegador.
- **Teste completo**: simular um POST com `PAYMENT_RECEIVED`, conferir que o GET passa a responder `approved:true`, e depois limpar o estado de teste.

---

## 4. O que já foi feito

### 4.1 Segurança (2026-09-15)

- Um token do GitHub estava exposto em repositório **público** e foi **revogado**. As referências foram limpas de 58 remotes, 16 scripts e da documentação.
- Nenhum segredo ficou em arquivo versionado. As credenciais passaram a viver em variável de ambiente.

### 4.2 E-mail e recuperação de acesso (2026-09-16)

- Camada de e-mail (Resend) e envio do código de recuperação em **Escola da Fé, Catecismo, ELEVATE e CodeLogic**.
- Cupons VIP migrados do navegador para o **servidor**, com validade e uso único por aluno.
- **VIP gravado no servidor** (o painel concede, o aluno lê ao abrir). O aluno não consegue mais se promover.
- Pedido de ajuda sem estar logado chega por e-mail para a coordenação.
- **Cadastro obrigatório no ELEVATE** ao abrir o app (antes dava para usar o curso inteiro sem se identificar).

### 4.3 Caixa postal (2026-09-16)

- Ciclo de vida completo nos 4 projetos: nova, lida, respondida, arquivada; arquivar e excluir pelo aluno; filtros e ações na coordenação.
- Comunicados passaram a vir do servidor, então aparecem em **todos** os aparelhos.

### 4.4 Curso do ELEVATE — auditoria (2026-09-17)

- Auditados **285 lições e 1.215 exercícios**: inglês íntegro; 150 exercícios de espanhol estavam no esquema antigo e foram migrados **sem alterar a quantidade**.
- Regra freemium corrigida: vale pela **posição da unidade dentro do nível** (o espanhol numera as unidades em sequência: 6–10 no intermediário), não pelo número bruto.
- Tradução passou a aceitar resposta equivalente ("Eu tenho…" para "Tenho…") e ignora acentuação; o exercício repete no máximo uma vez.

### 4.5 Mobile (2026-09-16)

- Tailwind passou a ser servido pelo próprio site nos 4 projetos (era a causa do layout quebrado no celular).

### 4.6 Pagamento PIX / Asaas (2026-09-17)

- Documentado o contrato do webhook nos 4 projetos e conferido que os 4 endpoints estão **no ar** (`approved:false`, sem 404).
- **Aprovação migrada da memória para o Firestore nos 4 projetos** — o polling nunca mais perde o pagamento em serverless.
- **Acesso liberado no próprio webhook** (não depende do aluno estar com a página aberta): ELEVATE e CodeLogic concedem o VIP ao e-mail que pagou; a Escola da Fé marca o aluno como apoiador.
- Testado ponta a ponta no ELEVATE: pagamento enviado → `approved:true` → **VIP liberado automaticamente**; e nos quatro: aprovação gravada e limpa depois.
- **Pendente crítico:** `codelogic-pro/api/create-pix.js` tem a **chave de produção do Asaas escrita no código** (base64). O repositório é público, então essa chave precisa ser movida para variável de ambiente na Vercel e depois **rotacionada no painel do Asaas**.

### 4.7 ELEVATE — Sincronização Cloud de Alunos & Correção do Painel ADM (2026-09-17)

- **Causa raiz:** O arquivo `index.html` foi sobrescrito por uma versão legada de `lingoclone.html` (de 15/09), perdendo a função `carregarAlunosDaNuvem` e fazendo o painel ADM depender apenas de `localStorage` local. Com isso, novos cadastros (como o aluno Murilo Martins) sumiram do painel ao trocar de aba ou atualizar.
- **Dados intactos no Firestore:** O cadastro de alunos na nuvem (`elevate_students`) nunca foi perdido; a falha era puramente de exibição no frontend.
- **Solução implementada:**
  - Restaurada a versão moderna de 9.707 linhas de `index.html` a partir do backup `index.html.bak_2026-09-17_11-25`.
  - Chave Asaas mantida 100% sanitizada (`const ASAAS_API_KEY = "";`).
  - Funções `refreshAdminAnalyticsData` e `carregarAlunosDaNuvem` refatoradas para consultar **sempre** a rota `/api/admin/students` (Firestore).
  - Adicionado helper `getElevateApiBase()` para permitir carregar os alunos da nuvem mesmo quando rodando em ambiente local (VSCode Live Server ou `file:///`).
  - Arquivos `index.html` e `lingoclone.html` sincronizados e idênticos.
  - Commit e push realizados na branch `main` (`c91a600`) com deploy confirmado na Vercel.

### 4.8 Catecismo — Correção Hagiográfica de Santas & Prompts IA (2026-09-17)

- **Problema:** Na raspagem original dos dados do Vaticano, a abreviação genérica "S." fez com que 43 mulheres santas fossem importadas incorretamente com o prefixo masculino "São" (ex: *São Clara de Assis*, *São Maria Madalena*, *São Marta*, *São Mônica*, *São Faustina*, *São Bárbara*, etc.).
- **Solução implementada:**
  - Backup preventivo compactado em `backups/santos_2026_pre_santa_fix_2026-09-17.zip`.
  - Script automatizado com validação de nomes canônicos femininos, títulos (virgem, abadessa, viúva, freira, imperatriz) e texto biográfico corrigiu as 43 santas para o título oficial **"Santa"** nos arquivos diários de 2026 (`data/santos/2026/`).
  - Todos os 365 arquivos JSON do ano validados com sucesso.
  - O painel de auditoria (`test_santos_dinamico.html`) e o gerador de prompts foram adaptados com detecção de gênero, gerando prompts de IA em português e inglês com vestimentas sacras femininas históricas adequadas (hábito de clarissa, véu tradicional, túnicas nobres de mártir, etc.).

### 4.9. Catecismo: Caixa Postal Completa e Mural de Comunicados Oficiais (18/09/2026)

- **Contexto**: O Catecismo possuía envio de mensagens pelo fiel, mas faltava o canal oficial de comunicados da coordenação e a capacidade do administrador responder às mensagens diretamente pelo painel (o botão era apenas um link `mailto:`). O usuário solicitou essa funcionalidade para poder enviar comunicados a todos os fiéis sobre a atualização do banco de imagens dos santos com notificação visual para o usuário.
- **O que foi feito**:
  - **Firestore / Banco de dados**: Adicionada a coleção `catecismo_broadcasts` e funções `getBroadcastsDatabase()`, `saveBroadcastToDatabase()` e `deleteBroadcastInDatabase()` em `api/_db.js`.
  - **Ciclo de vida de mensagens**: Suporte à ação `reply` em `api/contact/update-status.js` para registrar a resposta no Firestore, gravar `repliedAt` e mudar o status para `respondido`.
  - **Endpoint de Comunicados**: Criado `api/admin/broadcasts.js` consolidado (GET para listar, POST para criar e excluir com `action: 'delete'`), mantendo o projeto rigorosamente dentro do limite de 12 funções da Vercel.
  - **Servidor local**: `server.mjs` atualizado para rotear as funções serverless localmente na porta 3000.
  - **Painel Administrativo (`index.html`)**: Adicionada seção "Mural de Comunicados Oficiais" com formulário de publicação e listagem com botão de exclusão; na tabela de mensagens recebidas, adicionado botão de ação "💬 Responder" integrado.
  - **Interface do Fiel (`index.html`)**: Adicionada aba "📢 Mural de Comunicados" dentro da Caixa Postal, badges de não lidos no cabeçalho e menu lateral, e banner na tela inicial ("📢 Novo comunicado da Coordenação") quando houver comunicado não visualizado, limpando o badge assim que o fiel abre o modal.

### 4.10 Catecismo — Sincronização Inteligente (Smart Sync), Compartilhar Evangelho no WhatsApp, Desafio Rápido e Luz Espiritual (18/09/2026)

- **Smart Sync Bidirecional (`CloudSyncManager`)**:
  - Resolvido o problema de divergência de XP e dias de ofensiva entre smartphone e PC (730 XP no banco vs desatualizado no celular).
  - O sistema agora faz auto-pull ao iniciar se houver e-mail logado: compara timestamp e XP da nuvem com o local. Se a nuvem tiver mais XP ou for mais recente, mescla e atualiza os stats locais sem sobrescrever reflexões; se o local avançou offline, envia para a nuvem.
  - No `server.mjs`, as rotas `/api/cloud-sync/save` e `/api/cloud-sync/load` foram unificadas aos handlers oficiais do Firestore, eliminando a divergência com o arquivo local `cloud_users.json`.
- **Compartilhamento Litúrgico do Evangelho no WhatsApp**:
  - Adicionado botão de compartilhamento com a fórmula litúrgica da Igreja Católica: *"📖 Proclamação do Evangelho de Jesus Cristo segundo..."*, texto integral e aclamação de encerramento *"— Palavra da Salvação. — Glória a Vós, Senhor."*.
- **Desafio Rápido do Dia (Home)**:
  - Card compacto com 1 pergunta rápida rotativa baseada nos módulos do Catecismo (CCC), pontuando +15 XP com som e explicação doutrinária.
- **Uma Luz para o Seu Dia (Home)**:
  - Card de acolhimento espiritual com versículo, doutrina do Catecismo, jaculatória/prece e botão de envio rápido no WhatsApp.

---

## 5. Armadilhas conhecidas

- **Sincronização unilateral de progresso (apenas PUSH / SAVE no startup)**: se o aplicativo só faz upload do estado local ao abrir, ele nunca puxa pontos conquistados em outro aparelho e pode sobrescrever a nuvem com um estado defasado. A sincronização ao abrir precisa ser inteligente (smart sync com auto-pull e merge).
- **Sobrescrita cega de arquivos gêmeos (`lingoclone.html` -> `index.html`)**: Nunca copiar um arquivo antigo por cima de um mais novo sem antes checar data de modificação e quantidade de linhas. O arquivo principal servido pela Vercel é o `index.html`.
- **Listagem de alunos baseada em `localStorage`**: O painel ADM nunca deve confiar exclusivamente no cache do navegador local para listar alunos. A listagem deve ser sempre puxada da API do servidor (`/api/admin/students`), caso contrário novos alunos de outros aparelhos somem da tela.
- **Abreviação italiana "S." em dados religiosos**: Em fontes italianas ou latinas, "S." serve para *San* e *Santa*. Importações cegas transformam mulheres em "São".
- **PIN dentro do estado do aluno** vira PIN de quem não pediu e trava o acesso. Foi um bug real.
- **Regra de acesso pelo número bruto da unidade** quebra quando o nível numera as unidades em sequência (caso do espanhol).
- **Função da Vercel por ação** estoura o limite de 12 do plano gratuito. Use um endpoint com `action`.
- **Aviso guardado no navegador** não aparece em outro aparelho: comunicado tem de vir do servidor.
- **Cache de leitura de 3 segundos** faz o teste parecer errado logo após gravar: aguarde antes de conferir.
- **Página aberta antes do deploy** continua rodando o código antigo: feche e reabra para testar.
- **Excluir de verdade** quando o aluno pede faz a coordenação perder o histórico: ocultar é o certo.
- **Copiar texto de um projeto para outro** sem verificar se aquilo existe no destino. Já aconteceu: aviso de certificado no ELEVATE, que não emite certificado.
- **Auto-declaração de acesso**: se o campo de booleano de acesso for escrito pelo aparelho do aluno, qualquer um vira VIP.

---

## 6. Como manter este documento atualizado

1. Ao terminar qualquer mudança, **atualize a seção 4** com o que foi feito (data, projeto e o que mudou).
2. Se aparecer um problema novo, registre em **Armadilhas conhecidas** — uma linha basta.
3. Se criar um padrão novo, registre na **seção 3**.
4. Este documento fica em `processos.html` / `contexto.md` no repositório `auditoria_projetos`. Commit e push: a página publica sozinha.
