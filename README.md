📌 PROJETO – Modelagem de Banco de Dados em Grafos
Sistema de Análise de Engajamento em Rede Social (Instagram)
🎯 1. Contexto do Problema

Uma startup de análise de mídias sociais deseja desenvolver um produto capaz de gerar insights sobre:

Engajamento de usuários

Popularidade de conteúdos

Conexões entre usuários

Formação de comunidades por interesse

A solução proposta utiliza Banco de Dados em Grafos (Neo4j), pois redes sociais possuem estrutura naturalmente conectada.

🧠 2. Justificativa do Uso de Grafos

Redes sociais são compostas por:

Usuários conectados entre si

Conteúdos gerando interações

Comunidades formadas por interesses comuns

Modelar isso em banco relacional exigiria múltiplos JOINs complexos.
Em grafos, as relações são armazenadas diretamente, permitindo consultas mais performáticas e naturais.

🏗 3. Modelagem Conceitual (Arrow)
🔹 Nós (Entidades)

User

userId (único)

username

name

followersCount

followingCount

verified (boolean)

Post

postId (único)

caption

createdAt

likesCount

Comment

commentId (único)

text

createdAt

Hashtag

name (único)

Topic (Comunidade)

name

🔹 Relacionamentos

(User)-[:POSTED]->(Post)

(User)-[:FOLLOWS]->(User)

(User)-[:LIKED]->(Post)

(User)-[:COMMENTED]->(Comment)

(Comment)-[:ON_POST]->(Post)

(Post)-[:HAS_HASHTAG]->(Hashtag)

(Hashtag)-[:RELATED_TO]->(Topic)

🔐 4. Constraints (Integridade dos Dados)

Para garantir unicidade:

CREATE CONSTRAINT user_id IF NOT EXISTS
FOR (u:User) REQUIRE u.userId IS UNIQUE;

CREATE CONSTRAINT post_id IF NOT EXISTS
FOR (p:Post) REQUIRE p.postId IS UNIQUE;

CREATE CONSTRAINT comment_id IF NOT EXISTS
FOR (c:Comment) REQUIRE c.commentId IS UNIQUE;

CREATE CONSTRAINT hashtag_name IF NOT EXISTS
FOR (h:Hashtag) REQUIRE h.name IS UNIQUE;

📌 Justificativa:
Evita duplicidade de usuários, posts, comentários e hashtags.

⚡ 5. Índices (Performance)
CREATE INDEX user_username IF NOT EXISTS
FOR (u:User) ON (u.username);

CREATE INDEX post_caption IF NOT EXISTS
FOR (p:Post) ON (p.caption);

📌 Justificativa:
Melhora desempenho em buscas por username ou palavras na legenda.

🔎 6. Consultas Analíticas
🔥 Usuários com maior engajamento
MATCH (u:User)-[:POSTED]->(p:Post)
RETURN u.username, SUM(p.likesCount) AS totalLikes
ORDER BY totalLikes DESC
LIMIT 5;
🔥 Comunidades formadas por hashtag
MATCH (u:User)-[:POSTED]->(p:Post)-[:HAS_HASHTAG]->(h:Hashtag)
RETURN h.name, COUNT(DISTINCT u) AS activeUsers
ORDER BY activeUsers DESC;
🔥 Usuários que possuem interação mútua
MATCH (u1:User)-[:FOLLOWS]->(u2:User),
      (u2)-[:FOLLOWS]->(u1)
RETURN u1.username, u2.username;
🔍 Uso de CONTAINS
MATCH (p:Post)
WHERE p.caption CONTAINS "dados"
RETURN p.caption;
🔍 Uso de ENDS WITH
MATCH (u:User)
WHERE u.username ENDS WITH "dev"
RETURN u.username;
🔎 Uso de EXISTS
MATCH (u:User)
WHERE EXISTS {
    MATCH (u)-[:POSTED]->(:Post)
}
RETURN u.username;
📊 7. Uso de EXPLAIN e PROFILE

EXPLAIN:

EXPLAIN
MATCH (u:User {username: "patricia_dev"})
RETURN u;

PROFILE:

PROFILE
MATCH (u:User)-[:POSTED]->(p:Post)
RETURN u, p;

📌 EXPLAIN mostra o plano de execução da query.
📌 PROFILE executa e apresenta custo real da operação.

🏁 8. Conclusão

A modelagem proposta permite:

Analisar engajamento

Identificar usuários influentes

Detectar comunidades por interesse

Explorar conexões bidirecionais

Realizar buscas textuais eficientes

O banco de dados em grafos mostrou-se adequado para representar estruturas altamente conectadas como redes sociais.