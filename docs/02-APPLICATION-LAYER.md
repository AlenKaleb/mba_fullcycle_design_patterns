# Camada de Aplicação (Application Layer)

## 📚 Visão Geral

A camada de aplicação **orquestra** o fluxo de dados entre a camada de domínio e a infraestrutura. Esta camada contém:
- **Use Cases**: Casos de uso da aplicação
- **Interfaces**: Contratos para Repository e Presenter
- **Decorators**: Adição de funcionalidades transversais

**Responsabilidade**: Implementar os casos de uso (user stories) sem conter regras de negócio ou detalhes de infraestrutura.

---

## 1️⃣ Use Case Pattern

### 📄 Usecase.ts - Interface Base

```typescript
export default interface Usecase {
	execute (input: any): Promise<any>;
}
```

#### 🔍 Explicação Técnica

**Use Case Pattern:**
- Representa uma **ação** que o usuário pode executar no sistema
- Cada use case implementa um **caso de uso específico**
- Interface genérica que todos os use cases implementam

**Assinatura do Método:**
```typescript
execute (input: any): Promise<any>
```
- **`execute`**: Nome padronizado para executar o caso de uso
- **`input: any`**: Entrada genérica (pode ser tipada em implementações)
- **`Promise<any>`**: Retorno assíncrono (operações podem envolver I/O)

**Por que Interface Genérica?**
- Permite que diferentes use cases tenham inputs/outputs diferentes
- Facilita decorators e middlewares que trabalham com qualquer use case
- **Trade-off**: Perde type safety, ganha flexibilidade

**Aplicação do ISP (Interface Segregation Principle):**
- Interface mínima com apenas um método
- Clientes não são forçados a implementar métodos desnecessários

---

### 📄 GenerateInvoices.ts - Use Case Principal

```typescript
import ContractDatabaseRepository from "../../infra/repository/ContractDatabaseRepository";
import ContractRepository from "../repository/ContractRepository";
import Presenter from "../presenter/Presenter";
import JsonPresenter from "../../infra/presenter/JsonPresenter";
import Usecase from "./Usecase";
import Mediator from "../../infra/mediator/Mediator";

export default class GenerateInvoices implements Usecase {

	constructor (
		readonly contractRepository: ContractRepository, 
		readonly presenter: Presenter = new JsonPresenter(),
		readonly mediator: Mediator = new Mediator()
	) {
	}

	async execute (input: Input): Promise<any> {
		const output: Output[] = [];
		const contracts = await this.contractRepository.list();
		for (const contract of contracts) {
			const invoices = contract.generateInvoices(input.month, input.year, input.type);
			for (const invoice of invoices) {
				output.push({ date: invoice.date, amount: invoice.amount });
			}
		}
		await this.mediator.publish("InvoicesGenerated", output);
		return this.presenter.present(output);
	}
}

type Input = {
	month: number,
	year: number,
	type: string,
	format?: string
}

export type Output = {
	date: Date,
	amount: number
}
```

#### 🔍 Explicação Técnica Detalhada

**Dependency Injection via Constructor:**
```typescript
constructor (
    readonly contractRepository: ContractRepository, 
    readonly presenter: Presenter = new JsonPresenter(),
    readonly mediator: Mediator = new Mediator()
) {
```

**Análise das Dependências:**

1. **`contractRepository: ContractRepository`** (obrigatória)
   - Tipo: **Interface** (DIP aplicado)
   - Injetada sem default
   - Permite diferentes implementações (DB, in-memory, mock)

2. **`presenter: Presenter = new JsonPresenter()`** (opcional com default)
   - Default: `JsonPresenter` para JSON
   - Pode ser substituído por `CsvPresenter` ou outros
   - **Flexibilidade**: Cliente pode passar outro presenter

3. **`mediator: Mediator = new Mediator()`** (opcional com default)
   - Default: Novo mediator
   - Permite injetar mediator compartilhado com observers
   - Facilita testes (pode injetar mock mediator)

