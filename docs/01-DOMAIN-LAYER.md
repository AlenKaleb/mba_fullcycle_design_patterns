# Camada de Domínio (Domain Layer)

## 📚 Visão Geral

A camada de domínio contém as **regras de negócio puras** da aplicação. Esta camada é completamente independente de frameworks, bibliotecas externas e detalhes de infraestrutura. Aqui implementamos entities, value objects e estratégias de negócio.

---

## 1️⃣ Entities (Entidades)

### 📄 Contract.ts - Entidade de Contrato

```typescript
import Invoice from "./Invoice";
import InvoiceGenerationFactory from "./InvoiceGenerationFactory";
import InvoiceGenerationStrategy from "./InvoiceGenerationStrategy";
import Payment from "./Payment";
import moment from "moment";

export default class Contract {
	private payments: Payment[];

	constructor (
		readonly idContract: string,
		readonly description: string,
		readonly amount: number,
		readonly periods: number,
		readonly date: Date
	) {
		this.payments = [];
	}

	addPayment (payment: Payment) {
		this.payments.push(payment);
	}

	getPayments () {
		return this.payments;
	}

	getBalance () {
		let balance = this.amount;
		for (const payment of this.payments) {
			balance -= payment.amount;
		}
		return balance;
	}

	generateInvoices (month: number, year: number, type: string) {
		const invoiceGenerationStrategy = InvoiceGenerationFactory.create(type);
		return invoiceGenerationStrategy.generate(this, month, year);
	}
}
```

#### 🔍 Explicação Técnica

**Conceito de Entity:**
- Uma **Entity** é um objeto que possui **identidade única** (`idContract`)
- Contém **regras de negócio** e **comportamentos** relacionados ao domínio
- Diferente de um simples objeto de dados (DTO), possui lógica de negócio

**Responsabilidades da Classe:**
1. **Gerenciar estado do contrato**: Armazena informações do contrato (valor, períodos, data)
2. **Agregar pagamentos**: Mantém lista de pagamentos associados
3. **Calcular saldo**: Implementa lógica de cálculo de saldo (valor original - pagamentos)
4. **Gerar invoices**: Delega a geração para estratégias específicas

**Design Patterns Utilizados:**

**1. Strategy Pattern (linha 36-39)**
```typescript
generateInvoices (month: number, year: number, type: string) {
    const invoiceGenerationStrategy = InvoiceGenerationFactory.create(type);
    return invoiceGenerationStrategy.generate(this, month, year);
}
```
- Delega o algoritmo de geração de invoices para uma **estratégia**
- Permite trocar o comportamento em tempo de execução
- **Benefício**: Adicionar novos tipos de geração sem modificar `Contract` (OCP)

**2. Factory Pattern (linha 37)**
```typescript
const invoiceGenerationStrategy = InvoiceGenerationFactory.create(type);
```
- Usa uma **Factory** para criar a estratégia apropriada
- Desacopla a criação da instância do seu uso
- **Benefício**: `Contract` não precisa conhecer classes concretas de estratégias

**Encapsulamento:**
```typescript
private payments: Payment[];
```
- Array `payments` é **privado**, só pode ser modificado via métodos públicos
- Garante que pagamentos sejam adicionados de forma controlada (`addPayment`)
- **Princípio**: Information Hiding - protege invariantes do domínio

**Imutabilidade Parcial:**
```typescript
readonly idContract: string,
readonly description: string,
readonly amount: number,
readonly periods: number,
readonly date: Date
```
- Propriedades marcadas como `readonly` não podem ser alteradas após construção
- **Benefício**: Previne mudanças acidentais em dados críticos do contrato

---

### 📄 Invoice.ts - Value Object de Nota Fiscal

```typescript
export default class Invoice {

	constructor (readonly date: Date, readonly amount: number) {
	}
}
```

#### 🔍 Explicação Técnica

**Conceito de Value Object:**
- Um **Value Object** é definido por seus **atributos**, não por identidade
- Dois invoices com mesma data e valor são considerados iguais
- Tipicamente **imutáveis** (todas propriedades são `readonly`)

**Características:**
1. **Sem identidade própria**: Não possui ID único
2. **Imutável**: Propriedades `readonly` não podem ser alteradas
3. **Substitutable**: Pode ser substituído por outro com mesmos valores
4. **Sem comportamento complexo**: Apenas armazena dados

**Por que não é uma Entity?**
- Invoice não precisa de identidade única no contexto deste sistema
- É simplesmente um registro de data e valor gerado a partir de um contrato
- Se fossem persistidos e rastreados individualmente, poderiam ser Entities

---

### 📄 Payment.ts - Value Object de Pagamento

```typescript
export default class Payment {

	constructor (
		readonly idPayment: string,
		readonly date: Date,
		readonly amount: number
	) {
	}
}
```

#### 🔍 Explicação Técnica

