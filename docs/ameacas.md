# Modelo de ameaças — api-encomendas

**Grupo:** Família Alameda

## Casos de abuso

Formato: «Como atacante, quero *fazer X*, para *conseguir Y*.»

1. Como atacante, quero iniciar sessão indicando apenas o email da outra pessoa, para obter um token válido e aceder à sua conta. (auth/login)
2. Como atacante, quero falsificar um token JWT usando a chave que está no código, para fazer pedidos como outro utilizador. (config.js)
3. Como atacante, quero enviar pedidos de login sem limite, para conseguir descobrir que emails existem e entupir o serviço. (server.js)
4. Como atacante, quero juntar emails, nomes e NIFs dos clientes, para conseguir fazer phishing credível em nome da loja (db.js)
5. Como atacante, quero descobrir os ids das encomendas para enganar o cliente sobre "um problema na encomenda" (encomenda.js)
6. Como atacante, quero escrever SQL no id do pedido de uma encomenda, para ler ou alterar qualquer tabela da base de dados. (encomendas.js)
7. Como atacante, quero manipular o campo `sub` do JWT para um identificador arbitrário, para aceder ao perfil de outro utilizador na rota `/utilizadores/me` sem conhecer as credenciais. (autenticacao.js, utilizadores.js)
8. Como atacante, quero conhecer um id de encomenda válido e pedir `/encomendas/:id`, para visualizar dados de encomendas de terceiros sem ter autorização. (encomendas.js)

## As quatro que não aceitamos correr

| # | Ameaça | Onde na API | Owner |
|---|--------|-------------|-------|
| 1 |Iniciar sessão só com o email de outra pessoa e obter acesso à conta.|/auth/login| Raquel |
| 2 |Usar a chave JWT exposta para criar tokens e fazer pedidos como outro utilizador, incluindo manipular o sub.|config.js, autenticacao.js, utilizadores/me| Sofia |
| 3 |Injetar SQL no ID da encomenda para tentar aceder ou alterar dados da base de dados.|/encomendas/:id| Carlos |
| 4 |Consultar uma encomenda de outra pessoa conhecendo o seu ID.|/encomendas/:id| Maria |
