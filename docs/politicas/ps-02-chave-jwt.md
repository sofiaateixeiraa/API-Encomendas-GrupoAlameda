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