**Debate: Entity vs Value Object:**
- Possui `idPayment`, sugerindo identidade única
- Mas não possui comportamentos, apenas dados
- **Classificação**: Está entre Entity e Value Object (anemic model neste contexto)

**Na prática:**
- Se precisa rastrear pagamentos individualmente → **Entity**
- Se é apenas um registro imutável → **Value Object**
- Neste projeto, é tratado mais como **Value Object** com identificador

**Imutabilidade:**
- Todas propriedades `readonly`
- Uma vez criado, não pode ser modificado
- **Benefício**: Thread-safe, previsível, sem efeitos colaterais

---

## 2️⃣ Strategy Pattern - Algoritmos Intercambiáveis

### 📄 InvoiceGenerationStrategy.ts - Interface da Estratégia

```typescript
import Contract from "./Contract";
import Invoice from "./Invoice";

export default interface InvoiceGenerationStrategy {
	generate (contract: Contract, month: number, year: number): Invoice[];
}
```

#### 🔍 Explicação Técnica

**Strategy Pattern - Interface:**
- Define o **contrato** que todas as estratégias devem seguir
- Permite que diferentes algoritmos sejam **intercambiáveis**
- O cliente (`Contract`) depende da **abstração**, não de implementações concretas

**Assinatura do Método:**
```typescript
generate (contract: Contract, month: number, year: number): Invoice[]
```
- **Entrada**: Contrato e período (mês/ano)
- **Saída**: Array de Invoices gerados
- Diferentes estratégias implementam diferentes lógicas de geração

**Aplicação do DIP (Dependency Inversion Principle):**
- `Contract` depende de `InvoiceGenerationStrategy` (abstração)
- Estratégias concretas dependem de `InvoiceGenerationStrategy` (abstração)
- Ambos dependem da abstração, não um do outro diretamente

---

### 📄 CashBasisStrategy.ts - Regime de Caixa

```typescript
import Contract from "./Contract";
import Invoice from "./Invoice";
import InvoiceGenerationStrategy from "./InvoiceGenerationStrategy";

export default class CashBasisStrategy implements InvoiceGenerationStrategy {

	generate(contract: Contract, month: number, year: number): Invoice[] {
		const invoices: Invoice[] = [];
		for (const payment of contract.getPayments()) {
			if (payment.date.getMonth() + 1 !== month || payment.date.getFullYear() !== year) continue;
			invoices.push(new Invoice(payment.date, payment.amount));
		}
		return invoices;
	}

}
```

#### 🔍 Explicação Técnica

**Regime de Caixa (Cash Basis):**
- Reconhece receita **quando o pagamento é recebido**
- Não importa quando o serviço foi prestado, mas quando foi pago
- Comum em pequenas empresas e profissionais autônomos

**Algoritmo Implementado:**

1. **Inicializa array vazio** (linha 8):
```typescript
const invoices: Invoice[] = [];
```

2. **Itera sobre pagamentos do contrato** (linha 9):
```typescript
for (const payment of contract.getPayments()) {
```

3. **Filtra por mês e ano** (linha 10):
```typescript
if (payment.date.getMonth() + 1 !== month || payment.date.getFullYear() !== year) continue;
```
- `getMonth()` retorna 0-11, por isso soma 1
- Se não for o mês/ano desejado, pula para próximo

4. **Cria Invoice com data e valor do pagamento** (linha 11):
```typescript
invoices.push(new Invoice(payment.date, payment.amount));
```

**Strategy Pattern em Ação:**
- Implementa `InvoiceGenerationStrategy`
- Pode ser substituída por qualquer outra estratégia
- `Contract` não conhece detalhes desta implementação

---

### 📄 AccrualBasisStrategy.ts - Regime de Competência

```typescript
import Contract from "./Contract";
import Invoice from "./Invoice";
import InvoiceGenerationStrategy from "./InvoiceGenerationStrategy";
import moment from "moment";

export default class AccrualBasisStrategy implements InvoiceGenerationStrategy {

	generate(contract: Contract, month: number, year: number): Invoice[] {
		const invoices: Invoice[] = [];
		let period = 0;
		while (period <= contract.periods) {
			const date = moment(contract.date).add(period++, "months").toDate();
			if (date.getMonth() + 1 !== month || date.getFullYear() !== year) continue;
			const amount = contract.amount/contract.periods;
			invoices.push(new Invoice(date, amount));
		}
		return invoices;
	}

}
```

#### 🔍 Explicação Técnica

**Regime de Competência (Accrual Basis):**
- Reconhece receita **quando o serviço é prestado**, independente do pagamento
- Distribui o valor do contrato uniformemente pelos períodos
- Usado por empresas que seguem princípios contábeis formais

**Algoritmo Implementado:**

1. **Inicializa contador de período** (linha 10):
```typescript
let period = 0;
```

2. **Itera por todos os períodos do contrato** (linha 11):
```typescript
while (period <= contract.periods) {
```

