# `src/application/decorator/LoggerDecorator.ts`

## Papel na arquitetura

Decorator aplicado a casos de uso para **adicionar logging** sem alterar a implementação principal.

## Código original

```ts
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

## Anotações técnicas

- **Decorator Pattern**: encapsula um `Usecase` e intercepta a chamada.
- **Responsabilidade única**: adiciona logging de `userAgent` e delega o restante.
- **Transparência**: mantém a mesma interface (`Usecase`).

## Boas práticas

- **Não-invasivo**: permite adicionar comportamentos transversais (cross-cutting concerns) sem mudar o caso de uso.
