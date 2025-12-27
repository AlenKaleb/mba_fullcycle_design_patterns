# Camada de Infraestrutura (Infrastructure Layer)

## 📚 Visão Geral

A camada de infraestrutura contém **implementações concretas** e **detalhes técnicos**:
- **Adapters**: Adaptam bibliotecas externas às interfaces da aplicação
- **Repositories**: Implementam persistência em banco de dados
- **Presenters**: Formatam saída em formatos específicos
- **HTTP**: Controllers e servidores web
- **Mediator**: Implementa padrão publish/subscribe

**Princípio**: Esta camada **depende** das camadas internas (Application, Domain), mas elas **não dependem** dela (DIP).

---

## 1️⃣ Adapter Pattern - Database

### 📄 DatabaseConnection.ts - Interface

```typescript
export default interface DatabaseConnection {
	query (statement: string, params: any): Promise<any>;
	close (): Promise<void>;
}
```

#### 🔍 Explicação Técnica

**Adapter Pattern - Interface:**
- Define **contrato** para conexões de banco de dados
- Abstrai detalhes de bibliotecas específicas (pg-promise, mysql, etc.)
- Permite trocar implementação sem afetar código cliente

**Métodos:**

**1. `query()`:**
```typescript
query (statement: string, params: any): Promise<any>
```
- Executa query SQL
- `statement`: SQL com placeholders
- `params`: Parâmetros para substituir placeholders
- Retorna `Promise<any>`: Resultado genérico

**2. `close()`:**
```typescript
close (): Promise<void>
```
- Fecha conexão com banco
- Importante para liberar recursos
- Assíncrono porque pode precisar finalizar transações

**Benefícios da Abstração:**
1. **Testabilidade**: Mock/stub em testes
2. **Portabilidade**: Trocar PostgreSQL por MySQL
3. **ISP**: Interface mínima, sem métodos específicos de biblioteca

---

### 📄 PgPromiseAdapter.ts - Implementação PostgreSQL

```typescript
import DatabaseConnection from "./DatabaseConnection";
import pgp from "pg-promise";

export default class PgPromiseAdapter implements DatabaseConnection {
	connection: any;

	constructor () {
		this.connection = pgp()("postgres://postgres:123456@localhost:5432/app");
	}

	query(statement: string, params: any): Promise<any> {
		return this.connection.query(statement, params);
	}

	close(): Promise<void> {
		return this.connection.$pool.end();
	}

}
```

#### 🔍 Explicação Técnica Detalhada

**Adapter Pattern em Ação:**
- **Adaptee**: `pg-promise` (biblioteca externa)
- **Target Interface**: `DatabaseConnection`
- **Adapter**: `PgPromiseAdapter`
- **Cliente**: `ContractDatabaseRepository`

**Diagrama do Pattern:**
```
[Repository] → usa → [DatabaseConnection interface]
                              ↑
                              | implementa
                              |
                     [PgPromiseAdapter]
                              ↓ adapta
                         [pg-promise lib]
```

**Inicialização da Conexão:**
```typescript
constructor () {
    this.connection = pgp()("postgres://postgres:123456@localhost:5432/app");
}
```

**Análise:**
- `pgp()`: Cria factory do pg-promise
- String de conexão **hardcoded** ⚠️
  - Deveria vir de variáveis de ambiente
  - Problemas: credenciais no código, inflexível
- Conexão criada no constructor
  - Boa: Lazy initialization não necessária
  - Ruim: Dificulta testes

**⚠️ Problemas de Segurança:**
```typescript
"postgres://postgres:123456@localhost:5432/app"
```
- Senha em plaintext no código
- **Solução**: Usar variáveis de ambiente

```typescript
// Melhor implementação:
constructor () {
    const connectionString = process.env.DATABASE_URL || 
        "postgres://postgres:123456@localhost:5432/app";
    this.connection = pgp()(connectionString);
}
```