3. **Calcula data de cada período** (linha 12):
```typescript
const date = moment(contract.date).add(period++, "months").toDate();
```
- Usa `moment.js` para adicionar meses à data inicial
- `period++` incrementa após usar o valor

4. **Filtra por mês e ano desejado** (linha 13):
```typescript
if (date.getMonth() + 1 !== month || date.getFullYear() !== year) continue;
```

5. **Calcula valor proporcional** (linha 14):
```typescript
const amount = contract.amount/contract.periods;
```
- Divide valor total pelos períodos
- Cada invoice tem valor igual

6. **Cria Invoice com data calculada e valor proporcional** (linha 15):
```typescript
invoices.push(new Invoice(date, amount));
```

**Diferença para CashBasisStrategy:**
- **Cash**: Usa datas e valores dos **pagamentos reais**
- **Accrual**: Calcula datas e valores **uniformemente distribuídos**

**Uso de Biblioteca Externa:**
```typescript
import moment from "moment";
```
- Poderia ser considerado violação de "domínio puro"
- Em Clean Architecture estrita, usaria abstrações de data
- **Trade-off**: Simplicidade vs Pureza arquitetural

---

## 3️⃣ Factory Pattern - Criação Dinâmica

### 📄 InvoiceGenerationFactory.ts - Dynamic Factory

```typescript
import AccrualBasisStrategy from "./AccrualBasisStrategy";
import CashBasisStrategy from "./CashBasisStrategy";

export default class InvoiceGenerationFactory {

	static create (type: string) {
		if (type === "cash") {
			return new CashBasisStrategy();
		}
		if (type === "accrual") {
			return new AccrualBasisStrategy();
		}
		throw new Error("Invalid type");
	}
}
```

#### 🔍 Explicação Técnica

**Factory Pattern - Dynamic Factory:**
- **Factory Method**: Método que cria objetos
- **Dynamic**: Decisão de criação baseada em parâmetro runtime (string)
- **Benefício**: Centraliza lógica de criação, facilita manutenção

**Método Estático:**
```typescript
static create (type: string)
```
- Não precisa instanciar a factory
- Chamado diretamente: `InvoiceGenerationFactory.create("cash")`
- **Trade-off**: Menos flexível que factory instanciável, mais simples

**Mapeamento String → Classe:**

```typescript
if (type === "cash") {
    return new CashBasisStrategy();
}
if (type === "accrual") {
    return new AccrualBasisStrategy();
}
```
- Converte string em instância concreta
- Permite que camadas externas passem tipo como string (ex: HTTP body)

**Tratamento de Erro:**
```typescript
throw new Error("Invalid type");
```
- Falha rápido se tipo inválido
- Poderia retornar default ou usar null object pattern

**Alternativas de Implementação:**

1. **Switch Statement:**
```typescript
switch (type) {
    case "cash": return new CashBasisStrategy();
    case "accrual": return new AccrualBasisStrategy();
    default: throw new Error("Invalid type");
}
```

2. **Map/Dictionary:**
```typescript
const strategies = {
    cash: CashBasisStrategy,
    accrual: AccrualBasisStrategy
};
const StrategyClass = strategies[type];
if (!StrategyClass) throw new Error("Invalid type");
return new StrategyClass();
```

**Princípio OCP (Open/Closed):**
- ❌ **Problema**: Para adicionar nova estratégia, precisa modificar factory
- ✅ **Solução avançada**: Registry pattern ou plugin system
- **Trade-off**: Simplicidade atual vs Extensibilidade futura

**Onde é Usado:**
```typescript
// Em Contract.ts
const invoiceGenerationStrategy = InvoiceGenerationFactory.create(type);
```
- `Contract` não instancia estratégias diretamente
- Delega para Factory, mantendo baixo acoplamento

---

## 🎯 Resumo dos Patterns da Camada de Domínio

| Pattern | Classe | Propósito |
|---------|--------|-----------|
| **Entity** | `Contract` | Objeto com identidade e comportamento de negócio |
| **Value Object** | `Invoice`, `Payment` | Objetos imutáveis definidos por seus valores |
| **Strategy** | `InvoiceGenerationStrategy` | Interface para algoritmos intercambiáveis |
| **Concrete Strategy** | `CashBasisStrategy`, `AccrualBasisStrategy` | Implementações específicas de algoritmos |
| **Factory** | `InvoiceGenerationFactory` | Criação dinâmica de estratégias |

## 📊 Princípios SOLID na Camada de Domínio

✅ **SRP**: Cada classe tem uma responsabilidade clara
✅ **OCP**: Novas estratégias podem ser adicionadas (parcialmente)
✅ **LSP**: Estratégias são substituíveis entre si
✅ **ISP**: Interface `InvoiceGenerationStrategy` é coesa
✅ **DIP**: `Contract` depende de abstração, não de implementações

## 🔗 Próximos Passos

Continue para `02-APPLICATION-LAYER.md` para entender como a camada de aplicação orquestra esses componentes de domínio.
