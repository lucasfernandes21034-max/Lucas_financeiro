# Prompt — App de gestão da LSF Soluções

> Cole este texto inteiro como primeira mensagem para o assistente de código.
> Antes de colar, preencha os campos marcados com `[PREENCHER]` na seção "Números que eu já sei".
> Se não souber algum, deixe o marcador: o assistente deve perguntar, não assumir.

---

## 1. Contexto

Sou consultor solo. Minha empresa é a **LSF Soluções** (Santa Maria/RS), optante pelo **Simples Nacional**. Trabalho sozinho, sem funcionários, e atendo escritórios contábeis e PMEs em três linhas de serviço:

1. **Gestão de Processos** — consultoria (mapeamento, melhoria e padronização de processos)
2. **Desenvolvimento de Software** — apps e automações sob medida
3. **Análise e Engenharia de Dados** — dashboards, pipelines, ETL

**Situação atual:** a empresa opera no vermelho. Faturo cerca de R$ 4.500/mês e isso não cobre custos fixos, pró-labore e impostos. Não sei com clareza:

- quais clientes e projetos dão lucro e quais dão prejuízo;
- quanto tempo gasto em cada projeto e quanto do meu tempo não vira receita;
- se o preço que cobro cobre meu custo-hora real;
- quanto preciso faturar por mês para empatar.

Hoje controlo tudo de forma dispersa (planilha, memória, extrato bancário). Não existe sistema nenhum.

## 2. Números que eu já sei

Use estes valores como padrão nas configurações e no seed. O que estiver em branco, pergunte.

| Item | Valor |
|---|---|
| Faturamento médio mensal atual | R$ 4.500 |
| Custos fixos mensais (VPS, softwares, contador, telefone etc.) | R$ [PREENCHER] |
| Pró-labore desejado por mês | R$ [PREENCHER] |
| Horas disponíveis para trabalho por mês (capacidade) | [PREENCHER] h (ex.: 160) |
| Meta de horas faturáveis por mês | [PREENCHER] h (ex.: 100) |
| Saldo em caixa hoje | R$ [PREENCHER] |
| Anexo do Simples em que estou enquadrado | Anexo [III ou V] |
| Alíquota efetiva atual do DAS | [PREENCHER] % |
| Receita bruta dos últimos 12 meses (RBT12) | R$ [PREENCHER] |
| Regime de reconhecimento da receita no PGDAS | [competência ou caixa] |

## 3. Objetivo do app

Um **app web, de uso exclusivo meu**, que responda a três perguntas, nesta ordem de prioridade:

1. **Estou ganhando ou perdendo dinheiro, e onde?** — por cliente, por projeto e por linha de serviço.
2. **Quanto vai entrar e sair de caixa nos próximos 90 dias?**
3. **O que preciso entregar, para quem e quando?**

Se precisar escolher entre fazer bem a pergunta 1 ou fazer as três pela metade, faça bem a pergunta 1.

## 4. Escopo do MVP

Implemente **somente** o que está aqui. Qualquer coisa fora disto vai para a seção 5.

### 4.1 Cadastros

- **Clientes:** razão social, nome fantasia, CNPJ (validar dígitos), contato (nome, e-mail, telefone), cidade/UF, status (`prospect`, `ativo`, `inativo`), observações.
- **Linhas de serviço:** as três acima, com nome e cor para o painel. Cadastro simples, editável.
- **Categorias de custo:** fixo/variável, nome (ex.: Infraestrutura, Software, Contador, Pró-labore, Impostos, Comercial).
- **Configurações da empresa:** os valores da seção 2 (custos fixos, pró-labore desejado, capacidade de horas, meta de horas faturáveis, anexo, alíquota, saldo inicial de caixa e data desse saldo).

### 4.2 Projetos e contratos

- Projeto vinculado a **um cliente** e **uma linha de serviço**.
- **Modelo de cobrança:** `preco_fechado`, `mensalidade` (recorrente) ou `por_hora`.
- Campos: nome, descrição, valor contratado (ou valor da mensalidade, ou valor-hora), horas estimadas, data de início, data de fim prevista, data de fim real, status (`proposta`, `ativo`, `pausado`, `concluido`, `cancelado`).
- **Marcos/entregas:** nome, data prevista, data real, valor a faturar (opcional), status. Concluir um marco com valor deve gerar automaticamente uma conta a receber.
- Para `mensalidade`: gerar as contas a receber mensais automaticamente a partir da data de início até o fim previsto (ou até 12 meses à frente se não houver fim).
- Para `por_hora`: a receita prevista é `horas apontadas faturáveis × valor-hora`; permitir "fechar o período" e gerar a conta a receber correspondente.

