# PS-04 - Política de controlo de acesso às encomendas de outros utilizadores

**Dono:** Maria Novais

## Ameaça a que responde
Um utilizador autenticado consegue consultar uma encomenda pertencente a outro utilizador apenas sabendo o identificador da encomenda (`/encomendas/:id`).

## Objetivo
Garantir que o acesso a uma encomenda só é permitido quando o utilizador autenticado for o dono da encomenda ou tiver autorização explícita para a consultar, mesmo que conheça o ID do registo.

Deve:
 - Verificar, em todas as rotas de leitura de encomendas, se o utilizador autenticado está associado à encomenda que está a tentar consultar;
 - Comparar a identidade do utilizador com o proprietário da encomenda antes de devolver qualquer detalhe da encomenda;
 - Rejeitar pedidos com resposta `403` (ou `404` em algumas implementações, se for política da API) quando o utilizador não for autorizado;
 - Validar a associação entre o utilizador e a encomenda mesmo quando a consulta é feita por ID, por query string ou por qualquer outra chave de identificação;
 - Mantém a lógica de autorização centralizada numa função ou middleware reutilizável, em vez de espalhar verificações por rotas; 
 - Aplicar o mesmo controlo a listagens, filtros e endpoints que devolvam dados parciais de encomendas de terceiros;
 - Registar eventos de tentativa de acesso não autorizado para auditoria e deteção de abuso;
 - Garantir que a resposta não revele informação sensível sobre a encomenda ou sobre o proprietário quando o acesso for negado.

Não deve:
 - Permitir que um utilizador aceda a encomendas de outros apenas porque sabe o `id` da encomenda;
 - Confiar apenas em parâmetros públicos, como ID ou token de sessão, sem confirmar a propriedade;
 - Devolver dados de encomenda em respostas de erro para sinalizar que um registo existe;
 - Usar filtragem apenas no cliente ou em query strings para ocultar dados que não deveriam sair do servidor;
 - Fazer verificações de autorização só em endpoints específicos sem cobrir endpoints equivalentes;
 - Permitir contornar a regra por variantes de URL, query params ou IDs alternativos que apontem para o mesmo registo;
 - Basear a autorização numa comparação fraca, por exemplo apenas validar a presença de um token sem confirmar o dono da encomenda;
 - Expor detalhes internos da base de dados, stack traces ou mensagens de erro que indiquem o estado da encomenda do outro utilizador.

## Âmbito
Aplica-se a:
 - A rota `GET /encomendas/:id`, implementada em `src/routes/encomendas.js`, e a qualquer rota de consulta de encomenda por identificador;
 - Qualquer endpoint que leia, liste ou filtre encomendas com base no ID do utilizador autenticado ou no ID da encomenda;
 - A lógica de acesso aos dados em `src/db.js` e qualquer camada que execute consultas de leitura de encomendas;
 - Filtros e middlewares de autorização aplicados antes de devolver dados ao cliente;
 - Alterações futuras que acrescentem novas operações de leitura sobre encomendas.

Não se aplica a:
 - A autenticação em si, que é tratada em `src/autenticacao.js`, `src/routes/auth.js` e nos fluxos de JWT;
 - A validação do formato do `id` da encomenda, que é tratada pela política de prevenção de injeção SQL;
 - Regras de pagamento, faturação ou gestão de segredos;
 - O controlo de acesso a outros recursos, como utilizadores, sessões ou tokens, que têm políticas próprias;
 - Ataques de negação de serviço ou limitações de pedidos, que são abordados em outras políticas de segurança.

## Regras de implementação
A política deve ser aplicada de forma consistente em todos os pontos de leitura de encomendas. A implementação recomendada é:
 - Obter o utilizador autenticado do contexto da requisição;
 - Carregar a encomenda pelo `id` fornecido;
 - Confirmar se a encomenda existe e se o `userId` da encomenda coincide com o utilizador autenticado;
 - Se não coincidir, negar o acesso sem revelar mais informação do que o necessário;
 - Se coincidir, permitir a leitura do registo;
 - Aplicar a mesma lógica a listagens das quais uma encomenda possa ser inferida por associação, filtros ou joins.

Exemplo de regra válida:
 - `if (encomenda.userId !== req.user.id) return res.status(403).json({ erro: 'Acesso negado' });`

Exemplo de comportamento inválido:
 - `SELECT * FROM encomendas WHERE id = req.params.id;` sem verificar a associação entre utilizador e encomenda;
 - Devolver `200 OK` com dados da encomenda mesmo quando o utilizador autenticado não é o dono;
 - Basear a autorização em headers ou campos parcialmente controlados pelo cliente.

## Regras
| # | Regra | Como se verifica |
|---|-------|------------------|
| 1 | A interface/sistema não deve permitir que um utilizador autenticado consulte encomendas pertencentes a outro utilizador apenas conhecendo o ID da encomenda | [TESTE AUTOMÁTICO FUNCIONAL] |
| 2 | A interface/sistema não deve devolver detalhes que revelem a existência, o conteúdo ou a propriedade de uma encomenda de outro utilizador em respostas de erro ou de acesso negado | [TESTES NÃO FUNCIONAIS] |

## Responsabilidades
- Dono PS: Maria Novais
- Aprova exceções/Revisor: Gonçalo Pereira
- Cumpre/Executa: Raquel Azevedo