**Método `query()`:**
```typescript
query(statement: string, params: any): Promise<any> {
    return this.connection.query(statement, params);
}
```
- Delega diretamente para `pg-promise`
- Não adiciona lógica extra
- **Adapter puro**: Apenas adapta interface

**Método `close()`:**
```typescript
close(): Promise<void> {
    return this.connection.$pool.end();
}
```
- `$pool`: API específica do pg-promise
- Fecha pool de conexões
- Cliente não precisa conhecer `$pool`

**Vantagens do Adapter:**
1. **Desacoplamento**: Repository não conhece pg-promise
2. **Substituibilidade**: Fácil trocar por MySQL, MongoDB, etc.
3. **Testabilidade**: Mock interface em testes

**Desvantagens:**
1. **Camada extra**: Indireção adicional
2. **Funcionalidades perdidas**: Recursos específicos do pg-promise não expostos
3. **Type safety reduzida**: `any` em vez de tipos específicos

---

## 2️⃣ Repository Pattern - Implementação

### 📄 ContractDatabaseRepository.ts

```typescript
import AccrualBasisStrategy from "../../domain/AccrualBasisStrategy";
import Contract from "../../domain/Contract";
import ContractRepository from "../../application/repository/ContractRepository";
import DatabaseConnection from "../database/DatabaseConnection";
import Payment from "../../domain/Payment";

export default class ContractDatabaseRepository implements ContractRepository {

	constructor (readonly connection: DatabaseConnection) {
	}

	async list(): Promise<Contract[]> {
		const contracts: Contract[] = [];
		const contractsData = await this.connection.query("select * from branas.contract", []);
		for (const contractData of contractsData) {
			const contract = new Contract(contractData.id_contract, contractData.description, parseFloat(contractData.amount), contractData.periods, contractData.date);
			const paymentsData = await this.connection.query("select * from branas.payment where id_contract = $1", [contract.idContract]);
			for (const paymentData of paymentsData) {
				contract.addPayment(new Payment(paymentData.id_payment, paymentData.date, parseFloat(paymentData.amount)));
			}
			contracts.push(contract);
		}
		return contracts;
	}

}
```

#### 🔍 Explicação Técnica Detalhada

**Repository Pattern - Implementação Concreta:**
- Implementa interface `ContractRepository` da camada application
- Responsável por **reconstruir aggregates** do banco de dados
- Usa `DatabaseConnection` (abstração) para acessar DB

**Dependency Injection:**
```typescript
constructor (readonly connection: DatabaseConnection) {
}
```
- Recebe `DatabaseConnection` (interface)
- Não cria conexão internamente
- **Benefício**: Testável, flexível, DIP aplicado

**Método `list()` - Reconstrução de Aggregates:**

**Passo 1: Inicializa array de resultado** (linha 13)
```typescript
const contracts: Contract[] = [];
```

**Passo 2: Busca todos os contratos** (linha 14)
```typescript
const contractsData = await this.connection.query("select * from branas.contract", []);
```
- Query SQL direto
- `[]`: Array vazio de parâmetros (nenhum placeholder)
- `await`: Operação assíncrona

**Passo 3: Itera sobre dados de contratos** (linha 15)
```typescript
for (const contractData of contractsData) {
```

**Passo 4: Reconstrói entity Contract** (linha 16)
```typescript
const contract = new Contract(
    contractData.id_contract, 
    contractData.description, 
    parseFloat(contractData.amount),  // Converte string para number
    contractData.periods, 
    contractData.date
);
```
- Cria nova instância de `Contract` (entity de domínio)
- `parseFloat()`: Banco retorna numeric como string, precisa converter
- **Importante**: Reconstrói estado completo da entity

**Passo 5: Busca pagamentos relacionados** (linha 17)
```typescript
const paymentsData = await this.connection.query(
    "select * from branas.payment where id_contract = $1", 
    [contract.idContract]
);
```
- Query com parâmetro: `$1` placeholder
- `[contract.idContract]`: Parâmetro substituído de forma segura
- **SQL Injection Protection**: Parâmetros parametrizados

