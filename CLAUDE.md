# CLAUDE.md

Este arquivo fornece orientações ao Claude Code (claude.ai/code) ao trabalhar com código neste repositório.

## Visão geral do repositório

Aplicação web de controle financeiro em arquivo único para **JM Transportes**
(empresa brasileira de transporte/logística). Toda a aplicação — markup,
estilos, estado, renderização e desenho de gráficos — vive em `index.html`
(~944 linhas). **Não há etapa de build, gerenciador de pacotes, testes ou
linter.** A página é aberta diretamente no navegador; CSS e bibliotecas de
runtime são carregados de CDNs (Tailwind 2.2.19, mais
React/ReactDOM/Recharts/Lucide que são referenciados mas não utilizados pelo
código de runtime).

Strings de UI, categorias e comentários estão em português brasileiro.
Preserve o texto pt-BR e a formatação de moeda BRL ao editar.

## Executando e "buildando"

- **Abrir localmente:** abra `index.html` em qualquer navegador, ou sirva o
  diretório: `python3 -m http.server 8000` e acesse
  `http://localhost:8000/`.
- **Sem build, sem instalação, sem testes.** Não adicione `package.json`,
  bundler ou framework a menos que solicitado explicitamente — a propriedade
  "arquivo único, zero build" é intencional (veja o comentário na linha ~78:
  *"Arquivo único HTML puro (sem build, sem framework)"*).
- **CI:** `.github/workflows/jekyll-docker.yml` executa um build Docker do
  Jekyll, mas seu filtro `branches:` é uma string de placeholder
  (`git-branch--m-main-<BRANCH>-...`) que nunca casará com uma branch real,
  então o workflow efetivamente nunca roda. Não suponha que o CI bloqueia
  qualquer coisa; se a publicação Jekyll for desejada, o filtro de branch
  precisa ser corrigido primeiro.

## Arquitetura

A aplicação usa um loop de renderização vanilla-JS feito à mão — **não**
React, apesar do React estar incluído nas tags `<script>`. O bloco Babel
`<script type="text/babel">` (linha ~69) está vazio; todo o código real vive
no bloco `<script>` simples logo abaixo.

**Ciclo de renderização (`index.html`):**
1. `state` (linha ~123) é um único objeto mutável contendo a aba ativa,
   quatro arrays de dados, estado de modal e texto de busca.
2. `setState(patch)` muta `state` via `Object.assign` e chama `render()`.
3. `render()` reconstrói `#app-root.innerHTML` a partir de `buildApp()`,
   depois chama `bindEvents()` para reanexar listeners do DOM e
   `renderCharts()` para redesenhar os canvases. **Cada mudança de estado
   re-renderiza a página inteira** — não há diffing. Novos elementos
   interativos precisam ser conectados dentro de `bindEvents()` ou ficarão
   inertes após o próximo render.
4. `persist(key, arr)` salva no `localStorage` *e* chama `setState`.

**Templating:** A UI é construída concatenando strings via helpers (`card`,
`btn`, `inputField`, `selectField`, `badge`) em strings de HTML. **Sempre
passe valores fornecidos pelo usuário por `escHtml()`** antes de interpolar
nesses templates — o helper `inputField` já faz isso para `value`, mas
templates escritos à mão frequentemente não fazem.

**Gráficos:** Desenhados diretamente em `<canvas>` com o contexto 2D em
`renderBarChart()` e `renderPieChart()`. Recharts é carregado mas não
utilizado; prefira estender o código de canvas existente em vez de trazer
Recharts.

**Ícones:** SVGs inline em strings no objeto `ICONS` (linha ~179). Lucide é
carregado mas não utilizado.

## Modelo de domínio

Quatro coleções, todas persistidas no `localStorage` sob estas chaves
exatas (forma curta, não renomeie sem migração):

| Campo de `state` | Chave do localStorage | Formato do item (campos principais)                                                                              |
| ---------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `lancamentos`    | `jm:lanc`             | `{id, data, descricao, categoria, tipo: "Receita"\|"Despesa", valor}`                                            |
| `boletos`        | `jm:bol`              | `{id, fornecedor, valor, vencimento, categoria, status: "A Pagar"\|"Pago"}`                                      |
| `notas`          | `jm:nota`             | `{id, cliente, numNota, valor, vencimento, status: "A Receber"\|"Recebido"}`                                     |
| `dividas`        | `jm:div`              | `{id, credor, tipo, valorOriginal, saldo, parcelaTotal, parcelaPaga, valorParcela, proxVenc, status: "Ativo"\|"Quitado"}` |

Convenções a manter consistentes:

- `id` = `uid()` (timestamp + aleatório). Nunca reutilize nem reordene por
  id.
- Datas são armazenadas como strings ISO `YYYY-MM-DD`; agrupamento mensal
  usa `iso.slice(0,7)` para comparar contra `todayISO().slice(0,7)`. Não
  introduza objetos `Date` em registros armazenados.
- Dinheiro é armazenado como `Number` JS e renderizado via `fmtBRL` /
  `fmtCompact`. Toda exibição passa por esses helpers.
- As listas de categorias `CATEGORIAS_RECEITA` / `CATEGORIAS_DESPESA`
  (linha ~90) são a única fonte da verdade para os selects de
  receita/despesa.
- Strings de status são rótulos pt-BR voltados ao usuário e são comparadas
  diretamente (`status==="Pago"`, etc.). Não traduza nem altere a
  capitalização.

Abas (`state.tab`): `dashboard`, `lancamentos`, `boletos`, `notas`,
`dividas` — mantenha essas strings literais sincronizadas entre
`buildApp()`, `buildBottomNav()` e qualquer nova seção.

## Paleta de cores

Um único objeto `C` no topo do arquivo (linha ~81) define a paleta da
marca em azul-marinho/laranja. Reutilize `C.navy`, `C.primary`, `C.green`,
`C.red`, `C.amber`, `C.purple`, etc., em vez de fixar valores hex no
código.

## Observação sobre ARCHITECTURE.md

`ARCHITECTURE.md` contém um diagrama Mermaid genérico de
Cliente/Servidor/Banco que **não descreve este codebase** (esta aplicação
não tem servidor, nem API, nem banco — apenas `localStorage`). Trate-o
como um template residual, não como documentação autoritativa. Prefira
este arquivo.
