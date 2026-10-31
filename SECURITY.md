# Política de Segurança — api-encomendas

**Grupo:** Família Alameda · **Versão:** 0.1 · **Data:** 2026-10-30

## Objetivo

Reduzir o risco de abuso do login, acesso indevido a contas e encomendas, falsificação de identidade através de JWT e injeção SQL na API.

## Âmbito

Aplica-se ao código em `src/`, aos testes em `test/` e às alterações ao pipeline em `.github/workflows/`, em particular aos pedidos de alteração dirigidos ao ramo `main` e à execução da API nos ambientes de desenvolvimento, teste e produção.

Não define controlos para a infraestrutura da base de dados, proteção de dados contra phishing, ataques volumétricos ao nível da rede ou limitação de pedidos a endpoints diferentes do login.

## Regras

| # | Regra | Como se verifica |
|---|-------|------------------|
| 1 | O login só pode conceder acesso à conta associada ao email verificado do utilizador, cada email pertence a, no máximo, uma conta. | Testes funcionais de login, associação e alteração de email, verificação da unicidade na base de dados. |
| 2 | A chave JWT deve ser secreta, própria de cada ambiente e obtida de configuração segura. A API rejeita tokens inválidos, expirados ou sem identidade válida. | Verificar a configuração e os segredos sem os revelar, testar a rejeição de tokens inválidos e expirados. |
| 3 | Dados recebidos do cliente são validados no servidor e enviados às consultas SQL como parâmetros, nunca concatenados ao texto SQL. | Testes com identificadores inválidos e inspeção das consultas para confirmar a parametrização. |
| 4 | Só o proprietário autorizado pode consultar uma encomenda, as respostas de acesso negado não revelam os dados nem a existência do registo. | Testes funcionais com utilizadores diferentes e inspeção das respostas de acesso negado. |
| 5 | O login limita pedidos por origem e tentativas falhadas por email, não revela se um email existe e não permite contornar os limites por cabeçalhos ou formatos diferentes do email. | Testes funcionais dos limites e das respostas, verificação do tamanho máximo do pedido, do registo de bloqueios e da configuração de proxies fiáveis. |

## Quem aprova

As alterações são submetidas por Pull Request para `main` e exigem aprovação do responsável pelo repositório, além da aprovação dos testes do pipeline. Alterações a `.github/workflows/` só podem ser aprovadas e integradas por um responsável com permissões de administração do repositório. Os donos e revisores das políticas específicas estão indicados em `docs/ameacas.md` e nos respetivos ficheiros.

## Relato de vulnerabilidades

Quem identificar uma vulnerabilidade deve abrir uma *issue* no repositório com a etiqueta `segurança`, descrevendo o problema e os passos para o reproduzir. Não incluir na issue pública chaves, tokens, palavras-passe ou dados pessoais reais, se forem necessários para demonstrar o problema, devem ser removidos ou substituídos por valores fictícios.

## Políticas específicas

| Código | Política | Ameaça | Dono |
|--------|----------|--------|------|
| PS-01 | [Autenticação por email](docs/politicas/ps-01-email.md) | Acesso a uma conta apenas por conhecer o email de outra pessoa. | Raquel Azevedo |
| PS-02 | [Proteção da chave JWT](docs/politicas/ps-02-chave-jwt.md) | Uso de uma chave exposta para criar tokens e agir como outro utilizador. | Sofia Teixeira |
| PS-03 | [Prevenção de injeção SQL](docs/politicas/ps-03-injetar-sql.md) | Injeção SQL através do identificador da encomenda ou de outros parâmetros. | Carlos Costa |
| PS-04 | [Controlo de acesso às encomendas](docs/politicas/ps-04-consultar-encomenda-outro.md) | Consulta de encomendas de terceiros apenas com conhecimento do identificador. | Maria Novais |
| PS-05 | [Limitação de pedidos](docs/politicas/ps-05-pedidos-sem-limite.md) | Pedidos de login sem limite para descobrir emails registados ou esgotar recursos do serviço. | Gonçalo Pereira |
