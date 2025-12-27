# `src/infra/http/HttpServer.ts`

## Papel na arquitetura

Interface para servidores HTTP, permitindo diferentes frameworks.

## Código original

```ts
export default interface HttpServer {
	on (method: string, url: string, callback: Function): void;
	listen (port: number): void;
}
```

## Anotações técnicas

- **Contrato genérico**: define apenas o necessário para registrar rotas e iniciar servidor.
- **Desacoplamento**: controllers não dependem de Express diretamente.

## Boas práticas

- **Adapter Pattern**: facilita troca de framework HTTP.
