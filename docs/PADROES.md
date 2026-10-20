# Padrões de desenvolvimento – Portfólio

Guia único para todos os repositórios do portfólio. Vale para qualquer linguagem.

**Regra zero:** a convenção oficial da linguagem e do framework vence sempre.
Isso vale para nomes, estrutura de pastas, logger, validação, injeção de
dependência, testes e front-end. Se o framework já traz a peça (Log do Laravel,
ILogger do .NET, logging do Python, FormRequest do Laravel, DI do .NET), usa-se
a peça do framework. Só se cria algo próprio onde não existe nada pronto.
As regras abaixo descrevem o espírito; o apêndice 12 diz como cada stack o aplica.

## 1. Origem destas regras

Extraídas de quatro projetos reais (SIMM.Builder, SIMM.LoginLDAP, PRXTREM L17,
SIMM.TrainControllerL17). O que se repete nos quatro é o estilo da casa. O que
só apareceu nos commits coautorados por IA foi deixado de fora de propósito.

## 2. Arquitetura (todas as linguagens)

- MVC em camadas: Route > Middleware > Controller > Service > Model.
- Injeção de dependência pelo construtor. Em Node, um Container manual com
  getters lazy (`getProductService()`); em .NET e Laravel, o container nativo.
- Tratamento de erro em três camadas (padrão validado no SIMM.LoginLDAP):
  - Model faz I/O puro, sem try/catch. A exceção sobe.
  - Service lança erro de domínio com código e status
    (`throw new DomainError('order.not_found', 404)`). Só captura quando existe
    um fallback legítimo (leitura não crítica que pode devolver `null`).
  - Controller **não tem try/catch**. Um único tradutor de exceções, na base
    ou no framework, responde o status e a mensagem do erro de domínio. Qualquer
    outra exceção vira 500 genérico com log; a causa técnica nunca vai ao cliente.
  - Em Node, handlers de controller são métodos de protótipo, não arrow class
    fields, senão o wrapper da base não os alcança.
- Zero-Crash: o processo nunca morre por erro de aplicação. Handlers globais de
  exceção só logam.
- Validação declarativa separada do controller, com a peça nativa da stack
  (FormRequest no Laravel, FluentValidation ou DataAnnotations no .NET, Pydantic
  no FastAPI, Zod ou FormRequest próprio em Node).
- Contrato de resposta JSON: `{ success, message?, data? }` no sucesso e
  `{ success: false, message, errors? }` no erro. Validação 422, autorização 403.
- Logger do framework quando existe (Laravel `Log`, .NET `ILogger<T>`, Python
  `logging`). Em Node, um logger próprio ou pino. Toda mensagem leva o contexto
  (nome da classe). Nada de `console.log` ou `print` fora do logger.
- Config em arquivo versionado + `.env.local` fora do git. Segredos nunca no repo.
- Eventos nomeados `entidade.acao` (`order.created`, `payment.approved`).
  Tópicos Kafka seguem o mesmo nome.

## 3. Nomes

Regra geral: **cada linguagem segue a convenção oficial dela** (tabelas no
apêndice 12). O que é comum a todas:

| Coisa | Convenção | Exemplo |
|---|---|---|
| Classe | sufixo da camada | `OrderService`, `ProductModel`, `StoreOrderRequest` |
| Arquivo de classe | mesmo nome da classe | `OrderService.js`, `OrderService.cs` |
| Coluna de banco | snake_case | `created_at`, `is_active`, `travel_time_s` |
| Campo JSON da API | camelCase | `createdAt`, `isActive` |
| Rota | kebab-case, sem prefixo `/api/` | `/orders/pending`, `/products/by-seller` |
| Tópico Kafka | `entidade.acao` | `order.created`, `payment.approved` |
| Chave de i18n | pontos | `order.not_found` |
| Classe CSS | kebab-case; BEM só em componente isolado | `.product-card`, `.empty-state--full` |
| Repositório no GitHub | kebab-case com prefixo do produto | `<produto>-orders` |

Sufixo de unidade em campo numérico: `_s` (segundos), `_ms`, `_cents`.

## 4. Idioma

| Onde | Idioma |
|---|---|
| Identificadores | inglês |
| Comentários | pt-BR, com acento |
| Logs | pt-BR curto, termos técnicos em inglês |
| Mensagens de erro e UI | pt-BR via i18n (`lang('chave')`), nunca texto fixo na view |
| Nomes de teste | pt-BR |
| Commits | inglês |
| README e docs | pt-BR, com acento |

