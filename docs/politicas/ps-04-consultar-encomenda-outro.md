# PS-04 - Política de controlo de acesso às encomendas

**Dono:** Maria Novais

# Ameaça a que responde
Consultar uma encomenda de outra pessoa conhecendo o seu ID.|/encomendas/:id|

# Objetivo
Evitar que um utilizador autenticado aceda a encomendas de terceiros apenas conhecendo o ID da encomenda.

# Âmbito
Aplica-se às rotas de leitura de encomendas e às queries que devolvem registos por ID. Não cobre autenticação de utilizadores, gestão de segredos ou regras de pagamento.