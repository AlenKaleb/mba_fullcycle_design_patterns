# `src/infra/mediator/Mediator.ts`

## Papel na arquitetura

Implementação simples do **Mediator/Observer**, permitindo publicar eventos para múltiplos interessados.

## Código original

```ts
export default class Mediator {
	observers: { event: string, callback: Function }[];

	constructor () {
		this.observers = [];
	}

	on (event: string, callback: Function) {
		this.observers.push({ event, callback });
	}

	async publish (event: string, data: any) {
		for (const observer of this.observers) {
			if (observer.event === event) {
				await observer.callback(data);
			}
		}
	}
}
```

## Anotações técnicas

- **Registro de observadores**: `on` registra callbacks para eventos.
- **Publicação assíncrona**: `publish` aguarda cada callback, garantindo ordem de execução.
- **Baixo acoplamento**: o publicador não conhece quem escuta.

## Boas práticas

- **Extensibilidade**: novos listeners podem ser adicionados sem modificar o publicador.
