# Composition Root - Injeção de Dependências

## 📚 Visão Geral

O **Composition Root** é o **único lugar** na aplicação onde:
- Todas as **dependências são criadas**
- O **grafo de dependências** é montado
- As **implementações concretas** são conectadas

**Localização**: `main.ts` (entrypoint da aplicação)

**Princípio**: "New is Glue" - O operador `new` é a "cola" que conecta componentes, e deve estar isolado em um único lugar.

---

## 📄 main.ts - Composition Root Completo

```typescript
import ContractDatabaseRepository from "./infra/repository/ContractDatabaseRepository";
import ExpressAdapter from "./infra/http/ExpressAdapter";
import GenerateInvoices from "./application/usecase/GenerateInvoices";
import JsonPresenter from "./infra/presenter/JsonPresenter";
import LoggerDecorator from "./application/decorator/LoggerDecorator";
import MainController from "./infra/http/MainController";
import PgPromiseAdapter from "./infra/database/PgPromiseAdapter";
import Mediator from "./infra/mediator/Mediator";
import SendEmail from "./application/usecase/SendEmail";

const connection = new PgPromiseAdapter();
const contractRepository = new ContractDatabaseRepository(connection);
const mediator = new Mediator();
const sendEmail = new SendEmail();
mediator.on("InvoicesGenerated", async function (data: any) {
	await sendEmail.execute(data);
});
const generateInvoices = new LoggerDecorator(new GenerateInvoices(contractRepository, new JsonPresenter(), mediator));
const httpServer = new ExpressAdapter();
new MainController(httpServer, generateInvoices);
httpServer.listen(3000);
```

---

## 🔍 Explicação Técnica Linha por Linha

### Imports - Todas as Dependências

```typescript
import ContractDatabaseRepository from "./infra/repository/ContractDatabaseRepository";
import ExpressAdapter from "./infra/http/ExpressAdapter";
import GenerateInvoices from "./application/usecase/GenerateInvoices";
import JsonPresenter from "./infra/presenter/JsonPresenter";
import LoggerDecorator from "./application/decorator/LoggerDecorator";
import MainController from "./infra/http/MainController";
import PgPromiseAdapter from "./infra/database/PgPromiseAdapter";
import Mediator from "./infra/mediator/Mediator";
import SendEmail from "./application/usecase/SendEmail";
```

#### 🔍 Análise dos Imports

**Características:**
- Imports de **classes concretas**, não interfaces
- Único lugar na aplicação que importa implementações específicas
- Outras camadas importam apenas abstrações

**Camadas Representadas:**
- **Infrastructure**: `PgPromiseAdapter`, `ExpressAdapter`, `ContractDatabaseRepository`, `JsonPresenter`, `MainController`
- **Application**: `GenerateInvoices`, `LoggerDecorator`, `SendEmail`
- **Mediator**: `Mediator`

**Note que NÃO importa:**
- Interfaces (elas são importadas por quem as usa)
- Domain entities (criadas pelo repositório)
- Outras estratégias/presenters (apenas as que serão usadas)

---

### Linha 11: Database Connection

```typescript
const connection = new PgPromiseAdapter();
```

#### 🔍 Explicação Técnica

**O que cria:**
- Instância de `PgPromiseAdapter`
- Conexão com PostgreSQL é estabelecida

**Características:**
- **Singleton implícito**: Uma única instância criada
- Será **compartilhada** por todos que precisam de DB
- **Importante**: Não fecha a conexão neste código (deveria ter shutdown handler)

**Melhorias Possíveis:**

```typescript
// 1. Configuração externa
const databaseUrl = process.env.DATABASE_URL || "postgres://localhost/app";
const connection = new PgPromiseAdapter(databaseUrl);

// 2. Graceful shutdown
process.on('SIGTERM', async () => {
    await connection.close();
    process.exit(0);
});
```

**Por que criar primeiro?**
- Será injetado no `ContractDatabaseRepository`
- Deve existir antes de ser usado

---

### Linha 12: Repository

```typescript
const contractRepository = new ContractDatabaseRepository(connection);
```

#### 🔍 Explicação Técnica

