# Modelo de ameaças — api-encomendas

**Grupo:** Família Alameda

## Casos de abuso

Formato: «Como atacante, quero *fazer X*, para *conseguir Y*.»

1. Como atacante, quero iniciar sessão indicando apenas o email da outra pessoa, para obter um token válido e aceder à sua conta. (auth/login)
2. Como atacante, quero falsificar um token JWT usando a chave que está no código, para fazer pedidos como outro utilizador. (config.js)
3. Como atacante, quero enviar pedidos de login sem limite, para conseguir descobrir que emails existem e entupir o serviço. (server.js)
4. Como atacante, quero juntar emails, nomes e NIFs dos clientes, para conseguir fazer phishing credível em nome da loja
5. Como atacante, quero descobrir os ids das encomendas para enganar o cliente sobre "um problema na encomenda" (encomenda.js)
6. Como atacante, quero escrever SQL no id do pedido de uma encomenda, para ler ou alterar qualquer tabela da base de dados. (encomendas.js)

## As quatro que não aceitamos correr

| # | Ameaça | Onde na API |
|---|--------|-------------|
| 1 |        |             |
| 2 |        |             |
| 3 |        |             |
| 4 |        |             |
