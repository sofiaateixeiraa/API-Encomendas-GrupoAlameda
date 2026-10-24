# PS-03 - Política de prevenção de injeção SQL

**Dono:** Carlos Costa

## Ameaça a que responde
Injetar SQL no identificador da encomenda (`/encomendas/:id`) para ler ou alterar dados da base de dados que não deveriam estar acessíveis.

## Objetivo
Impedir que um valor enviado pelo cliente no `id` seja interpretado como código SQL, garantindo que é sempre tratado apenas como um dado.

Deve:
 - Validar que o `id` recebido é um identificador válido antes de o usar, e responder com erro 400 se não for;
 - Usar consultas parametrizadas mantendo o valor do `id` separado do texto da consulta SQL;
 - Construir as consultas apenas com texto fixo definido no código.
 - Usar na base de dados uma conta com os privilégios mínimos necessários (por exemplo, sem permissão para alterar o esquema ou aceder a outras tabelas);
 - Devolver mensagens de erro genéricas ao cliente, sem expor detalhes da consulta ou da base de dados;
 - Aplicar as mesmas regras a qualquer outro parâmetro que chegue a uma consulta.

Não deve:
 - Concatenar ou interpolar o `id`, ou qualquer outro dado do cliente, numa string SQL;
 - Confiar na validação feita apenas no cliente;
 - Tentar "limpar" o valor com listas de caracteres proibidos em vez de usar parâmetros;
 - Expor ao cliente erros da base de dados e texto das consultas;
 - Registar nos logs dados sensíveis que façam parte da consulta.

## Âmbito
Aplica-se a:
 - A rota `GET /encomendas/:id`, implementada em `src/routes/encomendas.js`, que hoje constrói a consulta por concatenação (`"SELECT * FROM encomendas WHERE id = " + req.params.id`);
 - A função `query` em `src/db.js`, que deve passar a aceitar parâmetros separados do texto SQL;
 - Qualquer outra rota ou módulo que construa consultas com dados vindos do utilizador, incluindo `/utilizadores` e `/auth/login`;
 - Alterações futuras ao código de acesso à base de dados.

Não se aplica a:
 - A verificação de que a encomenda pertence ao utilizador autenticado;
 - A criação ou validação dos tokens JWT;
 - A proteção da própria base de dados a nível de infraestrutura (rede, cópias de segurança, cifragem).

## Regras

| # | Regra | Como se verifica |
|---|-------|------------------|
| 1 | Validar o `id` recebido é um número inteiro válido antes de o usar na consulta | Verificar que o código testa com valores não-numéricos (ex. `abc`) e responde com erro 400 |
| 2 | Usar consultas parametrizadas o parâmetro deve ser separado do texto SQL | Confirmar que a consulta usa `?`.
| 3 | Validar todos os parâmetros da consulta no servidor antes de os usar | Confirmar que há validação server-side|
| 4 | Devolver mensagens de erro genéricas que não exponham detalhes da BD ou da consulta | Testar uma consulta inválida e confirmar que a resposta ao cliente não inclui consultas e outras informações pertinentes para ataques |
| 5 | A conta de base de dados deve ter os privilégios mínimos necessários | Verificar que a conta não tem permissões para alterar, aceder e eliminar dados |

## Responsabilidade
Dono PS: Carlos Costa  
Revisor: Maria  
Executor: Gonçalo
