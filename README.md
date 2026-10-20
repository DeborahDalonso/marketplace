# marketplace

Repositório guarda-chuva de um marketplace construído como ecossistema de
microsserviços. Aqui ficam o guia de padrões, o docker-compose que sobe tudo
e o diagrama geral. Cada serviço vive no próprio repositório.

## Serviços

| Repositório | Linguagem | Papel |
|---|---|---|
| marketplace-customer | React + TypeScript | vitrine do comprador e landing |
| marketplace-catalog | Node.js | catálogo de produtos e API gateway |
| marketplace-orders | C# / .NET 8 | pedidos e checkout |
| marketplace-auth | Node.js | autenticação, JWT, 2FA |
| marketplace-seller | PHP / Laravel | painel do vendedor |
| marketplace-analytics | Python | relatórios de consumo e recomendação |
| marketplace-notifications | Node.js | e-mail e Discord a partir de eventos |
| marketplace-app | React Native | app do comprador |

## Stack compartilhada

- Mensageria: Kafka (Redpanda).
- Banco compartilhado: MySQL. Dados locais de cada serviço: SQLite.
- Infra: Docker Compose em uma única VM, GitHub Actions para CI.

## Como rodar

Infra local (MySQL e Redpanda):

```bash
docker compose up -d mysql redpanda
docker compose --profile tools up -d redpanda-console   # opcional, http://localhost:8080
```

Depois, cada serviço sobe com o comando de desenvolvimento descrito no README
dele, apontando para `localhost:3306` e `localhost:19092`. O `.env.example` de
cada repositório já vem com esses valores.

## Documentação

- [Arquitetura](docs/ARQUITETURA.md): contexto, containers, decisões.
- [Contratos](docs/CONTRATOS.md): portas, eventos, JWT, formato de resposta.
- [Padrões de desenvolvimento](docs/PADROES.md): estilo de código válido para todos os repositórios.

## Licença

MIT
