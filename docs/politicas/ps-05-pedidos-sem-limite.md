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

 ## Regras de implementação
A política deve ser aplicada de forma consistente em todo o fluxo de `POST /auth/login`. A implementação recomendada é:
 - Aplicar, antes de qualquer acesso à base de dados ou verificação de palavra-passe, um middleware de limitação por origem (IP) numa janela de tempo;
 - Limitar o tamanho do corpo do pedido antes de o interpretar;
 - Normalizar o email (remover espaços, converter para minúsculas) antes de o usar como chave do limite de tentativas falhadas;
 - Contar as tentativas falhadas por email numa janela de tempo e, ao exceder o limite, responder com `429` e o cabeçalho `Retry-After`;
 - Fazer o bloqueio por email temporário, com expiração automática, e combiná-lo com a origem, para que um atacante não consiga bloquear a conta de outra pessoa por tempo indefinido;
 - Procurar a conta pelo email; se não existir, executar a verificação da palavra-passe contra um hash fictício, para que o custo e o tempo de resposta sejam semelhantes aos de uma palavra-passe errada;
 - Devolver sempre a mesma resposta (mesmo código de estado, mesma mensagem, mesmo formato) para email inexistente e para palavra-passe errada;
 - Registar os pedidos bloqueados e os logins falhados (origem, email normalizado, momento), sem guardar palavras-passe;
 - Confiar em `X-Forwarded-For` apenas quando o pedido vier de um proxy configurado como fiável;
 - Devolver mensagens de erro genéricas, sem erros da base de dados nem stack traces.

Exemplo de regra válida:
 - `if (excedeLimite(ip, emailNormalizado)) return res.status(429).set('Retry-After', segundos).json({ erro: 'Demasiados pedidos' });`
 - `return res.status(401).json({ erro: 'Credenciais inválidas' });` tanto para email inexistente como para palavra-passe errada.

Exemplo de comportamento inválido:
 - Responder `404 { erro: 'Email não registado' }` quando o email não existe e `401 { erro: 'Palavra-passe errada' }` quando existe;
 - Só executar `bcrypt.compare` quando o utilizador existe, o que torna as respostas de emails inexistentes visivelmente mais rápidas;
 - Contar tentativas por email sem normalizar, permitindo contornar o limite com `Ana@x.pt` e `ana@x.pt `;
 - Ler o IP diretamente de `X-Forwarded-For` sem proxy fiável configurado;
 - Bloquear a conta indefinidamente após N falhas, sem expiração.

## Regras
| # | Regra | Como se verifica |
|---|-------|------------------|
| 1 | O sistema deve limitar o número de pedidos de login por origem (IP) e o número de tentativas falhadas por email, respondendo com `429` e `Retry-After` quando um limite é excedido | [TESTE AUTOMÁTICO FUNCIONAL] |
| 2 | A resposta a um email inexistente deve ser indistinguível da resposta a uma palavra-passe errada (mesmo código de estado, mesma mensagem, mesmo formato) | [TESTE AUTOMÁTICO FUNCIONAL] |
| 3 | O tempo de resposta do login não deve revelar se o email existe | [TESTES NÃO FUNCIONAIS] |
| 4 | O limite não deve poder ser contornado alterando cabeçalhos (`X-Forwarded-For`) nem variando o formato do email (maiúsculas, espaços) | [TESTE AUTOMÁTICO FUNCIONAL] |
| 5 | O sistema deve limitar o tamanho do corpo dos pedidos de login e não deve devolver detalhes internos (erros da base de dados, stack traces) nas respostas | [TESTES NÃO FUNCIONAIS] |
| 6 | Os pedidos bloqueados por excesso devem ser registados para deteção de abuso, e o bloqueio de uma conta deve expirar, sem permitir bloqueios indefinidos por terceiros | [TESTES NÃO FUNCIONAIS] |

## Responsabilidades
- Dono PS: Gonçalo
- Aprova exceções/Revisor: Raquel
- Cumpre/Executa: Sofia