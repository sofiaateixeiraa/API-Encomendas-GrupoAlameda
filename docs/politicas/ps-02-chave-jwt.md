# PS-02 - Política de proteção da chave JWT

**Dono:** Sofia Teixeira

## Ameaça a que responde
Usar a chave JWT exposta para criar tokens e fazer pedidos como outro utilizador, incluindo manipular o `sub`.

## Objetivo
Impedir que alguém sem autorização consiga criar ou alterar tokens JWT válidos para aceder a contas de outros utilizadores.

Deve:
 - A chave usada para assinar e validar tokens JWT deve ser secreta, aleatória e suficientemente forte;
 - A chave deve ser fornecida através de configuração segura do ambiente, e não escrita diretamente no código ou no repositório;
 - A aplicação deve falhar ao iniciar se a chave não estiver configurada, em vez de usar uma chave padrão ou de desenvolvimento;
 - Os ambientes de desenvolvimento, teste e produção devem usar chaves próprias, guardadas nos respetivos sistemas de gestão de segredos;
 - A aplicação deve validar a assinatura e a validade temporal do token antes de aceitar os seus dados, incluindo o `sub`;
 - A chave deve poder ser substituída de forma controlada caso seja exposta.

Não deve:
 - Guardar uma chave JWT real no código, no repositório ou em ficheiros de configuração versionados;
 - Usar chaves previsíveis, partilhadas publicamente ou iguais entre ambientes;
 - Aceitar tokens com assinatura inválida, expirados ou sem os dados de identidade necessários;
 - Registar a chave ou tokens completos nos logs.

## Ambito
Aplica-se a:
 - A configuração da chave JWT em `src/config.js` e nos ambientes onde a API é executada;
 - A emissão de tokens no endpoint `POST /auth/login`, implementada em `src/routes/auth.js`;
 - A validação de tokens no middleware `src/autenticacao.js` e o acesso às rotas protegidas, como `/utilizadores/me` e `/encomendas`;
 - A gestão de segredos em desenvolvimento, teste, produção e nos sistemas de integração e entrega contínuas (CI/CD);
 - Alterações ao código, configuração ou infraestrutura que possam expor ou enfraquecer a chave JWT.

Não se aplica a:
 - A validação da palavra-passe ou a associação entre email e conta no login;
 - A verificação de que uma encomenda pertence ao utilizador autenticado;
 - A gestão de outros segredos da aplicação que não sejam usados para assinar ou validar tokens JWT.

 ## Regras
| # | Regra | Como se verifica |
|---|-------|------------------|
| 1 | A aplicação não deve iniciar se a chave JWT não estiver configurada ou estiver vazia, nem deve recorrer a uma chave padrão. | Iniciar a aplicação sem configurar a chave e com a chave vazia; confirmar em ambos os casos que a inicialização falha e que não é usada uma chave padrão. [TESTE AUTOMÁTICO FUNCIONAL] |
| 2 | A chave JWT deve ser obtida de configuração segura e não deve estar escrita no código, em ficheiros versionados ou no repositório. | Procurar credenciais e valores de chave nos ficheiros versionados e no histórico do repositório; confirmar que a aplicação obtém a chave da configuração segura do ambiente. [VERIFICAÇÃO NÃO FUNCIONAL] |
| 3 | Cada ambiente deve usar uma chave secreta, aleatória e própria, com força adequada ao algoritmo de assinatura configurado. | Inspecionar a configuração do gestor de segredos e confirmar que desenvolvimento, teste e produção têm segredos distintos; verificar que a chave foi gerada segundo os requisitos de força do algoritmo, sem revelar o seu valor. [VERIFICAÇÃO NÃO FUNCIONAL] |
| 4 | O sistema deve rejeitar tokens com assinatura inválida ou alterados, incluindo tokens cujo `sub` tenha sido manipulado. | Enviar a uma rota protegida um token assinado com outra chave e outro token válido cuja assinatura tenha sido alterada após mudar o `sub`; confirmar que ambos recebem erro 401 e não dão acesso. [TESTE AUTOMÁTICO FUNCIONAL] |
| 5 | O sistema deve rejeitar tokens expirados ou sem o `sub` necessário para identificar o utilizador. | Enviar a uma rota protegida um token expirado e um token válido sem `sub`; confirmar que ambos recebem erro 401 e não dão acesso. [TESTE AUTOMÁTICO FUNCIONAL] |
| 6 | A chave JWT deve poder ser substituída através do processo de gestão de segredos, e tokens assinados com a chave antiga devem deixar de ser aceites após a substituição. | Substituir a chave no ambiente de teste; confirmar que um token emitido com a chave antiga recebe erro 401 e que um token emitido com a nova chave é aceite. [TESTE AUTOMÁTICO FUNCIONAL] |
| 7 | A chave JWT e os tokens completos não devem ser incluídos em logs, mensagens de erro ou respostas da API. | Provocar erros de autenticação e inspecionar as respostas da API e os logs correspondentes; confirmar que não contêm a chave nem tokens completos. [TESTE AUTOMÁTICO FUNCIONAL] |

 ## Responsabilidades
Dono PS: Sofia Teixeira  
Aprova execuções/Revisor: Carlos Costa  
Cumpre/Executa: Maria Novais  