### 4.3 Apontamento de horas

- Registro **rápido**: projeto, data (padrão: hoje), horas (aceitar `1,5`, `1:30` e `90min`), descrição curta, flag faturável.
- A tela precisa funcionar em **menos de 15 segundos no celular**: último projeto usado pré-selecionado, botões de atalho para 0,5h / 1h / 2h, salvar com um toque.
- **Horas não faturáveis** com categoria interna: `comercial`, `administrativo`, `estudo`, `outro`. Não exigem projeto.
- Lista do dia/semana com edição inline e exclusão.
- Opcional, se couber sem atrasar o marco: cronômetro iniciar/parar que gera um apontamento.

### 4.4 Financeiro

**Contas a receber**
- Vinculadas a projeto (e marco, quando houver): descrição, valor, data de vencimento, data de recebimento, status (`previsto`, `faturado`, `recebido`, `atrasado`, `cancelado`).
- `atrasado` é calculado: vencimento passado e não recebido. Não é status gravado à mão.
- Permitir recebimento parcial (registrar valor recebido menor e manter saldo em aberto).

**Contas a pagar**
- Categoria, descrição, valor, vencimento, pagamento, status (`previsto`, `pago`, `atrasado`, `cancelado`).
- **Recorrência:** mensal, com dia do vencimento e data de fim opcional. O sistema gera as parcelas futuras para o horizonte de 90 dias.
- Custos variáveis vinculados opcionalmente a um projeto (ex.: API paga usada num projeto específico), para entrar na margem daquele projeto.

**Impostos (DAS)**
- Calcular o DAS estimado do mês = `receita bruta do mês × alíquota efetiva`.
- A alíquota é **configurável** e tem duas formas: (a) informada manualmente; (b) calculada pela tabela do Anexo (III ou V) a partir do RBT12, com a fórmula oficial `(RBT12 × alíquota nominal − parcela a deduzir) / RBT12`. Implemente (a) no primeiro marco e (b) depois.
- Lançar o DAS automaticamente como **conta a pagar** com vencimento no dia 20 do mês seguinte, categoria "Impostos". Recalcular quando a receita do mês mudar, enquanto a conta não estiver paga.
- Deixar explícito na tela que é uma **estimativa**, não substitui o PGDAS.

**Fluxo de caixa projetado (90 dias)**
- `saldo projetado(dia) = saldo atual + Σ a receber até o dia − Σ a pagar até o dia`.
- Mostrar em gráfico de linha e em tabela semanal. Destacar o primeiro dia em que o saldo fica negativo.
- Saldo atual = saldo inicial configurado + recebimentos realizados − pagamentos realizados desde a data do saldo inicial. Permitir ajuste manual do saldo com justificativa.

### 4.5 Painel (tela inicial)

Tudo abaixo deve ter **a fórmula visível num tooltip** e ser calculado no backend, com testes.

**Resultado do mês** (regime de competência e de caixa, lado a lado)
- Receita: competência = contas a receber com vencimento no mês; caixa = recebidas no mês.
- Custos fixos, custos variáveis, impostos estimados, pró-labore.
- Lucro/prejuízo = receita − custos − impostos − pró-labore. Vermelho se negativo.

**Custo-hora** (dois números)
- **Custo-hora meta** = `(custos fixos + pró-labore desejado + DAS estimado sobre a receita-meta) / meta de horas faturáveis`. É o piso de preço.
- **Custo-hora realizado** = `(custos fixos + pró-labore desejado + impostos do mês) / horas faturáveis apontadas no mês`. Mostra quanto cada hora vendida de fato custou.

**Rentabilidade por projeto**
- **Valor-hora efetivo** = `receita do projeto / horas apontadas no projeto`. Para `preco_fechado`, usar valor contratado (projeção) e valor recebido (realizado), ambos.
- **Margem** = `receita − custos variáveis do projeto − (horas apontadas × custo-hora meta)`.
- Comparar com o custo-hora meta: verde ≥ meta, amarelo entre 80 % e 100 %, vermelho abaixo.
- **Consumo de horas** = `horas apontadas / horas estimadas`, com barra de progresso.