**Dependency Injection Manual:**
- Cria `ContractDatabaseRepository`
- **Injeta** `connection` via constructor
- Repository agora pode acessar banco de dados

**Diagrama de Dependência:**
```
contractRepository (ContractDatabaseRepository)
    ↓ depende de
connection (PgPromiseAdapter → DatabaseConnection interface)
```

**Tipo Real vs Tipo Aparente:**
```typescript
const contractRepository = new ContractDatabaseRepository(connection);
// Tipo real: ContractDatabaseRepository
// Poderia ser tipado como: ContractRepository (interface)

// Mais explícito:
const contractRepository: ContractRepository = new ContractDatabaseRepository(connection);
```

**Por que não usar interface aqui?**
- Não há necessidade de polimorfismo no Composition Root
- Composition Root **conhece** implementações concretas
- Tipagem explícita é opcional mas recomendada para clareza

---

### Linha 13: Mediator

```typescript
const mediator = new Mediator();
```

#### 🔍 Explicação Técnica

**Singleton do Mediator:**
- Uma única instância para toda aplicação
- Será compartilhada entre use cases e observers
- **Importante**: Mesmo mediator usado em vários lugares

**Responsabilidade:**
- Coordenar eventos entre componentes
- Publish/Subscribe pattern central

---

### Linha 14: SendEmail Use Case

```typescript
const sendEmail = new SendEmail();
```

#### 🔍 Explicação Técnica

**Observer Use Case:**
- Será registrado como observer de eventos
- Não tem dependências (constructor vazio)
- Em produção, receberia serviço de email injetado

**Melhor Implementação:**
```typescript
const emailService = new SendGridAdapter(process.env.SENDGRID_API_KEY);
const sendEmail = new SendEmail(emailService);
```

---

### Linhas 15-17: Registro de Observer

```typescript
mediator.on("InvoicesGenerated", async function (data: any) {
	await sendEmail.execute(data);
});
```

#### 🔍 Explicação Técnica Detalhada

**Configuração de Event Listener:**

**Linha 15: Registra observer**
```typescript
mediator.on("InvoicesGenerated", async function (data: any) {
```
- Evento: `"InvoicesGenerated"`
- Callback: Função assíncrona anônima

**Linha 16: Executa use case**
```typescript
await sendEmail.execute(data);
```
- Quando evento é publicado, executa `sendEmail`
- `data`: Será o output de `GenerateInvoices`

**Fluxo de Evento:**
```
1. GenerateInvoices termina
   ↓
2. Publica "InvoicesGenerated" com output
   ↓
3. Mediator encontra observer registrado
   ↓
4. Executa callback
   ↓
5. sendEmail.execute(data) é chamado
   ↓
6. Email é enviado (em teoria)
```

**Benefícios:**
- `GenerateInvoices` não conhece `SendEmail`
- Adicionar novos observers é trivial
- Desacoplamento total

**Múltiplos Observers:**
```typescript
mediator.on("InvoicesGenerated", async (data) => {
    await sendEmail.execute(data);
});

mediator.on("InvoicesGenerated", async (data) => {
    await logToAnalytics.execute(data);
});

mediator.on("InvoicesGenerated", async (data) => {
    await updateCache.execute(data);
});
```
- Todos serão executados quando evento for publicado
- Ordem de execução: Ordem de registro

---

### Linha 18: Use Case Principal com Decorator

```typescript
const generateInvoices = new LoggerDecorator(new GenerateInvoices(contractRepository, new JsonPresenter(), mediator));
```

#### 🔍 Explicação Técnica Detalhada

**Composição Complexa - Vamos Decompor:**

**Nível 1: JsonPresenter**
```typescript
new JsonPresenter()
```
- Cria instância de presenter
- Sem dependências

**Nível 2: GenerateInvoices**
```typescript
new GenerateInvoices(contractRepository, new JsonPresenter(), mediator)
```
- Cria use case principal
- **Injeta 3 dependências**:
  1. `contractRepository`: Criado na linha 12
  2. `new JsonPresenter()`: Criado inline
  3. `mediator`: Criado na linha 13

**Nível 3: LoggerDecorator**
```typescript
new LoggerDecorator(new GenerateInvoices(...))
```
- **Envolve** use case com decorator
- Adiciona funcionalidade de logging
- Retorna algo que ainda implementa `Usecase`

