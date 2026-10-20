# marketplace

Repositório guarda-chuva de um marketplace construído como ecossistema de
microsserviços. Aqui ficam o guia de padrões, o docker-compose que sobe tudo
e o diagrama geral. Cada serviço vive no próprio repositório.

## Serviços

| Repositório | Linguagem | Papel |
|---|---|---|
| marketplace-web | React + TypeScript | vitrine pública e landing |
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

Em breve, quando os primeiros serviços existirem.

## Documentação

- [Padrões de desenvolvimento](docs/PADROES.md)

## Licença

MIT