**⚠️ Problema N+1:**
```typescript
for (const contractData of contractsData) {
    // ...
    const paymentsData = await this.connection.query(...);  // Query dentro de loop!
}
```
- Para cada contrato, faz uma query adicional
- Se 100 contratos: 1 query + 100 queries = 101 queries
- **Solução**: JOIN ou buscar todos payments de uma vez

**Otimização possível:**
```typescript
async list(): Promise<Contract[]> {
    const contractsData = await this.connection.query(`
        SELECT c.*, 
               json_agg(p.*) as payments
        FROM branas.contract c
        LEFT JOIN branas.payment p ON p.id_contract = c.id_contract
        GROUP BY c.id_contract
    `, []);
    // Processar resultado agregado...
}
```

**Passo 6: Reconstrói Payment value objects** (linha 18-20)
```typescript
for (const paymentData of paymentsData) {
    contract.addPayment(
        new Payment(
            paymentData.id_payment, 
            paymentData.date, 
            parseFloat(paymentData.amount)
        )
    );
}
```
- Cria instâncias de `Payment`
- Adiciona ao contrato via método `addPayment()`
- **Encapsulamento**: Usa API pública da entity

**Passo 7: Adiciona contrato completo ao resultado** (linha 21)
```typescript
contracts.push(contract);
```

**Import Problemático:**
```typescript
import AccrualBasisStrategy from "../../domain/AccrualBasisStrategy";
```
- **Não é usado** no código ❌
- Deveria ser removido
- Possível resíduo de refatoração

**Responsabilidades do Repository:**
1. **Tradução**: DB row → Domain entity
2. **Reconstrução**: Aggregate com suas partes (contract + payments)
3. **Persistência**: (Não implementado aqui: save, update, delete)

**Princípios Aplicados:**
✅ **SRP**: Única responsabilidade - persistência de contratos
✅ **DIP**: Depende de `DatabaseConnection` (abstração)
✅ **ISP**: Implementa apenas `list()`, não métodos desnecessários

---

## 3️⃣ Presenter Pattern - Implementações

### 📄 JsonPresenter.ts - Formatação JSON

```typescript
import { Output } from "../../application/usecase/GenerateInvoices";
import Presenter from "../../application/presenter/Presenter";

export default class JsonPresenter implements Presenter {

	present(output: Output[]): any {
		return output;
	}

}
```

#### 🔍 Explicação Técnica

**Presenter Pattern - Implementação JSON:**
- Implementa interface `Presenter`
- Formata saída para JSON

**Implementação Trivial:**
```typescript
present(output: Output[]): any {
    return output;
}
```
- Apenas retorna o array como está
- JavaScript/TypeScript arrays são automaticamente JSON-serializáveis
- Express faz `JSON.stringify()` automaticamente

**Quando Seria Mais Complexo:**
```typescript
present(output: Output[]): any {
    return {
        success: true,
        data: output,
        timestamp: new Date().toISOString(),
        count: output.length
    };
}
```

**Formatação de Datas:**
```typescript
present(output: Output[]): any {
    return output.map(item => ({
        date: item.date.toISOString(),  // Date → ISO string
        amount: item.amount
    }));
}
```

**Por que Presenter Separado?**
- Mesmo para JSON simples, separa formatação do use case
- Facilita mudanças futuras
- Mantém use case focado em lógica, não formatação

---

### 📄 CsvPresenter.ts - Formatação CSV

```typescript
import { Output } from "../../application/usecase/GenerateInvoices";
import Presenter from "../../application/presenter/Presenter";
import moment from "moment";

export default class CsvPresenter implements Presenter {

	present(output: Output[]): any {
		const lines: any[] = [];
		for (const data of output) {
			const line: string[] = [];
			line.push(moment(data.date).format("YYYY-MM-DD"));
			line.push(`${data.amount}`);
			lines.push(line.join(";"));
		}
		return lines.join("\n");
	}

}
```

