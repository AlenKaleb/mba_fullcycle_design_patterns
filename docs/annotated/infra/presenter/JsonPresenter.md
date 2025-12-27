# `src/infra/presenter/JsonPresenter.ts`

## Papel na arquitetura

Presenter que retorna a saída em formato **JSON puro** (array de objetos).

## Código original

```ts
import { Output } from "../../application/usecase/GenerateInvoices";
import Presenter from "../../application/presenter/Presenter";

export default class JsonPresenter implements Presenter {

	present(output: Output[]): any {
		return output;
	}

}
```

## Anotações técnicas

- **Implementa `Presenter`**: respeita o contrato de apresentação.
- **Sem transformação**: retorna o `Output` tal como produzido pelo caso de uso.

## Boas práticas

- **Simplicidade**: formato padrão ideal para APIs.