**Ranking de clientes** por margem acumulada no período (padrão: últimos 90 dias), com receita, horas e valor-hora efetivo.

**Uso do tempo:** horas faturáveis × não faturáveis no mês, em percentual, com quebra por categoria interna.

**Ponto de equilíbrio**
- `faturamento mínimo mensal = (custos fixos + pró-labore desejado) / (1 − alíquota efetiva − % de custos variáveis sobre receita)`.
- Mostrar também em horas: `ponto de equilíbrio / valor-hora médio efetivo`.
- Mostrar a distância entre a receita atual e o ponto de equilíbrio.

**Alertas** (lista no topo do painel)
- Recebimento atrasado (dias de atraso e valor).
- Projeto com consumo de horas > 80 % da estimativa e ainda não concluído.
- Marco com data prevista vencida e sem data real.
- Caixa projetado negativo em algum dia dos próximos 90 dias (informar o dia e o valor).
- Custo-hora realizado acima do valor-hora efetivo médio do mês.

**Filtros globais** do painel: mês/período, linha de serviço, cliente.

## 5. Fora do MVP

Não implementar agora, mas **modelar para não bloquear depois**:

- Portal do cliente e multiusuário/multi-tenant (deixe `empresa_id` nas tabelas de negócio e `usuario_id` nos registros criados, mesmo que hoje haja um só).
- Emissão de NFS-e, integração bancária/OFX, conciliação.
- Propostas comerciais, CRM, funil de vendas.
- Relatórios em PDF, exportação contábil.
- Notificações por e-mail/WhatsApp.

## 6. Stack e restrições técnicas

- **Backend:** Python 3.12 + FastAPI + SQLAlchemy 2 + Alembic (migrations) + Pydantic v2. Testes com pytest.
- **Frontend:** React + TypeScript + Vite. Biblioteca de UI leve (shadcn/ui ou equivalente). Gráficos com Recharts. **PWA instalável** no celular.
- **Banco:** PostgreSQL 16.
- **Infra:** Docker Compose com três serviços (`api`, `web`, `db`) e um `docker-compose.prod.yml` para deploy via **Coolify** em VPS Hostinger. HTTPS fica a cargo do Coolify. Incluir `.env.example` e um script de backup (`pg_dump` diário).
- **Autenticação:** usuário único, e-mail + senha com hash (argon2 ou bcrypt), JWT em cookie `HttpOnly` + `Secure` + `SameSite=Lax`, rate limit no login, expiração e renovação. Sem cadastro aberto: o usuário é criado por comando/seed.
- **Dinheiro:** `NUMERIC(14,2)` no banco, `Decimal` no Python, string no JSON. **Nunca float.** Arredondamento `ROUND_HALF_UP` em duas casas em todas as fórmulas.
- **Horas:** `NUMERIC(6,2)` em horas decimais.
- **Datas:** `DATE` para vencimentos e apontamentos; `TIMESTAMPTZ` para auditoria. Fuso `America/Sao_Paulo`.
- **Interface:** português do Brasil; moeda `R$ 1.234,56`; datas `dd/mm/aaaa`; mobile-first; sem texto em inglês visível.
- **Código:** nomes de tabelas, colunas, variáveis e endpoints em **inglês**; textos de interface, mensagens de erro e documentação em **português**.
- Toda regra de cálculo fica em **módulo de domínio puro** (sem dependência de banco ou HTTP), para ser testada isoladamente.
- Soft delete (`deleted_at`) em clientes, projetos e lançamentos financeiros; apontamentos podem ser excluídos de verdade.
- Auditoria mínima: `created_at`, `updated_at` em tudo.

## 7. Como quero que você trabalhe

Siga este fluxo e **pare para minha aprovação entre as etapas**. Não avance sem meu "ok".

**Etapa 1 — Especificação (`SPEC.md`)**
- Modelo de dados: tabelas, colunas com tipo, chaves, índices, relacionamentos, e um diagrama ER em Mermaid.
- Regras de negócio escritas como frases testáveis ("Quando X, então Y").
- Fórmulas de todos os indicadores da seção 4.5, com um exemplo numérico cada.
- Máquinas de estado dos status (projeto, marco, conta a receber, conta a pagar).
- **Lista de dúvidas** antes de qualquer suposição. Se algo na seção 2 estiver em branco, pergunte.