#### 🔍 Explicação Técnica Detalhada

**Presenter Pattern - Implementação CSV:**
- Mesmo input, saída completamente diferente
- Demonstra valor do pattern

**Algoritmo de Formatação:**

**Passo 1: Inicializa array de linhas** (linha 8)
```typescript
const lines: any[] = [];
```
- Cada linha será uma string CSV

**Passo 2: Itera sobre outputs** (linha 9)
```typescript
for (const data of output) {
```

**Passo 3: Cria array de colunas** (linha 10)
```typescript
const line: string[] = [];
```

**Passo 4: Formata data** (linha 11)
```typescript
line.push(moment(data.date).format("YYYY-MM-DD"));
```
- Usa `moment.js` para formatar data
- Formato ISO: `YYYY-MM-DD` (ex: 2024-01-15)
- **Consistente** independente de timezone

**Passo 5: Converte amount para string** (linha 12)
```typescript
line.push(`${data.amount}`);
```
- Template literal converte número para string
- Poderia formatar com decimais: `amount.toFixed(2)`

**Passo 6: Junta colunas com ponto-e-vírgula** (linha 13)
```typescript
lines.push(line.join(";"));
```
- `;` como separador (padrão europeu)
- CSV americano usa `,`

**Passo 7: Junta linhas com quebra de linha** (linha 15)
```typescript
return lines.join("\n");
```
- `\n`: Nova linha Unix/Linux
- Windows usa `\r\n`

**Exemplo de Saída:**
```
2024-01-15;5000
2024-02-15;5000
2024-03-15;5000
```

**Melhorias Possíveis:**

**1. Cabeçalho CSV:**
```typescript
present(output: Output[]): any {
    const lines: string[] = ["date;amount"];  // Header
    for (const data of output) {
        const line = [
            moment(data.date).format("YYYY-MM-DD"),
            data.amount.toFixed(2)
        ];
        lines.push(line.join(";"));
    }
    return lines.join("\n");
}
```

**2. Separador Configurável:**
```typescript
export default class CsvPresenter implements Presenter {
    constructor(private separator: string = ";") {}
    
    present(output: Output[]): any {
        // ... use this.separator
    }
}
```

**3. Escapar Valores:**
```typescript
// Se valor contém separador, precisa de aspas
const escape = (value: string) => {
    if (value.includes(";") || value.includes("\n")) {
        return `"${value.replace(/"/g, '""')}"`;
    }
    return value;
};
```

**Uso de Biblioteca Externa:**
```typescript
import moment from "moment";
```
- Infraestrutura pode usar bibliotecas específicas
- Não viola Clean Architecture (está na camada certa)

---

## 4️⃣ Adapter Pattern - HTTP

### 📄 HttpServer.ts - Interface

```typescript
export default interface HttpServer {
	on (method: string, url: string, callback: Function): void;
	listen (port: number): void;
}
```

#### 🔍 Explicação Técnica

**Adapter Pattern - HTTP Interface:**
- Abstrai framework HTTP (Express, Fastify, Koa, etc.)
- Define contrato mínimo para servidor web

**Método `on()`:**
```typescript
on (method: string, url: string, callback: Function): void
```
- Registra rota HTTP
- `method`: "get", "post", "put", "delete", etc.
- `url`: Path da rota (ex: "/generate_invoices")
- `callback`: Função executada quando rota é chamada

**Método `listen()`:**
```typescript
listen (port: number): void
```
- Inicia servidor na porta especificada
- Não retorna Promise (poderia ser async)

**Por que Interface Genérica?**
- Diferentes frameworks têm APIs diferentes
- Interface comum permite trocar implementação
- Testes podem usar mock HTTP server

---

### 📄 ExpressAdapter.ts - Implementação Express

```typescript
import HttpServer from "./HttpServer";
import express from "express";