## 5. Comentários (o ponto mais importante contra "cara de IA")

1. Cabeçalho de arquivo só quando o arquivo precisa de contexto: 2 a 4 linhas,
   sem autor, sem data, sem `@see` para doc inexistente.
2. Docblock (`/** */`, `///`, docstring) só em método público de service, helper
   ou API. Getter, setter, construtor e método óbvio ficam sem.
3. Comentário de linha explica **por quê**, nunca **o quê**.
   Bom: `// maxAge === 0 desliga a expiração`.
   Ruim: `// incrementa o contador`.
4. Separador de seção, quando necessário, no formato simples:
   `// ==== Listeners ====`. Sem caixas ASCII, sem linhas de 40 colunas.
5. Ênfase em maiúscula só para regra de negócio crítica (`// REGRA: ...`).
6. Proibido: travessão, seta unicode, símbolos matemáticos, emoji, `TODO` sem
   issue, comentário em inglês no meio de código comentado em português.

## 6. Formatação

- 4 espaços. Aspas simples em JS/PHP/Python; a convenção nativa em C#.
- Ponto e vírgula sempre em JS. `const` por padrão, `let` só ao reatribuir.
- `async/await`, nunca `.then()` encadeado.
- Guard clause de uma linha: `if (!id) return null;`
- Alinhamento vertical em listas de rotas, mapas e factories é aceito.
- Linha até 120 colunas.
- Vírgula final em objeto multilinha.
- Os projetos originais não têm linter. No portfólio **vai ter**
  (ESLint + Prettier, PHP-CS-Fixer, dotnet format, ruff), porque é esperado em
  projeto público. As regras acima viram a config do linter.

## 7. Testes

- Sempre. Node: Jest com `test()`, nunca `it()`. .NET: xUnit. PHP: PHPUnit.
  Python: pytest.
- `describe('OrderService.create')` por classe e método.
- Nome do teste em pt-BR, presente, sem "deve":
  `test('retorna null quando o pedido não existe')`.
- Mocks à mão (objeto literal + `jest.fn()`), sem `jest.mock` de módulo.
- Banco em teste: SQLite `:memory:` ou MySQL em container do CI.
- Um arquivo de teste por classe, mesmo nome + `.test`.

## 8. Commits e branches

- Conventional Commits em minúsculo: `feat`, `fix`, `refactor`, `test`, `docs`,
  `chore`, `build`, `merge`. Escopo opcional entre parênteses: `feat(orders):`.
- Assunto em inglês, minúsculo, 50 a 80 caracteres, concreto:
  `fix(orders): reject order when stock is zero`.
- Corpo só quando o assunto não basta. Duas a quatro linhas de prosa curta.
  Sem lista de bullets com `[NEW]`/`[MODIFY]`, sem contagem de testes, sem
  linha `Co-Authored-By`.
- Commits pequenos e frequentes. Um commit de feature e um de teste é ok.
- Branches: `main`, `develop`, `feature/nome-curto`, `fix/nome-curto`.
  Merge com `merge: feature/x into develop`.
- Quem commita e faz push é a Deborah, sempre.

## 9. README (esqueleto único para todos os repos)

```
# Nome do projeto
Uma frase do que é e qual papel tem no ecossistema.

## Stack
## Como rodar (local e docker)
## Arquitetura (diagrama + árvore de pastas comentada)
## Rotas ou eventos (tabela)
## Decisões (lista curta: por que X e não Y)
## Testes
## Licença (MIT)
```

- Sem badges em excesso (no máximo CI e licença). Sem seção "Contributing".
- Diagramas em Mermaid. Alertas `> [!NOTE]` e `> [!WARNING]` quando precisar.
- Changelog em `CHANGELOG.md` no formato Keep a Changelog, com data.

## 10. Front-end

Depende da stack do serviço, não de uma escolha única para o portfólio:

| Serviço | Front-end |
|---|---|
| Laravel (painel do vendedor) | Blade + Bootstrap 5 + jQuery ou Alpine |
| Node com tela (auth, catálogo) | EJS + Bootstrap 5 + jQuery |
| Vitrine pública | React + TypeScript |
| App do comprador | React Native |
| Avulsos | React (player), Vue (filmes) |

Regras comuns:

- Componentes e hooks em PascalCase e `useNome` quando houver framework.
  Estado local primeiro; store global só quando dois componentes distantes
  precisam da mesma coisa.
- CSS: tokens em `:root` (`--primary`, `--bg`, `--text-muted`), tema escuro por
  `prefers-color-scheme` e `data-theme`. Regra curta fica numa linha.
- Dependência por npm ou Composer, não por CDN. Fontes locais ou Google Fonts.
- Feedback ao usuário por toast; erro de formulário no campo.

## 11. O que nunca aparece

- Emoji em código, log, commit ou comentário. No README, no máximo um no título.
- Abstração sem segundo uso (interface com uma implementação, factory genérica,
  repository separado do model).
- Classe de erro HTTP genérica; erro de domínio tem código e status próprios.
- Comentário descrevendo o óbvio. Comentário em bloco de 10 linhas.
- `console.log`, `var`, `.then()`, `any` em TypeScript.
- Arquivo com mais de 300 linhas sem motivo.
- `Co-Authored-By`, "generated with", assinatura de ferramenta.

## 12. Apêndice por linguagem

### Node.js / JavaScript
| Coisa | Convenção |
|---|---|
| Função, variável, método | camelCase |
| Classe | PascalCase |
| Constante | UPPER_SNAKE |
| Método privado | `#nome` (campo privado nativo) |
| Arquivo de helper/middleware | camelCase: `priceHelper.js`, `verifySession.js` |
| Id de DOM | camelCase: `#btnSave` |

- ESM (`"type": "module"`, `import`/`export`). Imports relativos ou `#services`
  via `imports` do package.json; sem module-alias.
- Express 4, mysql2 ou better-sqlite3, Jest com `--experimental-vm-modules`.
- Erro de domínio: classe `DomainError(code, status, params)` + wrapper na
  `BaseController` que traduz para `{ success: false, message }` e status.

### TypeScript / React
| Coisa | Convenção |
|---|---|
| Componente e arquivo dele | PascalCase: `ProductCard.tsx` |
| Hook | `useNome`: `useCart` |
| Interface e type | PascalCase, sem prefixo `I` |
| Enum e membros | PascalCase |
| Demais | igual ao JavaScript |

- `strict: true`. Props tipadas por `interface`. Sem `any`.
- Pastas `components/`, `pages/`, `hooks/`, `services/`.

### C# / .NET 8
| Coisa | Convenção |
|---|---|
| Classe, método, propriedade, namespace | PascalCase |
| Interface | `I` + PascalCase: `IOrderRepository` |
| Parâmetro e variável local | camelCase |
| Campo privado | `_camelCase` |
| Constante | PascalCase |
| Projeto e pasta | `Produto.Orders`, `Produto.Orders.Tests` |

- Minimal API ou controllers com `[ApiController]`. EF Core com migrations.
- Erro de domínio: exceção própria + `IExceptionHandler` devolvendo `ProblemDetails`.
- Logger: `ILogger<T>` injetado. DI: o container nativo.
- `dotnet format` no CI. Comentários `//` em pt-BR. XML doc só em API pública.

### PHP / Laravel
| Coisa | Convenção |
|---|---|
| Classe | PascalCase (PSR-1) |
| Método e variável | camelCase |
| Constante | UPPER_SNAKE |
| Tabela | snake_case plural: `orders` |
| Model Eloquent | singular: `Order` |
| Rota nomeada | ponto: `orders.show` |
| Arquivo de view | kebab-case: `order-list.blade.php` |

- PSR-12, PHP-CS-Fixer. FormRequest nativo, Eloquent, PHPUnit.
- Erro de domínio: exceção própria com `render()` ou registro no `Handler`.
- Logger: facade `Log`. DI: o container do Laravel.
- Rotas em `routes/web.php` e `routes/api.php` sem prefixo extra.

### Python
| Coisa | Convenção |
|---|---|
| Função, variável, método, módulo, pacote | snake_case |
| Classe | PascalCase |
| Constante | UPPER_SNAKE |
| Privado | `_nome` |

- PEP 8, ruff, pytest, type hints em tudo. FastAPI para serviço HTTP, Pydantic
  para validação. Nome de teste: `test_retorna_none_quando_pedido_nao_existe`.
- Erro de domínio: exceção própria + `@app.exception_handler`.
- Logger: módulo `logging` com um logger por módulo (`getLogger(__name__)`).