**Diagrama de Camadas:**
```
LoggerDecorator
    ↓ envolve
GenerateInvoices
    ↓ usa
contractRepository, presenter, mediator
    ↓ dependem de
connection
```

**Versão Explícita (mais legível):**
```typescript
const presenter = new JsonPresenter();
const generateInvoicesCore = new GenerateInvoices(contractRepository, presenter, mediator);
const generateInvoices = new LoggerDecorator(generateInvoicesCore);
```

**Decorator Chain Complexo:**
```typescript
const generateInvoices = 
    new ErrorHandlerDecorator(
        new TimerDecorator(
            new LoggerDecorator(
                new ValidationDecorator(
                    new GenerateInvoices(contractRepository, presenter, mediator)
                )
            )
        )
    );
```
- Cada decorator adiciona funcionalidade
- Ordem importa: De dentro para fora na execução

**Problema: JsonPresenter Hardcoded**
```typescript
new GenerateInvoices(contractRepository, new JsonPresenter(), mediator)
```
- Sempre usa JSON
- Para usar CSV, precisa modificar código
- **Melhor**: Injetar presenter como variável

**Solução - Configurável:**
```typescript
const format = process.env.OUTPUT_FORMAT || 'json';
const presenter = format === 'csv' ? new CsvPresenter() : new JsonPresenter();
const generateInvoices = new LoggerDecorator(
    new GenerateInvoices(contractRepository, presenter, mediator)
);
```

---

### Linha 19: HTTP Server

```typescript
const httpServer = new ExpressAdapter();
```

#### 🔍 Explicação Técnica

**Cria Servidor HTTP:**
- Instância de `ExpressAdapter`
- Express app é criado internamente
- Middlewares são configurados (JSON parser)

**Ainda não está escutando:**
- Servidor criado mas não iniciado
- `listen()` será chamado depois

**Alternativa - Outro Framework:**
```typescript
// Se quiser trocar Express por Fastify:
const httpServer = new FastifyAdapter();
// Resto do código permanece igual!
```
- Demonstra valor do Adapter Pattern
- Mudança localizada no Composition Root

---

### Linha 20: Controller

```typescript
new MainController(httpServer, generateInvoices);
```

#### 🔍 Explicação Técnica

**Conecta HTTP com Application:**
- Cria `MainController`
- **Injeta**:
  1. `httpServer`: ExpressAdapter criado na linha 19
  2. `generateInvoices`: Use case decorado criado na linha 18

**Note: Não armazena em variável**
```typescript
new MainController(httpServer, generateInvoices);
// Equivalente a:
const mainController = new MainController(httpServer, generateInvoices);
// Mas mainController não é usado depois, então pode omitir
```

**O que acontece internamente:**
- Controller registra rotas no httpServer
- Rota POST `/generate_invoices` é criada
- Quando requisição chegar, chamará `generateInvoices.execute()`

**Diagrama de Conexão:**
```
HTTP Request → ExpressAdapter → MainController → LoggerDecorator → GenerateInvoices
```

**Múltiplos Controllers:**
```typescript
new MainController(httpServer, generateInvoices);
new ContractController(httpServer, getContract, createContract);
new PaymentController(httpServer, processPayment);
```
- Cada controller registra suas rotas
- Todos compartilham mesmo httpServer

---

### Linha 21: Inicia Servidor

```typescript
httpServer.listen(3000);
```

#### 🔍 Explicação Técnica

**Inicia HTTP Server:**
- Servidor começa a escutar na porta 3000
- Aplicação agora está pronta para receber requisições

**Por que porta 3000?**
- Convenção comum em Node.js
- Deveria vir de configuração

**Melhor Implementação:**
```typescript
const port = process.env.PORT ? parseInt(process.env.PORT) : 3000;
httpServer.listen(port);
console.log(`Server listening on port ${port}`);
```

**Ordem Importa:**
1. Criar servidor
2. Registrar rotas (via controllers)
3. **Depois** chamar listen()

**Chamadas Assíncronas:**
```typescript
// listen() poderia retornar Promise
await httpServer.listen(3000);
console.log('Server started successfully');
```

