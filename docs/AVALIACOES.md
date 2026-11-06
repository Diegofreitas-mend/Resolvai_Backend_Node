# Avaliações (#92)

Aplicar `db/scripts/013_create_avaliacoes.sql` antes de publicar esta versão. Nenhum script é aplicado automaticamente na inicialização. A tabela usa chave primária por pedido, limites de nota/comentário, FKs e RLS sem políticas públicas. Somente o backend com credencial de banco autorizada deve acessá-la.

Rotas autenticadas, também disponíveis no Swagger:

- `POST /api/v1/orders/{id}/review`: `{ rating: 1..5, comment?: string }`, comentário com até 2000 caracteres. Retorna 201 com `{ orderId, providerId, rating, comment?, createdAt }`.
- `GET /api/v1/orders/{id}/review`: avaliação do próprio pedido ou `null` quando não há avaliação; 404 para outro proprietário.
- `GET /api/v1/users/me/reviews/summary`: quantidade e média das notas **enviadas** pelo cliente; média nula sem avaliações.

Somente clientes podem avaliar seus pedidos concluídos. A identificação do prestador vem da única proposta escolhida; pedidos legados sem proposta escolhida ou com mais de uma são recusados com 409. Não é criada avaliação para um prestador presumido. Nota inválida ou status não concluído retorna 400, falta de autenticação 401 e pedido inacessível 404. Reenvios retornam 409 e não alteram a avaliação original.

A criação bloqueia a linha do pedido e as propostas escolhidas durante a transação. `ON CONFLICT` e a chave primária impedem duplicidade independentemente do frontend. Comentários em branco são omitidos. Não se publicam nome, e-mail, CPF ou endereço do cliente na resposta.

Integração: frontend da mesma branch no Resolvai_Cliente. Esta PR de backend referencia a issue do frontend sem encerrá-la. A implantação da migration e da API deve preceder a ativação do frontend.

Testes: `npm run typecheck`, `npm test`, `npm run build`. A integração usa PostgreSQL PGlite efêmero com os scripts SQL reais, driver pg e Fastify. Inclui autorização, status, entradas inválidas, duas requisições HTTP simultâneas (pool de teste com uma conexão), leitura, resumo e Swagger. A unicidade também é garantida pelo esquema SQL; não há teste de carga em PostgreSQL externo nesta suíte.
