# 🎓 MBA Full Cycle - Design Patterns em TypeScript

## 📋 Descrição

Projeto educacional demonstrando a implementação de **Design Patterns**, **Princípios SOLID** e **Clean Architecture** em TypeScript.

O sistema implementa um gerador de notas fiscais (invoices) baseado em contratos, com suporte para diferentes regimes contábeis.

---

## 🎯 Objetivo do Projeto

Demonstrar de forma **prática e didática**:
- ✅ **10+ Design Patterns** em ação
- ✅ **Princípios SOLID** aplicados
- ✅ **Clean Architecture** em 3 camadas
- ✅ **Dependency Injection** manual
- ✅ **Separation of Concerns**

---

## 📚 Documentação Completa

### 🔥 **[ACESSE A DOCUMENTAÇÃO DETALHADA](./docs/README.md)** 🔥

A pasta `docs/` contém **documentação técnica completa** com:

- **80+ KB** de explicações técnicas
- **Código comentado** linha por linha
- **Diagramas** de arquitetura e fluxo
- **Exemplos práticos** de cada pattern
- **Trade-offs** e decisões de design
- **Exercícios sugeridos** para prática

### 📖 Arquivos de Documentação:

1. **[00-VISAO-GERAL.md](./docs/00-VISAO-GERAL.md)** - Introdução e overview do projeto
2. **[01-DOMAIN-LAYER.md](./docs/01-DOMAIN-LAYER.md)** - Strategy, Factory, Entities
3. **[02-APPLICATION-LAYER.md](./docs/02-APPLICATION-LAYER.md)** - Use Cases, Repository, Presenter, Decorator
4. **[03-INFRASTRUCTURE-LAYER.md](./docs/03-INFRASTRUCTURE-LAYER.md)** - Adapters, Mediator, Controller
5. **[04-COMPOSITION-ROOT.md](./docs/04-COMPOSITION-ROOT.md)** - Dependency Injection

**Tempo total de leitura:** ~2 horas

---

## 🚀 Tecnologias Utilizadas

- **TypeScript** - Linguagem principal
- **Node.js** - Runtime JavaScript
- **Express** - Framework HTTP
- **PostgreSQL** - Banco de dados
- **pg-promise** - Cliente PostgreSQL
- **Moment.js** - Manipulação de datas
- **Jest** - Framework de testes

---

## 🏗️ Arquitetura

```
src/
├── domain/          # Camada de Domínio (Regras de Negócio)
│   ├── Contract.ts
│   ├── Invoice.ts
│   ├── Payment.ts
│   ├── InvoiceGenerationStrategy.ts
│   ├── CashBasisStrategy.ts
│   ├── AccrualBasisStrategy.ts
│   └── InvoiceGenerationFactory.ts
│
├── application/     # Camada de Aplicação (Casos de Uso)
│   ├── usecase/
│   │   ├── GenerateInvoices.ts
│   │   └── SendEmail.ts
│   ├── repository/
│   │   └── ContractRepository.ts
│   ├── presenter/
│   │   └── Presenter.ts
│   └── decorator/
│       └── LoggerDecorator.ts
│
└── infra/          # Camada de Infraestrutura (Detalhes Técnicos)
    ├── database/
    │   ├── DatabaseConnection.ts
    │   └── PgPromiseAdapter.ts
    ├── http/
    │   ├── HttpServer.ts
    │   ├── ExpressAdapter.ts
    │   └── MainController.ts
    ├── repository/
    │   └── ContractDatabaseRepository.ts
    ├── presenter/
    │   ├── JsonPresenter.ts
    │   └── CsvPresenter.ts
    └── mediator/
        └── Mediator.ts
```

---

## 🎨 Design Patterns Implementados

### Padrões Criacionais
- ✅ **Factory Pattern** - `InvoiceGenerationFactory`

### Padrões Estruturais
- ✅ **Adapter Pattern** - `PgPromiseAdapter`, `ExpressAdapter`
- ✅ **Decorator Pattern** - `LoggerDecorator`

### Padrões Comportamentais
- ✅ **Strategy Pattern** - `CashBasisStrategy`, `AccrualBasisStrategy`
- ✅ **Mediator Pattern** - `Mediator`