---

## 🎯 Grafo de Dependências Completo

```
main.ts (Composition Root)
│
├─ connection: PgPromiseAdapter
│   └─ implements DatabaseConnection
│
├─ contractRepository: ContractDatabaseRepository
│   ├─ depends on → connection
│   └─ implements ContractRepository
│
├─ mediator: Mediator
│   └─ observers: []
│
├─ sendEmail: SendEmail
│   └─ registered in mediator
│
├─ generateInvoices: LoggerDecorator
│   └─ wraps → GenerateInvoices
│       ├─ depends on → contractRepository
│       ├─ depends on → JsonPresenter
│       └─ depends on → mediator
│
├─ httpServer: ExpressAdapter
│   └─ implements HttpServer
│
└─ mainController: MainController
    ├─ depends on → httpServer
    └─ depends on → generateInvoices
```

---

## 📊 Análise de Dependências por Camada

### Infrastructure → Application
```
ContractDatabaseRepository → ContractRepository (interface)
JsonPresenter → Presenter (interface)
ExpressAdapter → HttpServer (interface)
PgPromiseAdapter → DatabaseConnection (interface)
```
✅ **Correto**: Infra depende de abstrações da Application

### Application → Domain
```
GenerateInvoices → Contract (entity)
ContractRepository → Contract (entity)
```
✅ **Correto**: Application usa entities do Domain

### Main → Tudo
```
main.ts → All concrete implementations
```
✅ **Correto**: Composition Root conhece tudo

### Domain → Nada (Exceto Moment.js)
```
Domain não depende de outras camadas
```
⚠️ **Quase**: Moment.js é dependência externa no domain

---

## 🔧 Princípios Aplicados no Composition Root

### 1. Dependency Inversion Principle (DIP)

**Antes (errado):**
```typescript
// Em GenerateInvoices
class GenerateInvoices {
    constructor() {
        this.repository = new ContractDatabaseRepository(new PgPromiseAdapter());
        // Use case conhece implementação concreta!
    }
}
```

**Depois (correto):**
```typescript
// Em GenerateInvoices
class GenerateInvoices {
    constructor(readonly repository: ContractRepository) {
        // Use case conhece apenas interface
    }
}

// Em main.ts
const connection = new PgPromiseAdapter();
const repository = new ContractDatabaseRepository(connection);
const useCase = new GenerateInvoices(repository);
// Composition Root conecta as peças
```

### 2. Single Responsibility Principle (SRP)

**Responsabilidade do main.ts:**
- ✅ Criar instâncias
- ✅ Conectar dependências
- ✅ Configurar aplicação
- ❌ Lógica de negócio
- ❌ Validações
- ❌ Transformações de dados

### 3. Open/Closed Principle (OCP)

**Adicionar novo Presenter:**
```typescript
// Sem modificar GenerateInvoices
const presenter = new XmlPresenter();
const generateInvoices = new GenerateInvoices(repository, presenter, mediator);
```

**Adicionar novo Decorator:**
```typescript
const generateInvoices = 
    new CacheDecorator(
        new LoggerDecorator(
            new GenerateInvoices(repository, presenter, mediator)
        )
    );
```

---

## 🚀 Melhorias e Alternativas

### 1. Dependency Injection Container

**Problema Atual:**
- Criação manual de todas dependências
- Ordem de criação importa
- Difícil gerenciar em aplicações grandes

**Solução - DI Container:**
```typescript
import { Container } from 'inversify';

const container = new Container();
container.bind<DatabaseConnection>('DatabaseConnection').to(PgPromiseAdapter).inSingletonScope();
container.bind<ContractRepository>('ContractRepository').to(ContractDatabaseRepository);
container.bind<Usecase>('GenerateInvoices').to(GenerateInvoices);

const generateInvoices = container.get<Usecase>('GenerateInvoices');
```

**Bibliotecas:**
- InversifyJS
- TypeDI
- TSyringe
- Awilix

### 2. Factory Functions

