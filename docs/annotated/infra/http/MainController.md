# `src/infra/http/MainController.ts`

## Papel na arquitetura

Controller que expõe o caso de uso `GenerateInvoices` via HTTP.

## Código original

```ts
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

## Anotações técnicas

- **Endpoint dedicado**: rota `POST /generate_invoices` expõe o caso de uso.
- **Enriquecimento de input**: adiciona `userAgent` e `host` no `body` para logging/auditoria.
- **Controller fino**: não contém regra de negócio, apenas mapeia HTTP → caso de uso.

## Boas práticas

- **Separação de camadas**: controller não conhece infraestrutura interna nem regras do domínio.