**⚠️ Observação sobre Defaults:**
```typescript
presenter: Presenter = new JsonPresenter()
```
- Cria instância concreta dentro do módulo
- **Viola** pureza da injeção de dependência
- **Melhor**: Injetar todas dependências sem defaults no Composition Root
- **Trade-off**: Conveniência vs Controle total

**Fluxo de Execução do Use Case:**

**Passo 1: Inicializa output** (linha 18)
```typescript
const output: Output[] = [];
```
- Array para acumular resultados
- Tipo `Output[]` definido no final do arquivo (DTO)

**Passo 2: Busca contratos do repositório** (linha 19)
```typescript
const contracts = await this.contractRepository.list();
```
- Chama método `list()` do repositório (abstração)
- `await` porque operação de I/O é assíncrona
- Não sabe se vem de DB, arquivo, API ou memória

**Passo 3: Itera sobre contratos** (linha 20)
```typescript
for (const contract of contracts) {
```
- Processa cada contrato individualmente

**Passo 4: Gera invoices usando Strategy** (linha 21)
```typescript
const invoices = contract.generateInvoices(input.month, input.year, input.type);
```
- Delega para método de domínio `Contract.generateInvoices()`
- Passa parâmetros recebidos no input
- `type` determina qual Strategy será usada (cash/accrual)

**Passo 5: Converte para DTO** (linhas 22-24)
```typescript
for (const invoice of invoices) {
    output.push({ date: invoice.date, amount: invoice.amount });
}
```
- Cria objetos simples (DTOs) a partir das entities
- Não expõe objetos de domínio diretamente
- **Benefício**: Desacoplamento entre camadas

**Passo 6: Publica evento via Mediator** (linha 26)
```typescript
await this.mediator.publish("InvoicesGenerated", output);
```
- **Mediator Pattern** em ação
- Notifica observers sem acoplamento direto
- Permite ações side-effect (enviar email, logs, etc.)
- `await` garante que observers executem antes de retornar

**Passo 7: Formata saída com Presenter** (linha 27)
```typescript
return this.presenter.present(output);
```
- Delega formatação para Presenter
- Pode retornar JSON, CSV, XML, etc.
- Use case não conhece formato final

**DTO (Data Transfer Object):**

```typescript
type Input = {
	month: number,
	year: number,
	type: string,
	format?: string
}

export type Output = {
	date: Date,
	amount: number
}
```

**Input DTO:**
- Define contrato de entrada do use case
- `format?` é opcional (não usado atualmente, mas preparado para futuro)
- Vem tipicamente da camada HTTP

**Output DTO:**
- Estrutura simples, não é uma entity
- `export type` permite outros módulos importarem
- Usado por Presenters para formatar saída

**Princípios Aplicados:**

✅ **SRP**: Use case tem uma responsabilidade - gerar invoices
✅ **DIP**: Depende de abstrações (ContractRepository, Presenter)
✅ **OCP**: Aberto para novos presenters e repositories

---

### 📄 SendEmail.ts - Use Case de Notificação

```typescript
export default class SendEmail {

	constructor () {
	}

	async execute (input: any): Promise<void> {
		console.log(input);
	}
}
```

#### 🔍 Explicação Técnica

**Propósito:**
- Representa ação de enviar email com dados de invoices gerados
- Será chamado pelo **Mediator** quando evento "InvoicesGenerated" for publicado

**Implementação Simplificada:**
```typescript
console.log(input);
```
- Atualmente apenas loga no console
- Em produção, integraria com serviço de email (SendGrid, SES, etc.)
- **Stub**: Implementação placeholder para demonstração

**Por que não implementa `Usecase`?**
- Poderia implementar a interface `Usecase` para consistência
- Aqui está como classe simples
- **Observação**: Inconsistência arquitetural leve

**Como é Usado:**
```typescript
// Em main.ts
const sendEmail = new SendEmail();
mediator.on("InvoicesGenerated", async function (data: any) {
	await sendEmail.execute(data);
});
```
- Registrado como observer no mediator
- Executado automaticamente quando invoices são gerados

**Melhorias Futuras:**
1. Implementar `Usecase` interface
2. Injetar serviço de email concreto
3. Template de email formatado
4. Tratamento de erros

---

