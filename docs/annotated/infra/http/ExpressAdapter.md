# `src/infra/http/ExpressAdapter.ts`

## Papel na arquitetura

Adapter que implementa `HttpServer` usando **Express**.

## Código original

```ts
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

## Anotações técnicas

- **Inicialização**: habilita `express.json()` para parsear JSON.
- **Registro de rota genérico**: `this.app[method]` permite `get`, `post`, etc.
- **Callback unificado**: repassa `params`, `body`, `headers` para a camada de controller.
- **Resposta padrão JSON**: mantém consistência para APIs.

## Boas práticas

- **Adapter Pattern**: Express fica isolado da aplicação.