**Etapa 2 — Plano (`PLANO.md`)**
- Marcos pequenos, cada um entregável e testável sozinho, em ordem de valor para mim.
- **O marco 1 obrigatoriamente** entrega: configurações da empresa + lançamento manual de receitas e custos + painel com resultado do mês, custo-hora e ponto de equilíbrio. Mesmo sem projetos ou horas. Quero ver meu número real na primeira semana.
- Para cada marco: o que entra, o que não entra, como eu testo, critério de pronto.

**Etapa 3 — Implementação marco a marco**
- Um marco por vez. Ao terminar, pare e me mostre.
- Testes unitários obrigatórios para: custo-hora (meta e realizado), valor-hora efetivo, margem por projeto, DAS (manual e por tabela), fluxo projetado, ponto de equilíbrio, geração de recorrências, cálculo de `atrasado`.
- Casos de borda que precisam ter teste: mês sem horas faturáveis (divisão por zero), projeto sem horas estimadas, recebimento parcial, mensalidade com fim no meio do mês, DAS quando receita é zero.
- Commits pequenos, com mensagem em português descrevendo o quê e por quê.
- Não adicione dependências sem me dizer o motivo.

**Etapa 4 — Verificação de cada marco**
- Como subir (`docker compose up`) e como rodar os testes.
- Roteiro de teste manual: o que clicar e **quais números devem aparecer** com o seed carregado.
- Tabela "indicador → valor esperado → como foi calculado".

## 8. Seed de dados

Script `seed.py` idempotente (pode rodar duas vezes sem duplicar), com dados realistas para eu validar os indicadores:

- Empresa configurada com os valores da seção 2.
- **3 clientes:** um escritório contábil (ativo, dois projetos), uma PME de comércio (ativo), um prospect sem projeto.
- **5 projetos**, cobrindo os três modelos de cobrança e as três linhas de serviço:
  1. Consultoria de processos, `preco_fechado`, R$ 6.000, 40 h estimadas, 3 marcos, um atrasado.
  2. Suporte mensal de BI, `mensalidade`, R$ 1.200/mês, 10 h/mês estimadas, ativo há 4 meses.
  3. Desenvolvimento de app, `preco_fechado`, R$ 9.000, 80 h estimadas, já com 70 h apontadas (deve disparar alerta de 80 %).
  4. Automação de dados, `por_hora`, R$ 90/h, 30 h apontadas em 2 meses.
  5. Projeto concluído e lucrativo, `preco_fechado`, para o ranking ter um caso verde.
- **2 meses de apontamentos** (mês atual e anterior), incluindo horas não faturáveis em todas as categorias, somando algo em torno de 60 % faturável / 40 % não faturável.
- **Contas a receber** com pelo menos uma atrasada, uma recebida parcialmente e uma prevista no futuro.
- **Contas a pagar** recorrentes (VPS, 2 softwares, contador, pró-labore) e uma variável vinculada ao projeto 4.
- Resultado esperado do seed: o painel deve mostrar **prejuízo no mês atual**, custo-hora realizado acima do valor-hora efetivo, e um projeto no vermelho. Documente em `SEED.md` os valores que devem aparecer.

## 9. Critérios de aceite do MVP

Considero o MVP pronto quando, com o seed carregado:

1. Abro o painel no celular e em menos de 5 segundos vejo lucro/prejuízo do mês, custo-hora e ponto de equilíbrio.
2. Aponto 1,5 h num projeto pelo celular em menos de 15 segundos e o painel reflete na hora.
3. Concluo um marco com valor e a conta a receber aparece no fluxo de caixa.
4. Mudo a alíquota do DAS e o imposto estimado, o lucro e o ponto de equilíbrio recalculam.
5. Vejo qual cliente e qual projeto me dão prejuízo, com o número que prova.
6. Vejo o primeiro dia em que meu caixa fica negativo nos próximos 90 dias.
7. Todos os testes passam e `docker compose up` sobe do zero sem passo manual além do `.env`.

## 10. Primeira ação

Comece pela **Etapa 1**. Antes de escrever o `SPEC.md`, me envie a lista de dúvidas. Depois da minha resposta, escreva a especificação completa e pare.
