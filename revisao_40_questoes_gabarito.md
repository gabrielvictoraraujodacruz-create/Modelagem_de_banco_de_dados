# Gabarito: Caderno de Revisão de Banco de Dados (40 questões)

Caderno da Profa. Josyane Lannes Florenzano de Souza, MySQL 8.0.
**Este gabarito não é o oficial.** Foi montado por mim (Hermes) e as queries dos desafios foram testadas com dados de exemplo. Quando a professora passar o oficial, vale comparar.

## Parte I: múltipla escolha

| Q | R | Por quê |
|---|---|---|
| 01 | B | `SELECT * FROM tabela` traz todas as linhas e colunas |
| 02 | C | DDL define estrutura: CREATE, ALTER, DROP |
| 03 | A | `INSERT INTO` adiciona registro |
| 04 | B | `ALTER TABLE ... ADD PRIMARY KEY` |
| 05 | D | PRIMARY KEY = único + não nulo |
| 06 | C | FK referencia uma chave única/candidata de outra tabela |
| 07 | A | filtro de linhas é `WHERE cidade = '...'` |
| 08 | C | começa com Ana: `'Ana%'` |
| 09 | B | termina em Silva: `'%Silva'` |
| 10 | B | `DISTINCT` remove duplicatas |
| 11 | C | maior para o menor = `ORDER BY ... DESC` |
| 12 | C | `BETWEEN 100 AND 500` inclui os extremos |
| 13 | A | `IN (1,3,5)` |
| 14 | C | `COUNT(*)` conta linhas, mesmo com NULL em coluna |
| 15 | B | média = `AVG(salario)` |
| 16 | A | contar por grupo = `GROUP BY cidade` |
| 17 | D | `HAVING` filtra grupos depois do GROUP BY |
| 18 | B | `GROUP BY cidade HAVING COUNT(*) > 3` |
| 19 | B | INNER JOIN só traz o que tem correspondência |
| 20 | C | LEFT JOIN traz todos da esquerda, mesmo sem par |
| 21 | B | `ON pedido.id_cliente = cliente.id_cliente` (FK = PK) |
| 22 | A | `UPDATE ... SET ... WHERE` |
| 23 | B | `DELETE FROM pedido WHERE id_pedido = 7` |
| 24 | B | DELETE sem WHERE apaga todas as linhas (a estrutura fica) |
| 25 | A | atomicidade = tudo ou nada |
| 26 | C | isolamento = transações simultâneas sem interferir |
| 27 | B | durabilidade = persiste depois do COMMIT |
| 28 | A | consistência = respeita as regras de integridade |
| 29 | B | a FK fica no lado N (PEDIDO) |
| 30 | A | `MAX(valor) ... GROUP BY id_cliente` |

**Pegadinhas:** WHERE filtra linha e HAVING filtra grupo (17 e 18). `DELETE` apaga linhas e `DROP` apaga a tabela (23 e 24). `COUNT(*)` conta linhas e `COUNT(coluna)` ignora NULL (14).

## Parte II: laboratório (modelo ALUNO, MATRICULA, CURSO)

Colunas do diagrama: ALUNO(id_aluno, nome, cidade), CURSO(id_curso, nome_curso, carga_horaria), MATRICULA(id_matricula, id_aluno, id_curso, data_matricula, nota).

### Desafio 01
```sql
CREATE DATABASE revisao_matriculas;
USE revisao_matriculas;
```

### Desafio 02
```sql
CREATE TABLE aluno (
    id_aluno INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cidade VARCHAR(60)
) ENGINE=InnoDB;

CREATE TABLE curso (
    id_curso INT AUTO_INCREMENT PRIMARY KEY,
    nome_curso VARCHAR(100) NOT NULL,
    carga_horaria INT
) ENGINE=InnoDB;

CREATE TABLE matricula (
    id_matricula INT AUTO_INCREMENT PRIMARY KEY,
    id_aluno INT NOT NULL,
    id_curso INT NOT NULL,
    data_matricula DATE,
    nota DECIMAL(4,2),
    FOREIGN KEY (id_aluno) REFERENCES aluno(id_aluno),
    FOREIGN KEY (id_curso) REFERENCES curso(id_curso)
) ENGINE=InnoDB;
```
Ordem importa: ALUNO e CURSO primeiro, MATRICULA por último, porque ela referencia as duas.

### Desafio 03
```sql
INSERT INTO aluno (nome, cidade) VALUES
('Ana Souza', 'Brasília'),
('Bruno Lima', 'Goiânia'),
('Alice Costa', 'Brasília'),
('Carlos Dias', 'Taguatinga'),
('Daniela Rocha', 'Brasília');

INSERT INTO curso (nome_curso, carga_horaria) VALUES
('SQL Básico', 60),
('Modelagem de Dados', 80),
('Python', 120),
('Excel', 30);

INSERT INTO matricula (id_aluno, id_curso, data_matricula, nota) VALUES
(1, 1, '2026-02-10', 8.5),
(1, 2, '2026-02-12', 9.0),
(2, 1, '2026-02-15', 7.0),
(2, 3, '2026-03-01', 5.0),
(3, 1, '2026-03-05', 10.0),
(3, 2, '2026-03-07', 5.5),
(4, 2, '2026-03-10', 7.5),
(4, 1, '2026-03-12', 6.0);
```
Daniela (id 5) é o aluno sem matrícula e Excel (id 4) é o curso sem matrícula.

### Desafio 04
```sql
SELECT nome, cidade FROM aluno
WHERE cidade = 'Brasília'
ORDER BY nome;
```

### Desafio 05
```sql
SELECT DISTINCT nome FROM aluno
WHERE nome LIKE 'A%';
```

### Desafio 06
```sql
SELECT nome_curso, carga_horaria FROM curso
WHERE carga_horaria BETWEEN 40 AND 80
ORDER BY carga_horaria DESC;
```

### Desafio 07
```sql
SELECT a.nome, c.nome_curso, m.data_matricula
FROM matricula m
INNER JOIN aluno a ON m.id_aluno = a.id_aluno
INNER JOIN curso c ON m.id_curso = c.id_curso;
```
A tabela do meio (MATRICULA) é quem liga as duas pontas.

### Desafio 08
```sql
SELECT c.nome_curso, COUNT(*) AS total_matriculas
FROM curso c
INNER JOIN matricula m ON m.id_curso = c.id_curso
GROUP BY c.id_curso, c.nome_curso
HAVING COUNT(*) >= 2;
```
Com os dados acima: SQL Básico (4) e Modelagem de Dados (3). Python tem 1 e fica de fora.

### Desafio 09
```sql
SELECT c.nome_curso, AVG(m.nota) AS media
FROM curso c
INNER JOIN matricula m ON m.id_curso = c.id_curso
GROUP BY c.id_curso, c.nome_curso
HAVING AVG(m.nota) >= 7;
```
Com os dados acima: SQL Básico (7,875) e Modelagem de Dados (7,33).

### Desafio 10
```sql
SELECT a.nome, COUNT(m.id_matricula) AS total_matriculas
FROM aluno a
LEFT JOIN matricula m ON m.id_aluno = a.id_aluno
GROUP BY a.id_aluno, a.nome;
```
Use `COUNT(m.id_matricula)` e não `COUNT(*)`. Assim a Daniela aparece com 0 em vez de 1.
