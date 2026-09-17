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

- **Onde a aprovação é guardada hoje:**

| Projeto | Aprovação | Concede acesso automaticamente |
|---|---|---|
| Escola da Fé | Firestore (persiste) | não — marca o aluno como apoiador |
| ELEVATE | memória do processo | **sim — grava o VIP no servidor para o e-mail que pagou** |
| Catecismo | memória do processo | não |
| CodeLogic | memória do processo | não |

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
- O webhook do **ELEVATE** passou a **conceder o VIP no servidor** para o e-mail que pagou — quem paga recebe o acesso ao abrir o app.
- Registrada a limitação: em Catecismo, ELEVATE e CodeLogic a aprovação fica na **memória do processo** (em serverless o polling pode não ver a aprovação). Só a Escola da Fé guarda no Firestore.

---

## 5. Armadilhas conhecidas

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