**Em vez de new direto:**
```typescript
function createGenerateInvoicesUseCase(
    repository: ContractRepository,
    presenter: Presenter,
    mediator: Mediator
): Usecase {
    const core = new GenerateInvoices(repository, presenter, mediator);
    return new LoggerDecorator(core);
}

const generateInvoices = createGenerateInvoicesUseCase(
    contractRepository,
    jsonPresenter,
    mediator
);
```

### 3. Configuration Object

```typescript
interface AppConfig {
    database: {
        url: string;
    };
    server: {
        port: number;
    };
    output: {
        format: 'json' | 'csv';
    };
}

const config: AppConfig = {
    database: {
        url: process.env.DATABASE_URL || 'postgres://localhost/app'
    },
    server: {
        port: parseInt(process.env.PORT || '3000')
    },
    output: {
        format: (process.env.OUTPUT_FORMAT || 'json') as 'json' | 'csv'
    }
};

const connection = new PgPromiseAdapter(config.database.url);
const presenter = config.output.format === 'csv' 
    ? new CsvPresenter() 
    : new JsonPresenter();
// ...
httpServer.listen(config.server.port);
```

### 4. Graceful Shutdown

```typescript
// Ao final de main.ts
const shutdown = async () => {
    console.log('Shutting down gracefully...');
    await connection.close();
    console.log('Database connection closed');
    process.exit(0);
};

process.on('SIGTERM', shutdown);
process.on('SIGINT', shutdown);
```

### 5. Health Check

```typescript
httpServer.on('get', '/health', async () => {
    try {
        await connection.query('SELECT 1', []);
        return { status: 'healthy', database: 'connected' };
    } catch (error) {
        return { status: 'unhealthy', database: 'disconnected' };
    }
});
```

---

## 🎓 Padrões Arquiteturais Aplicados

### Composition Root Pattern

**Definição:**
- Único lugar onde `new` é usado para criar dependências
- Monta grafo de objetos completo
- Resto da aplicação usa Dependency Injection

**Benefícios:**
1. **Centralização**: Todas dependências em um lugar
2. **Visibilidade**: Fácil ver como aplicação é montada
3. **Testabilidade**: Fácil substituir implementações
4. **Manutenção**: Mudanças localizadas

**Princípio:**
> "Poor Man's DI" - Injeção de dependência manual sem container

### Registry Pattern (Mediator)

```typescript
mediator.on("InvoicesGenerated", handler);
```
- Registra observers em registry central
- Desacopla publishers de subscribers

### Decorator Pattern (Chain)

```typescript
new LoggerDecorator(new GenerateInvoices(...))
```
- Composição de comportamentos
- Configurado no Composition Root

---

## 📝 Checklist de Boas Práticas

### ✅ O que o main.ts faz bem:

- ✅ Centraliza criação de dependências
- ✅ Usa injeção de dependência manual
- ✅ Não contém lógica de negócio
- ✅ Demonstra todos os patterns do projeto
- ✅ Configuração de mediator no lugar certo

### ⚠️ O que poderia melhorar:

- ⚠️ Configurações hardcoded (porta, database URL)
- ⚠️ Sem tratamento de erros de inicialização
- ⚠️ Sem graceful shutdown
- ⚠️ Sem logging de startup
- ⚠️ JsonPresenter criado inline (poderia ser variável)
- ⚠️ Sem validação de variáveis de ambiente

---

## 🔗 Conclusão

O `main.ts` é o **coração** da aplicação que conecta todas as peças. É aqui que:

1. **Implementações concretas** são escolhidas
2. **Dependências são injetadas** manualmente
3. **Decorators são aplicados** para adicionar funcionalidades
4. **Eventos são configurados** via mediator
5. **Servidor é iniciado** e aplicação fica pronta

Este arquivo demonstra como **Clean Architecture** e **Design Patterns** se unem para criar uma aplicação **flexível**, **testável** e **manutenível**.

---

## 📚 Documentação Relacionada

- `00-VISAO-GERAL.md` - Overview completo do projeto
- `01-DOMAIN-LAYER.md` - Entities, Value Objects, Strategy, Factory
- `02-APPLICATION-LAYER.md` - Use Cases, Repository, Presenter, Decorator
- `03-INFRASTRUCTURE-LAYER.md` - Adapters, Implementations, Mediator, Controller