export default class ExpressAdapter implements HttpServer {
	app: any;

	constructor () {
		this.app = express();
		this.app.use(express.json());
	}

	on(method: string, url: string, callback: Function): void {
		this.app[method](url, async function (req: any, res: any) {
			const output = await callback(req.params, req.body, req.headers);
			res.json(output);
		});
	}

	listen(port: number): void {
		this.app.listen(port);
	}

}
```

#### 🔍 Explicação Técnica Detalhada

**Adapter Pattern - Express:**
- **Adaptee**: Express framework
- **Target**: HttpServer interface
- **Adapter**: ExpressAdapter

**Inicialização:**
```typescript
constructor () {
    this.app = express();
    this.app.use(express.json());
}
```

**1. Cria app Express:**
```typescript
this.app = express();
```

**2. Middleware de parsing JSON:**
```typescript
this.app.use(express.json());
```
- Parseia body JSON automaticamente
- Transforma string JSON → objeto JavaScript
- Disponível em `req.body`

**Método `on()` - Registro de Rotas:**
```typescript
on(method: string, url: string, callback: Function): void {
    this.app[method](url, async function (req: any, res: any) {
        const output = await callback(req.params, req.body, req.headers);
        res.json(output);
    });
}
```

**Análise Linha por Linha:**

**1. Bracket notation para método dinâmico:**
```typescript
this.app[method](url, ...)
```
- `method` pode ser "get", "post", etc.
- `this.app["post"]` é igual a `this.app.post`
- Permite método dinâmico baseado em string

**2. Handler assíncrono:**
```typescript
async function (req: any, res: any) {
```
- Express não requer async, mas útil para await
- `req`: Request object do Express
- `res`: Response object do Express

**3. Extrai dados da requisição:**
```typescript
const output = await callback(req.params, req.body, req.headers);
```
- `req.params`: URL parameters (ex: /users/:id)
- `req.body`: Body parseado (JSON)
- `req.headers`: HTTP headers
- **Abstração**: Callback não conhece Express, recebe dados simples

**4. Retorna JSON:**
```typescript
res.json(output);
```
- Express serializa objeto para JSON
- Define `Content-Type: application/json`
- Envia resposta HTTP

**Problema de Error Handling:**
```typescript
const output = await callback(req.params, req.body, req.headers);
```
- Se callback lançar erro, não é tratado
- Deveria ter try/catch

**Melhor Implementação:**
```typescript
on(method: string, url: string, callback: Function): void {
    this.app[method](url, async function (req: any, res: any) {
        try {
            const output = await callback(req.params, req.body, req.headers);
            res.json(output);
        } catch (error: any) {
            res.status(500).json({ error: error.message });
        }
    });
}
```

**Método `listen()`:**
```typescript
listen(port: number): void {
    this.app.listen(port);
}
```
- Delega diretamente para Express
- Inicia servidor HTTP

**Tipo `any` Problemático:**
```typescript
app: any;
```
- Perde type safety do TypeScript
- **Melhor**:
```typescript
import express, { Express } from "express";

export default class ExpressAdapter implements HttpServer {
    app: Express;
    // ...
}
```

---

## 5️⃣ Controller Pattern

### 📄 MainController.ts

```typescript
import HttpServer from "./HttpServer";
import Usecase from "../../application/usecase/Usecase";

export default class MainController {

	constructor (readonly httpServer: HttpServer, readonly usecase: Usecase) {
		httpServer.on("post", "/generate_invoices", async function (params: any, body: any, headers: any) {
			const input = body;
			body.userAgent = headers["user-agent"];
			body.host = headers.host;
			const output = await usecase.execute(input);
			return output;
		});
	}
}
```

#### 🔍 Explicação Técnica Detalhada

**Controller Pattern:**
- Conecta **driver externo** (HTTP) com **aplicação** (use case)
- Traduz dados HTTP para formato que use case entende
- Não contém lógica de negócio

**Dependency Injection:**
```typescript
constructor (readonly httpServer: HttpServer, readonly usecase: Usecase) {
```
- Recebe **abstrações**, não implementações concretas
- DIP aplicado
- Fácil testar com mocks

**Registro de Rota no Constructor:**
```typescript
constructor (...) {
    httpServer.on("post", "/generate_invoices", async function (...) {
        // ...
    });
}
```
- Rota registrada na criação do controller
- **Alternativa**: Método `setupRoutes()` separado
- **Trade-off**: Simples vs Flexível

**Handler da Rota:**

**Linha 7-8: Prepara input:**
```typescript
const input = body;
body.userAgent = headers["user-agent"];
body.host = headers.host;
```

**⚠️ Problema: Mutação do body:**
```typescript
const input = body;  // Referência, não cópia!
body.userAgent = ...;  // Modifica o mesmo objeto
```
- `input` e `body` apontam para mesmo objeto
- Modifica objeto original

**Melhor Implementação:**
```typescript
const input = {
    ...body,
    userAgent: headers["user-agent"],
    host: headers.host
};
```
- Cria novo objeto
- Spread operator copia propriedades
- Não modifica original

**Linha 10: Executa use case:**
```typescript
const output = await usecase.execute(input);
```
- Delega processamento para use case
- `await` porque use case é assíncrono
- Controller não sabe o que use case faz internamente

**Linha 11: Retorna output:**
```typescript
return output;
```
- Retorna resultado para HttpServer
- ExpressAdapter transformará em JSON

**Responsabilidades do Controller:**
1. **Receber requisição HTTP**
2. **Extrair dados necessários** (body, headers)
3. **Chamar use case**
4. **Retornar resposta**

**O que NÃO faz:**
- ❌ Lógica de negócio
- ❌ Validação complexa (deveria estar em use case ou domain)
- ❌ Formatação de saída (delegado para Presenter)

**Múltiplas Rotas:**
```typescript
export default class MainController {
    constructor (readonly httpServer: HttpServer, 
                 readonly generateInvoices: Usecase,
                 readonly getContract: Usecase) {
        
        httpServer.on("post", "/generate_invoices", async (params, body, headers) => {
            const input = { ...body, userAgent: headers["user-agent"] };
            return await generateInvoices.execute(input);
        });
        
        httpServer.on("get", "/contracts/:id", async (params, body, headers) => {
            return await getContract.execute({ id: params.id });
        });
    }
}
```

---

## 6️⃣ Mediator Pattern

### 📄 Mediator.ts - Publish/Subscribe

```typescript
export default class Mediator {
	observers: { event: string, callback: Function }[];

	constructor () {
		this.observers = [];
	}

	on (event: string, callback: Function) {
		this.observers.push({ event, callback });
	}

	async publish (event: string, data: any) {
		for (const observer of this.observers) {
			if (observer.event === event) {
				await observer.callback(data);
			}
		}
	}
}
```

#### 🔍 Explicação Técnica Detalhada

**Mediator Pattern (Publish/Subscribe):**
- **Desacopla** componentes que precisam comunicar
- Publishers não conhecem Subscribers
- Subscribers não conhecem Publishers
- **Central**: Mediator conhece ambos

**Estrutura de Dados:**
```typescript
observers: { event: string, callback: Function }[];
```
- Array de observers
- Cada observer:
  - `event`: Nome do evento que escuta
  - `callback`: Função executada quando evento ocorre

**Método `on()` - Registrar Observer:**
```typescript
on (event: string, callback: Function) {
    this.observers.push({ event, callback });
}
```
- Adiciona observer ao array
- **Não valida** duplicatas (poderia ter mesmo callback múltiplas vezes)
- Sintaxe curta: `{ event, callback }` = `{ event: event, callback: callback }`

**Uso:**
```typescript
mediator.on("InvoicesGenerated", async (data) => {
    await sendEmail.execute(data);
});
```

**Método `publish()` - Notificar Observers:**
```typescript
async publish (event: string, data: any) {
    for (const observer of this.observers) {
        if (observer.event === event) {
            await observer.callback(data);
        }
    }
}
```

**Análise:**

**1. Itera sobre todos observers:**
```typescript
for (const observer of this.observers) {
```

**2. Filtra por evento:**
```typescript
if (observer.event === event) {
```
- Apenas observers do evento específico são notificados

**3. Executa callback com await:**
```typescript
await observer.callback(data);
```
- **Sequencial**: Um observer por vez
- **Bloqueante**: Aguarda cada callback terminar
- **Vantagem**: Garante ordem de execução
- **Desvantagem**: Lento se muitos observers

**Uso:**
```typescript
await mediator.publish("InvoicesGenerated", invoicesData);
```

**Fluxo Completo:**
```typescript
// 1. Registro (startup)
mediator.on("InvoicesGenerated", async (data) => {
    await sendEmail.execute(data);
});

// 2. Publicação (durante execução)
await mediator.publish("InvoicesGenerated", output);
// → Callback é executado
// → Email é enviado
```

**Benefícios:**
1. **Desacoplamento**: GenerateInvoices não conhece SendEmail
2. **Extensibilidade**: Adicionar novos observers sem modificar publicador
3. **Single Responsibility**: Cada observer faz uma coisa

**Limitações desta Implementação:**

**1. Execução Sequencial:**
```typescript
await observer.callback(data);  // Um de cada vez
```
**Solução - Parallel:**
```typescript
async publish (event: string, data: any) {
    const callbacks = this.observers
        .filter(obs => obs.event === event)
        .map(obs => obs.callback(data));
    await Promise.all(callbacks);  // Paralelo
}
```

**2. Sem Error Handling:**
```typescript
await observer.callback(data);  // Se lançar erro, para tudo
```
**Solução:**
```typescript
async publish (event: string, data: any) {
    for (const observer of this.observers) {
        if (observer.event === event) {
            try {
                await observer.callback(data);
            } catch (error) {
                console.error(`Error in observer: ${error}`);
                // Continua executando outros observers
            }
        }
    }
}
```

**3. Sem Unsubscribe:**
```typescript
// Não há método para remover observers
```
**Solução:**
```typescript
off (event: string, callback: Function) {
    this.observers = this.observers.filter(
        obs => !(obs.event === event && obs.callback === callback)
    );
}
```

**Alternativas:**
- **EventEmitter** do Node.js
- **RxJS** Observables
- **Message Queue** (RabbitMQ, Kafka) para sistemas distribuídos

---

## 🎯 Resumo dos Patterns da Camada de Infraestrutura

| Pattern | Classe | Propósito |
|---------|--------|-----------|
| **Adapter** | `PgPromiseAdapter`, `ExpressAdapter` | Adapta bibliotecas externas |
| **Repository** | `ContractDatabaseRepository` | Implementa persistência |
| **Presenter** | `JsonPresenter`, `CsvPresenter` | Implementa formatação |
| **Controller** | `MainController` | Conecta HTTP com aplicação |
| **Mediator** | `Mediator` | Implementa publish/subscribe |

## 📊 Princípios SOLID na Camada de Infraestrutura

✅ **SRP**: Cada classe tem responsabilidade única e clara
✅ **OCP**: Novas implementações sem modificar existentes
✅ **LSP**: Implementações substituem interfaces corretamente
✅ **ISP**: Interfaces mínimas e específicas
✅ **DIP**: Camada depende de abstrações das camadas internas

## 🔗 Próximos Passos

Continue para `04-COMPOSITION-ROOT.md` para entender como todas as peças são conectadas na inicialização da aplicação.