## 2️⃣ Repository Pattern

### 📄 ContractRepository.ts - Interface do Repositório

```typescript
import Contract from "../../domain/Contract";

export default interface ContractRepository {
	list (): Promise<Contract[]>;
}
```

#### 🔍 Explicação Técnica

**Repository Pattern:**
- Abstrai a **lógica de persistência** de dados
- Interface define **o que** fazer, não **como** fazer
- Camada de domínio pode usar sem conhecer implementação

**Por que Interface na Application Layer?**
- Application define o **contrato**
- Infrastructure implementa o contrato
- **DIP**: Abstração pertence ao cliente (application), não ao implementador

**Método `list()`:**
```typescript
list (): Promise<Contract[]>
```
- Retorna todos os contratos
- `Promise`: Operação assíncrona (geralmente banco de dados)
- Retorna **entities de domínio** (`Contract[]`), não DTOs

**Métodos Ausentes:**
- `findById()`, `save()`, `update()`, `delete()`
- Este sistema só precisa listar
- **Princípio**: Interface mínima suficiente (YAGNI - You Aren't Gonna Need It)

**Benefícios:**
1. **Testabilidade**: Fácil criar mock repository
2. **Flexibilidade**: Trocar implementação (DB, API, arquivo)
3. **Separação de Responsabilidades**: Domínio não conhece persistência

**Implementações Possíveis:**
- `ContractDatabaseRepository`: Banco de dados real
- `ContractMemoryRepository`: Array em memória (testes)
- `ContractAPIRepository`: Busca de API externa
- `ContractFileRepository`: Lê de arquivo JSON

---

## 3️⃣ Presenter Pattern

### 📄 Presenter.ts - Interface do Apresentador

```typescript
import { Output } from "../usecase/GenerateInvoices";

export default interface Presenter {
	present (output: Output[]): any;
}
```

#### 🔍 Explicação Técnica

**Presenter Pattern:**
- Responsável por **formatar dados** para apresentação
- Separa lógica de negócio da formatação de saída
- Permite múltiplos formatos sem modificar use case

**Assinatura:**
```typescript
present (output: Output[]): any
```
- **Entrada**: Array de Output DTOs
- **Saída**: `any` (pode ser objeto, string, stream, etc.)
- Cada implementação define formato específico

**Por que no Application?**
- Define **interface** usada pela aplicação
- Implementations ficam na Infrastructure
- **Princípio**: Interface pertence ao cliente

**Acoplamento com DTO:**
```typescript
import { Output } from "../usecase/GenerateInvoices";
```
- Presenter conhece estrutura do Output
- Acoplamento aceitável (ambos na mesma camada)
- Poderia usar generic para mais flexibilidade

---

## 4️⃣ Decorator Pattern

### 📄 LoggerDecorator.ts - Decorator de Logging

```typescript
import Usecase from "../usecase/Usecase";

export default class LoggerDecorator implements Usecase {

	constructor (readonly usecase: Usecase) {
	}

	execute(input: any): Promise<any> {
		console.log(input.userAgent);
		return this.usecase.execute(input);
	}

}
```

#### 🔍 Explicação Técnica Completa

**Decorator Pattern:**
- **Adiciona comportamento** a um objeto sem modificá-lo
- Envolve (wraps) o objeto original
- Implementa mesma interface do objeto decorado

**Estrutura:**

```typescript
export default class LoggerDecorator implements Usecase {
```
- Implementa `Usecase` (mesma interface do decorado)
- Pode ser usado onde qualquer `Usecase` é esperado
- **LSP**: Substituível por use case decorado

**Composição:**
```typescript
constructor (readonly usecase: Usecase) {
```
- Recebe use case a ser decorado
- Mantém referência para delegar chamadas
- **Composition over Inheritance**

**Comportamento Adicionado:**
```typescript
execute(input: any): Promise<any> {
	console.log(input.userAgent);  // NOVO comportamento
	return this.usecase.execute(input);  // Delega para decorado
}
```

**Passo a passo:**
1. **Loga userAgent** do input
2. **Delega** para use case original
3. **Retorna** resultado do use case original (sem modificar)

**Uso Prático:**
```typescript
// Sem decorator
const generateInvoices = new GenerateInvoices(contractRepository);

// Com decorator
const generateInvoices = new LoggerDecorator(
    new GenerateInvoices(contractRepository)
);
```

**Múltiplos Decorators (Chain):**
```typescript
const generateInvoices = 
    new LoggerDecorator(
        new TimerDecorator(
            new AuthDecorator(
                new GenerateInvoices(contractRepository)
            )
        )
    );
```
- Cada decorator adiciona funcionalidade
- Ordem importa (executam de fora para dentro)

**Aplicação do OCP:**
- **Fechado para modificação**: `GenerateInvoices` não é alterado
- **Aberto para extensão**: Novas funcionalidades via decorators
- Não precisa herança ou modificar código existente

**Limitações desta Implementação:**
1. **Log hardcoded**: Sempre loga `userAgent`
   - Poderia receber função de log injetada
2. **Console.log**: Não é produtivo
   - Deveria usar logger apropriado (Winston, Bunyan)
3. **Não intercepta erros**: 
   - Poderia ter try/catch para log de erros

**Melhorias Possíveis:**

**1. Logger Injetável:**
```typescript
constructor (
    readonly usecase: Usecase,
    readonly logger: Logger
) {}

execute(input: any): Promise<any> {
    this.logger.info("Executing use case", { userAgent: input.userAgent });
    return this.usecase.execute(input);
}
```

**2. Decorator Genérico:**
```typescript
export default class LoggerDecorator<T extends Usecase> implements Usecase {
    constructor (readonly usecase: T) {}
    
    async execute(input: any): Promise<any> {
        console.log("Before execution", input);
        const result = await this.usecase.execute(input);
        console.log("After execution", result);
        return result;
    }
}
```

**Outros Decorators Úteis:**
- `TimerDecorator`: Mede tempo de execução
- `CacheDecorator`: Cacheia resultados
- `RetryDecorator`: Retenta em caso de erro
- `ValidationDecorator`: Valida input
- `TransactionDecorator`: Envolve em transação DB

---

## 🎯 Resumo dos Patterns da Camada de Aplicação

| Pattern | Classe/Interface | Propósito |
|---------|------------------|-----------|
| **Use Case** | `Usecase`, `GenerateInvoices`, `SendEmail` | Orquestra fluxo de dados |
| **Repository** | `ContractRepository` | Abstrai persistência |
| **Presenter** | `Presenter` | Abstrai formatação de saída |
| **Decorator** | `LoggerDecorator` | Adiciona funcionalidades transversais |
| **DTO** | `Input`, `Output` | Transfere dados entre camadas |

## 📊 Princípios SOLID na Camada de Aplicação

✅ **SRP**: Cada use case tem responsabilidade única
✅ **OCP**: Decorators permitem extensão sem modificação
✅ **LSP**: Decorator substitui use case decorado
✅ **ISP**: Interfaces coesas e mínimas
✅ **DIP**: Use cases dependem de abstrações (Repository, Presenter)

## 🔄 Fluxo de Dados

```
[HTTP Request] 
    ↓
[Controller - infra layer]
    ↓
[LoggerDecorator] → Loga userAgent
    ↓
[GenerateInvoices] 
    ↓ (busca dados)
[ContractRepository interface]
    ↓ (implementação)
[ContractDatabaseRepository - infra layer]
    ↓ (retorna entities)
[GenerateInvoices]
    ↓ (chama domain)
[Contract.generateInvoices()] 
    ↓ (usa strategy)
[CashBasisStrategy/AccrualBasisStrategy]
    ↓ (retorna invoices)
[GenerateInvoices]
    ↓ (converte para DTO)
[Output[]]
    ↓ (publica evento)
[Mediator] → Notifica observers
    ↓ (formata)
[Presenter interface]
    ↓ (implementação)
[JsonPresenter - infra layer]
    ↓
[HTTP Response]
```

## 🔗 Próximos Passos

Continue para `03-INFRASTRUCTURE-LAYER.md` para entender as implementações concretas de adapters, mediator e controllers.
