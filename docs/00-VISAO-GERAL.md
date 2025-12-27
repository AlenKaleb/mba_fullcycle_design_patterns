# Visão Geral do Projeto - Design Patterns em TypeScript

## 📋 Descrição do Projeto

Este projeto demonstra a implementação de diversos **Design Patterns** em um sistema de geração de notas fiscais (invoices) baseado em contratos. O sistema utiliza TypeScript e segue os princípios **SOLID** e **Clean Architecture**.

## 🏗️ Arquitetura

O projeto está organizado em 3 camadas principais:

```
src/
├── domain/          # Camada de Domínio (Regras de Negócio)
├── application/     # Camada de Aplicação (Casos de Uso)
└── infra/          # Camada de Infraestrutura (Detalhes Técnicos)
```

### Separação de Responsabilidades por Camada

**Domain (Domínio)**
- Contém as regras de negócio puras
- Independente de frameworks e bibliotecas externas
- Entities, Value Objects, Interfaces de Estratégias

**Application (Aplicação)**
- Orquestra o fluxo de dados entre camadas
- Implementa casos de uso (Use Cases)
- Define interfaces/contratos (Repository, Presenter)

**Infrastructure (Infraestrutura)**
- Implementações concretas das interfaces
- Adaptadores para bibliotecas externas (Express, PostgreSQL)
- Controllers, Banco de dados, HTTP

## 🎯 Design Patterns Implementados

### 1. **Strategy Pattern** 
- **Onde:** `CashBasisStrategy`, `AccrualBasisStrategy`
- **Propósito:** Permite alternar entre diferentes algoritmos de geração de invoices

### 2. **Factory Pattern (Dynamic Factory)**
- **Onde:** `InvoiceGenerationFactory`
- **Propósito:** Cria instâncias de estratégias baseado em uma string

### 3. **Repository Pattern**
- **Onde:** `ContractRepository` (interface) e `ContractDatabaseRepository` (implementação)
- **Propósito:** Abstrai a lógica de persistência de dados

### 4. **Adapter Pattern**
- **Onde:** `PgPromiseAdapter`, `ExpressAdapter`
- **Propósito:** Adapta interfaces de bibliotecas externas para interfaces da aplicação

### 5. **Presenter Pattern**
- **Onde:** `JsonPresenter`, `CsvPresenter`
- **Propósito:** Formata dados de saída de acordo com necessidades do cliente

### 6. **Decorator Pattern**
- **Onde:** `LoggerDecorator`
- **Propósito:** Adiciona funcionalidades (logging) sem modificar o código original

### 7. **Controller Pattern**
- **Onde:** `MainController`
- **Propósito:** Conecta a camada HTTP com a aplicação

### 8. **Mediator Pattern**
- **Onde:** `Mediator`
- **Propósito:** Implementa publish/subscribe para reduzir acoplamento entre componentes

### 9. **Composition Root**
- **Onde:** `main.ts`
- **Propósito:** Ponto único onde todas as dependências são criadas e conectadas

### 10. **DTO (Data Transfer Object)**
- **Onde:** Types `Input` e `Output` em `GenerateInvoices`
- **Propósito:** Transferência de dados entre camadas

## 🔧 Princípios SOLID Aplicados

### **SRP - Single Responsibility Principle**
Cada classe tem uma única responsabilidade:
- `Contract` gerencia contratos
- `Invoice` representa uma nota fiscal
- `GenerateInvoices` executa o caso de uso de geração

### **OCP - Open/Closed Principle**
Aberto para extensão, fechado para modificação:
- Novas estratégias podem ser adicionadas sem modificar código existente
- Novos presenters podem ser criados implementando a interface `Presenter`

### **LSP - Liskov Substitution Principle**
Implementações podem substituir suas interfaces:
- Qualquer `InvoiceGenerationStrategy` pode ser usada no lugar de outra
- `JsonPresenter` e `CsvPresenter` são intercambiáveis

### **ISP - Interface Segregation Principle**
Interfaces específicas e coesas:
- `HttpServer` define apenas métodos HTTP
- `DatabaseConnection` define apenas operações de banco
- `Presenter` define apenas formatação de saída

### **DIP - Dependency Inversion Principle**
Dependência de abstrações, não de implementações:
- `GenerateInvoices` depende de `ContractRepository` (interface)
- `MainController` depende de `HttpServer` (interface)
- Implementações concretas são injetadas no Composition Root

## 📊 Fluxo de Execução

```
1. HTTP Request → ExpressAdapter (Adapter)
2. MainController (Controller) → recebe request
3. LoggerDecorator (Decorator) → adiciona logging
4. GenerateInvoices (Use Case) → orquestra lógica
5. ContractDatabaseRepository (Repository) → busca dados
6. Contract.generateInvoices() → usa Strategy Pattern
7. InvoiceGenerationFactory (Factory) → cria estratégia
8. CashBasisStrategy/AccrualBasisStrategy → gera invoices
9. Mediator (Mediator) → publica evento "InvoicesGenerated"
10. SendEmail → recebe notificação do evento
11. Presenter (Presenter) → formata resposta
12. HTTP Response ← retorna para cliente
```

## 📚 Tecnologias Utilizadas

- **TypeScript**: Linguagem principal
- **Express**: Framework HTTP
- **pg-promise**: Cliente PostgreSQL
- **Moment.js**: Manipulação de datas
- **Jest**: Framework de testes

## 📖 Documentação Detalhada

Os arquivos markdown a seguir contêm explicações técnicas detalhadas de cada componente:

1. `01-DOMAIN-LAYER.md` - Camada de Domínio (Strategy, Factory, Entities)
2. `02-APPLICATION-LAYER.md` - Camada de Aplicação (Repository, Presenter, Use Cases, Decorator)
3. `03-INFRASTRUCTURE-LAYER.md` - Camada de Infraestrutura (Adapters, Mediator, Controller)
4. `04-COMPOSITION-ROOT.md` - Injeção de Dependências e inicialização

## 🎓 Referências

### Livros Mencionados no Projeto
- **GoF (Gang of Four)** - Design Patterns: Elements of Reusable Object-Oriented Software
- **Head First Design Patterns** - Eric Freeman, Elisabeth Robson
- **Patterns of Enterprise Application Architecture** - Martin Fowler

### Conceitos Fundamentais
- **Clean Architecture** - Robert C. Martin
- **Domain-Driven Design** - Eric Evans
- **SOLID Principles** - Robert C. Martin
