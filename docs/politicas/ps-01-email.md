# PS-01 - Política de autenticação por email

**Dono:** Raquel Azevedo

## Ameaça a que responde
Iniciar sessão com o email de outra pessoa e obter acesso à conta.


## Objetivo
Garantir que cada conta só é acessada por um único user
Evitar que uma conta tenha mais doq um email de acesso

Deve: 
 - Cada conta deve estar associada a um único utilizador.
 - Cada conta deve ter apenas um email autorizado para iniciar sessão.
 - O sistema deve confirmar que o utilizador controla o email antes de o associar a uma conta.
 - O acesso deve ser concedido apenas quando o email usado para iniciar sessão corresponde ao email associado à conta.
 - A alteração do email de acesso deve exigir a confirmação do novo endereço.
 - O sistema deve impedir que um email já associado a outra conta seja usado para aceder ou reclamar essa conta.

Não deve:
 - Permitir iniciar sessão numa conta apenas por conhecer ou introduzir o email de outra pessoa.
 - Permitir que vários utilizadores acedam à mesma conta com emails diferentes.
 - Permitir associar mais do que um email de acesso à mesma conta.
 - Permitir alterar o email de acesso sem verificar que o utilizador controla o novo endereço.
 - Permitir que um utilizador associe à sua conta um email que pertence a outra pessoa.


## Ambito
Aplica-se a:
 - O processo de autenticação pelo endpoint POST /auth/login, implementado em auth.js.
 - A procura da conta pelo email na base de dados, através de db.js.
 - A emissão do token de acesso após o login e o acesso às rotas protegidas com esse token, como /utilizadores/me e as rotas de    encomendas.
 - A associação ou alteração do email de uma conta, caso estas funcionalidades sejam implementadas.

Não se aplica a:
 - A falsificação ou manipulação de tokens JWT — essa é uma ameaça separada, descrita no modelo em config.js, autenticacao.js e /utilizadores/me.
 - A tentativa repetida de descobrir emails ou sobrecarregar o login — corresponde à ameaça de pedidos sem limite.
 - A consulta ou alteração de encomendas por ID, incluindo injeção SQL ou acesso a encomendas de terceiros.
 - A recuperação de palavra-passe, criação de contas ou confirmação de email, pois essas funcionalidades não aparecem entre os endpoints documentados atualmente.


## Regras
| # | Regra | Como se verifica |
|---|-------|------------------|
| 1 | O sistema deve permitir o acesso a uma conta apenas quando o email utilizado no login corresponde ao email associado a essa conta. | [TESTE AUTOMÁTICO FUNCIONAL]
| 2 | Cada conta deve estar associada a um único email de acesso. | [TESTE AUTOMÁTICO FUNCIONAL]
| 3 | Um email só pode estar associado a uma única conta. | [TESTE AUTOMÁTICO FUNCIONAL]
| 4 | O sistema deve confirmar que o utilizador controla um novo email antes de o associar ou substituir o email da conta. | [TESTE AUTOMÁTICO FUNCIONAL]
| 5 | Não deve ser possível iniciar sessão numa conta apenas conhecendo ou introduzindo o email de outra pessoa. | [TESTE AUTOMÁTICO FUNCIONAL]
| 6 | Após autenticação bem-sucedida, o token emitido deve identificar exclusivamente a conta associada ao email autenticado. | [TESTE AUTOMÁTICO FUNCIONAL]
| 7 | A procura da conta pelo email na base de dados deve devolver no máximo uma conta para cada email. | [TESTE NÃO FUNCIONAL]
| 8 | A base de dados deve impedir a existência de emails duplicados associados a contas diferentes. | [TESTE NÃO FUNCIONAL]
| 9 | O sistema não deve permitir a reclamação ou apropriação de uma conta através da associação de um email pertencente a outro utilizador. | [TESTE AUTOMÁTICO FUNCIONAL]


## Responsabilidades
- Dono PS: Raquel Azevedo
- Revisor: Sofia 
- Executor: Carlos