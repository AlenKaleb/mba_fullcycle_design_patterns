# Exportação anotada do código

Este diretório contém uma versão **anotada** do código da branch atual. Cada arquivo `.md` representa um arquivo de origem e inclui:

- **Trecho original** em bloco de código.
- **Anotações técnicas** por responsabilidade e decisões de design.
- **Conexões com arquitetura** (camadas, limites e dependências).
- **Padrões de projeto** e **boas práticas** aplicadas.

## Arquitetura geral

O projeto segue uma **arquitetura em camadas** (Domain / Application / Infra) com um **entrypoint** (`src/main.ts`).

- **Domain**: regras de negócio puras e entidades (ex.: `Contract`, `Invoice`), além de estratégias para geração de faturas.
- **Application**: casos de uso (ex.: `GenerateInvoices`) e contratos (interfaces) para repositórios e presenters.
- **Infra**: adaptações técnicas (HTTP, DB, presenters, mediador), mantendo o domínio desacoplado.

Principais conceitos e padrões aplicados:

- **Strategy + Factory**: seleção do algoritmo de geração de faturas por tipo (`cash` vs `accrual`).
- **Repository**: abstração do acesso a dados (`ContractRepository`).
- **Decorator**: enriquecimento de casos de uso com logging (`LoggerDecorator`).
- **Mediator/Observer**: publicação de eventos de domínio para ações colaterais (ex.: envio de e-mail).
- **Adapter**: adaptação de frameworks externos (Express, PgPromise) para interfaces internas.
- **Presenter**: formatação de saída (JSON/CSV) sem acoplar o caso de uso a um formato específico.

## Índice de arquivos anotados

- `src/main.ts` → `main.md`
- `src/domain/Contract.ts` → `domain/Contract.md`
- `src/domain/Invoice.ts` → `domain/Invoice.md`
- `src/domain/Payment.ts` → `domain/Payment.md`
- `src/domain/InvoiceGenerationStrategy.ts` → `domain/InvoiceGenerationStrategy.md`
- `src/domain/InvoiceGenerationFactory.ts` → `domain/InvoiceGenerationFactory.md`
- `src/domain/CashBasisStrategy.ts` → `domain/CashBasisStrategy.md`
- `src/domain/AccrualBasisStrategy.ts` → `domain/AccrualBasisStrategy.md`
- `src/application/usecase/Usecase.ts` → `application/usecase/Usecase.md`
- `src/application/usecase/GenerateInvoices.ts` → `application/usecase/GenerateInvoices.md`
- `src/application/usecase/SendEmail.ts` → `application/usecase/SendEmail.md`
- `src/application/repository/ContractRepository.ts` → `application/repository/ContractRepository.md`
- `src/application/presenter/Presenter.ts` → `application/presenter/Presenter.md`
- `src/application/decorator/LoggerDecorator.ts` → `application/decorator/LoggerDecorator.md`
- `src/infra/mediator/Mediator.ts` → `infra/mediator/Mediator.md`
- `src/infra/repository/ContractDatabaseRepository.ts` → `infra/repository/ContractDatabaseRepository.md`
- `src/infra/presenter/JsonPresenter.ts` → `infra/presenter/JsonPresenter.md`
- `src/infra/presenter/CsvPresenter.ts` → `infra/presenter/CsvPresenter.md`
- `src/infra/database/DatabaseConnection.ts` → `infra/database/DatabaseConnection.md`
- `src/infra/database/PgPromiseAdapter.ts` → `infra/database/PgPromiseAdapter.md`
- `src/infra/http/HttpServer.ts` → `infra/http/HttpServer.md`
- `src/infra/http/ExpressAdapter.ts` → `infra/http/ExpressAdapter.md`
- `src/infra/http/MainController.ts` → `infra/http/MainController.md`
