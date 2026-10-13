# Folgas · Leão do Parque

Escala de folgas e extras da Churrascaria Leão do Parque. Página única: tudo (HTML, CSS e JS) fica em `index.html`, sem build e sem dependências locais.

- Repositório: https://github.com/adrianpastore/folgas-leao (branch `main`)
- Banco: Firebase Firestore, projeto `folgas-leao` (SDK modular 10.12.2 via CDN, `persistentLocalCache`). A config fica em `firebaseConfig` no próprio `index.html`.
- Login por e-mail e senha (Firebase Auth). As contas são criadas só no console do Firebase; o app não tem cadastro. Todas as contas liberadas veem e editam tudo.

## Fluxo de trabalho combinado

- Falar em português, explicar as mudanças em linguagem simples.
- Commit ao fim de cada etapa. **Push só com o ok do usuário** (muda o site público).
- Custo zero: nada que exija o plano Blaze do Firebase.
- Antes de commitar: checar a sintaxe do `<script type="module">` (extrair para um `.mjs` e rodar `node --check`).

## Telas

- **Calendário › Semana**: os 7 dias (segunda a domingo). Cada dia aberto mostra, em passos:
  1. **Quem folga**: toque no nome marca/tira a folga. Domingo: **1 por mês** por pessoa (quem já tem domingo no mês fica desabilitado).
  2. **Quem cobre**: para cada folga, "Chamar extra", "Vendeu a folga" ou "Não precisa". O extra passa por *Ainda não → Confirmou / Recusou*; recusou = escolher outro.
  3. **Reforço**: extras chamados em dia de movimento, mesmo sem folga.
  - **Lista de extras e pagamento**: diárias do dia e da semana, com Pix de cada um e o resumo pronto para copiar.
  - Botões de **feriado** (diária de domingo) e **fechar/abrir neste dia** (abertura especial).
  - "Escala da semana": texto para o WhatsApp (`wa.me`).
- **Calendário › Mês**: visão do mês, resumo (folgas, quem está sem domingo), extras do mês com total e mensagem do mês.
- **Equipe**: fixos (função, área, folga fixa semanal, chave Pix) e extras (função, área, WhatsApp, chave Pix). **Áreas e funções**: diária por área e por dia da semana.
- Filtro por área (Salão, Churrasqueira, Cozinha) no topo; vale para tudo.
- Badge na aba Calendário: pendências dos próximos 14 dias.

## Regras de negócio

- Dias abertos por padrão: `cfg.open` (dias da semana; segunda fechada). `cfg.exc[data]` = abertura ou fechamento especial.
- **Abertura especial num dia normalmente fechado** (ex.: segunda feriado): cada fixo precisa escolher **outro dia de folga** nos 13 dias seguintes (`dias[k].comp`), e a diária é a de domingo.
- **Folga fixa** (`fixo.fw`): dias da semana em que a pessoa sempre folga (nunca domingo, nunca em dia de exceção). Dá para trocar no dia (`dias[k].on`).
- **Vendeu a folga**: a pessoa trabalha no dia da folga. Se for **domingo**, entra na lista de extras do dia com a diária da área dela.
- **Diária**: `rate(área, dia)` usa `area.vd[diaDaSemana]`; feriado (`cfg.fer` ou abertura especial) usa o valor de domingo.
- Na lista de pagamento só entram extras **confirmados** e fixos que **venderam o domingo**.

## Dados

Tudo num único documento: `app/estado` = `{ v, fixos, extras, dias, cfg, _by, _at }`.

| Campo | Observação |
|---|---|
| `fixos[]` | `{ id, n (nome), r (função), a (área), pix, fw: [dias da semana] }` |
| `extras[]` | `{ id, n, r, a, t (WhatsApp), pix }` |
| `dias[AAAA-MM-DD]` | `{ off: [ids], on: [ids], cob: { idFixo: { e (extra), s (chamado/confirmado/recusou), v (vendeu), n (não precisa), rec: [recusaram] } }, ref: [{ id, e, s, a, rec }], comp: { idFixo: dia } }` |
| `cfg` | `{ open, seg, exc, fer, areas: [{ id, n, v, vd, f (funções) }], vv }` |

- `norm()` migra dados antigos (v1/v2) e marca `_mig` para regravar.
- Gravação com atraso de 450 ms (`push`), documento inteiro com `setDoc`. `_by`/`_at` evitam reprocessar a própria gravação.
- Cópia local no `localStorage` (`folgas_extras_v2`) e cache offline do Firestore.

## Login e regras

- Tela de login (e-mail e senha) antes do app. O `onSnapshot` só começa depois do login. "Sair" na aba Equipe.
- `firestore.rules` no repositório é a cópia versionada, **com e-mails de exemplo**; o que vale é o publicado no console, com os e-mails reais. Libera só o documento `app/estado`; o resto fica fechado.
- No console: desmarcar "Enable create (sign-up)" em Authentication > Settings > User actions.
- Limitação: o documento é gravado inteiro; se duas pessoas editarem ao mesmo tempo, vale a última gravação.

## Ideias para depois

- "Esqueci a senha" na tela de login (como no eventos-leao).
- Saber quem fez cada alteração.
