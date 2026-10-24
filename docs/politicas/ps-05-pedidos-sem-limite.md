# PS-05 - Política de limitação de pedidos

**Dono:** Gonçalo Pereira

## Ameaça a que responde
Enviar pedidos de login sem limite, para descobrir que emails existem e entupir o serviço.

## Objetivo
Garantir que o login não pode ser usado para descobrir que emails estão registados
Evitar que pedidos em massa esgotem os recursos do serviço

Deve:
 - O sistema deve limitar o número de pedidos de login por origem (IP) numa janela de tempo.
 - O sistema deve limitar o número de tentativas falhadas por email numa janela de tempo.
 - O sistema deve responder com 429 (e cabeçalho Retry-After) quando um limite é excedido.
 - A resposta a um email inexistente deve ser indistinguível da resposta a uma palavra-passe errada (mesmo código de estado, mesma mensagem, mesmo formato).
 - O tempo de resposta não deve revelar se o email existe (a verificação da palavra-passe deve ter custo semelhante nos dois casos).
 - O sistema deve limitar o tamanho do corpo dos pedidos de login.
 - O sistema deve registar os pedidos bloqueados por excesso, para deteção de abuso.
 - O limite por origem só deve confiar em cabeçalhos como X-Forwarded-For quando vierem de um proxy configurado como fiável.

Não deve:
 - Permitir um número ilimitado de pedidos de login, de qualquer origem.
 - Revelar, por mensagem, código de estado ou tempo de resposta, se um email está registado.
 - Permitir que o limite seja contornado alterando cabeçalhos ou variando o formato do email (maiúsculas, espaços).
 - Executar a verificação de palavra-passe (operação custosa) sem estar sujeita ao limite.
 - Permitir que um atacante bloqueie por tempo indefinido a conta de outra pessoa através de tentativas falhadas.
 - Devolver detalhes internos (erros da base de dados, stack traces) nas respostas de login.

## Ambito
Aplica-se a:
 - O endpoint POST /auth/login, implementado em auth.js, incluindo o formato e o tempo das respostas de sucesso e de erro.
 - A procura da conta pelo email na base de dados, através de db.js, quando executada por pedidos de login.
 - A configuração de limites de pedidos, tamanho do corpo e proxies fiáveis na aplicação.
 - O registo de eventos de login falhado ou bloqueado.

Não se aplica a:
 - O uso do email de outra pessoa para entrar na conta — essa é uma ameaça separada, sobre a associação entre conta e email.
 - A falsificação ou manipulação de tokens JWT — ameaça separada, descrita no modelo em config.js, autenticacao.js e /utilizadores/me.
 - Pedidos em massa a rotas que não sejam o login, como /utilizadores/me ou as rotas de encomendas.
 - A consulta ou alteração de encomendas por ID, incluindo injeção SQL ou acesso a encomendas de terceiros.
 - A recuperação de palavra-passe, criação de contas ou confirmação de email, pois essas funcionalidades não aparecem entre os endpoints documentados atualmente.
 - Ataques de negação de serviço volumétricos ao nível da rede, que devem ser tratados na infraestrutura (firewall, CDN, provedor).