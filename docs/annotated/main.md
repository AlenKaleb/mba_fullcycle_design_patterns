# `src/main.ts`

## Papel na arquitetura

Entrypoint que **orquestra a composição** das dependências e inicializa a aplicação. Aqui é aplicado o *composition root*, mantendo o domínio e a aplicação livres de detalhes de infraestrutura.

## Código original

```ts
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

## Anotações técnicas

- **Importações direcionadas para infra e application**: o entrypoint conhece implementações concretas (repositório, servidor HTTP, presenter, mediador), pois aqui ocorre a **montagem de dependências**.
- **`PgPromiseAdapter`** cria a conexão concreta com banco, mas é exposto ao repositório via a interface `DatabaseConnection` (princípio da **inversão de dependência**).
- **`ContractDatabaseRepository`** é instanciado com a conexão, respeitando o **Repository Pattern** para isolar persistência.
- **`Mediator`** recebe um *handler* para o evento `InvoicesGenerated`, ilustrando **Observer/Mediator** para efeitos colaterais (envio de e-mail) sem acoplar o caso de uso.
- **`LoggerDecorator`** envolve `GenerateInvoices`, aplicando o **Decorator Pattern** para adicionar logging sem alterar a classe original.
- **`MainController`** registra as rotas no `HttpServer` (aqui implementado pelo `ExpressAdapter`).
- **`httpServer.listen(3000)`** inicia o servidor, mantendo a lógica de transporte restrita à infraestrutura.

## Boas práticas evidenciadas

- *Composition root*: todas as dependências são montadas em um único ponto.
- *Separação de preocupações*: domínio e aplicação não dependem de Express ou de pg-promise.
- *Extensibilidade*: troca de presenter, banco ou transport layer sem alterar o domínio.