### Padrões Arquiteturais
- ✅ **Repository Pattern** - `ContractRepository`
- ✅ **Presenter Pattern** - `JsonPresenter`, `CsvPresenter`
- ✅ **Controller Pattern** - `MainController`
- ✅ **Use Case Pattern** - `GenerateInvoices`
- ✅ **DTO Pattern** - `Input`, `Output`
- ✅ **Composition Root** - `main.ts`

---

## 🔧 Princípios SOLID

- ✅ **S**ingle Responsibility Principle
- ✅ **O**pen/Closed Principle
- ✅ **L**iskov Substitution Principle
- ✅ **I**nterface Segregation Principle
- ✅ **D**ependency Inversion Principle

Cada princípio é explicado em detalhes na documentação.

---

## 🛠️ Instalação e Execução

### Pré-requisitos

- Node.js 14+
- PostgreSQL 11+
- Yarn ou NPM

### Instalação

```bash
# Instalar dependências
yarn install
# ou
npm install
```

### Configuração do Banco de Dados

```bash
# Execute o script SQL
psql -U postgres -d app < create.sql
```

### Executar Aplicação

```bash
# Desenvolvimento
yarn start
# ou
npm start

# A aplicação estará disponível em http://localhost:3000
```

### Executar Testes

```bash
# Rodar todos os testes
yarn test
# ou
npm test
```

---

## 📡 API

### POST /generate_invoices

Gera invoices baseado em contratos no banco de dados.

**Request Body:**
```json
{
  "month": 1,
  "year": 2022,
  "type": "accrual"
}
```

**Parâmetros:**
- `month`: Mês (1-12)
- `year`: Ano (ex: 2022)
- `type`: Tipo de regime (`"cash"` ou `"accrual"`)

**Response (JSON):**
```json
[
  {
    "date": "2022-01-01T00:00:00.000Z",
    "amount": 500
  },
  {
    "date": "2022-01-15T00:00:00.000Z",
    "amount": 1000
  }
]
```

---

## 🧪 Testes

O projeto contém testes unitários e de integração:

- `test/Contract.test.ts` - Testes de domínio
- `test/GenerateInvoices.test.ts` - Testes de use case
- `test/api.test.ts` - Testes de API HTTP

---

## 📚 Referências Bibliográficas

### Livros Utilizados como Base:

1. **Design Patterns: Elements of Reusable Object-Oriented Software** (GoF)
   - Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides

2. **Head First Design Patterns**
   - Eric Freeman, Elisabeth Robson

3. **Patterns of Enterprise Application Architecture**
   - Martin Fowler

4. **Clean Architecture**
   - Robert C. Martin

---

## 👨‍🎓 Para Estudantes

Este projeto é ideal para:
- Estudar Design Patterns na prática
- Aprender Clean Architecture
- Entender SOLID principles
- Preparar para entrevistas técnicas
- Referência para projetos acadêmicos

---

## 🎓 Sobre o MBA Full Cycle

Este projeto faz parte do conteúdo educacional do **MBA Full Cycle Development**, que aborda:
- Arquitetura de Software
- Design Patterns
- Clean Code
- Domain-Driven Design
- Microsserviços
- DevOps

---

## 📝 Licença

MIT License - Livre para uso educacional e comercial.

---

## 🤝 Contribuições

Contribuições são bem-vindas! Sinta-se à vontade para:
- Reportar bugs
- Sugerir melhorias
- Adicionar novos patterns
- Melhorar documentação

---

## ✨ Destaques

- 📚 **Documentação excepcional** com 80+ KB de explicações
- 🎯 **Código limpo** seguindo boas práticas
- 🏗️ **Arquitetura sólida** em 3 camadas
- 🧪 **Testes** unitários e integração
- 🎨 **10+ Design Patterns** implementados
- 🔧 **SOLID** principles aplicados

---

## 📧 Contato

Para dúvidas sobre o projeto ou documentação, consulte os arquivos na pasta `docs/`.

---

**Bons estudos! 🚀📚**

*Desenvolvido com foco em educação e qualidade.*
