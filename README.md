<div align="center">

# Sistema de Gestão de Aprendizagem (BD)

![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat-square&logo=sqlite&logoColor=white)

</div>

## Sobre

Projeto desenvolvido durante o curso de **SQLite da Origamid**, com o objetivo de modelar o banco de dados de um LMS (Learning Management System): usuários, cursos, aulas, progresso de conclusão e certificados. O foco não é a aplicação em si, mas o banco, desde a modelagem das tabelas até consultas, views, triggers e otimizações.

## O que aprendi / O que pratiquei:

- **Modo `STRICT` e `WITHOUT ROWID`**: usei `STRICT` nas tabelas para forçar checagem de tipos (algo que o SQLite não faz por padrão) e `WITHOUT ROWID` em tabelas com chave primária composta ou textual (`resets`, `sessions`, `lessons_completed`, `certificates`), evitando o rowid implícito desnecessário.
- **Chaves estrangeiras com `ON DELETE CASCADE`**: ao deletar um usuário, suas sessões, resets e conclusões de aula são removidos automaticamente, testei isso na prática deletando o usuário de id 10.
- **Índices**: criei índices em `resets.user_id`, `sessions.user_id` e no par `(course_id, "order")` de `lessons`, e usei `EXPLAIN QUERY PLAN` para confirmar que as consultas estavam de fato usando os índices.
- **Views**: criei `lessons_completed_full` e `certificates_full` para juntar dados de várias tabelas em uma consulta só. Também tentei uma view `lesson_nav` para navegação entre aula anterior/próxima, mas percebi que a lógica de `BETWEEN "order" - 1 AND "order" + 1` não resolve corretamente o caso, deixei o comentário no código como registro do problema para revisar depois.
- **Triggers**: um trigger `AFTER UPDATE` na tabela `users` que atualiza automaticamente o campo `updated` sempre que o registro é alterado, sem precisar lembrar de fazer isso manualmente em cada UPDATE.
- **Tokens e expiração com Unix timestamp**: as tabelas `sessions` e `resets` usam `strftime('%s', 'now', '+N days')` para gerar expiração, e uma query pronta para simular um cron job de limpeza (`DELETE FROM sessions WHERE expires < strftime('%s', 'now')`).
- **PRAGMAs de configuração**: ajustei `journal_mode = WAL`, `synchronous = NORMAL`, `cache_size`, `busy_timeout` e `optimize` para simular um ambiente mais próximo de produção, e não só o padrão do SQLite.
- **Geração de IDs aleatórios**: usei `lower(hex(randomblob(8)))` para gerar o id dos certificados como uma string hexadecimal, em vez de depender de autoincremento.

## Como rodar

Você vai precisar do [SQLite](https://www.sqlite.org/download.html) instalado (ou de um cliente gráfico como o [DB Browser for SQLite](https://sqlitebrowser.org/)).

```bash
# Clone o repositório
git clone https://github.com/niusdev/Sistema_de_Gestao_de_Aprendizagem_bd.git

# Entre na pasta do projeto
cd Sistema_de_Gestao_de_Aprendizagem_bd

# Abra o banco já populado
sqlite3 dbLMS.sqlite
```

> O arquivo `querrysLMS.sql` contém toda a criação do banco (tabelas, dados, views, triggers) e também as consultas usadas para testar cada parte. Pode ser executado do zero com `sqlite3 novo_banco.sqlite < querrysLMS.sql`, ou lido como referência junto com o `dbLMS.sqlite` já pronto.

## Estrutura do banco

```
users                 # usuários da plataforma
resets                # tokens de redefinição de senha
sessions              # tokens de sessão ativa
courses               # cursos disponíveis
lessons               # aulas de cada curso
lessons_completed     # progresso do usuário por aula
certificates          # certificados emitidos por curso concluído

views:
  lessons_completed_full  # progresso com nome do curso/aula
  certificates_full       # certificados com dados do curso e do usuário
  lesson_nav              # navegação entre aulas (com bug conhecido)
```

## Autor

Feito por **Vinícius Gomes Damascena**

[![GitHub](https://img.shields.io/badge/GitHub-niusdev-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/niusdev)
