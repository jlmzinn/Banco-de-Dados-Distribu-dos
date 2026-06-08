# Questão 1
# GEEK Commerce – Estratégia de Banco de Dados Distribuído

## Descrição do Cenário

A GEEK Commerce é uma varejista online que possui quatro filiais distribuídas pelo Brasil, cada uma com características específicas de rede e responsabilidades operacionais.

| Site   | Localização      | Link de Rede      | Observação                          |
| ------ | ---------------- | ----------------- | ----------------------------------- |
| Site 1 | Porto Velho/RO   | 2 MB              | Matriz – Dados estratégicos         |
| Site 2 | Seropédica/RJ    | 512 kb (satélite) | Vendas e notas fiscais – Sudeste    |
| Site 3 | Teofilândia/BA   | 256 kb (satélite) | Vendas e notas fiscais – Nordeste   |
| Site 4 | Guajará-Mirim/RO | 1 MB              | Marketing e inteligência de negócio |

As principais tabelas do sistema são:

* Cliente
* Produto
* Sexo
* Nota Fiscal

---

# Estratégia de Distribuição dos Dados

## Tabela Cliente

**Técnica escolhida:** Fragmentação Horizontal

### Justificativa

A tabela de clientes possui grande volume de registros e pode ser dividida por região geográfica. Dessa forma, cada filial acessa principalmente os clientes de sua área de atuação, reduzindo o tráfego entre os sites e melhorando o desempenho das consultas.

---

## Tabela Produto

**Técnica escolhida:** Replicação Total

### Justificativa

O catálogo de produtos precisa estar disponível em todas as filiais para consultas e operações de venda. Como as atualizações são menos frequentes que as consultas, a replicação total garante acesso rápido e alta disponibilidade dos dados.

---

## Tabela Sexo

**Técnica escolhida:** Replicação Total

### Justificativa

Trata-se de uma tabela pequena e praticamente estática. O custo de replicação é mínimo, permitindo que todas as filiais realizem consultas e análises de marketing sem necessidade de comunicação constante pela rede.

---

## Tabela Nota Fiscal

**Técnica escolhida:** Fragmentação Horizontal

### Justificativa

As notas fiscais são geradas localmente em cada região e apresentam crescimento constante. A fragmentação horizontal permite armazenar os registros próximos de onde são produzidos, reduzindo o tráfego nos links de comunicação e aumentando a eficiência das operações.

---

# Resumo das Técnicas Aplicadas

| Tabela      | Técnica Escolhida       |
| ----------- | ----------------------- |
| Cliente     | Fragmentação Horizontal |
| Produto     | Replicação Total        |
| Sexo        | Replicação Total        |
| Nota Fiscal | Fragmentação Horizontal |

---

# Conclusão

A solução proposta busca equilibrar desempenho, disponibilidade e eficiência da rede. As tabelas **Produto** e **Sexo** utilizam **Replicação Total**, pois possuem baixo custo de sincronização e alta necessidade de acesso em todos os sites. Já as tabelas **Cliente** e **Nota Fiscal** utilizam **Fragmentação Horizontal**, reduzindo o tráfego de dados entre filiais e melhorando o desempenho das consultas regionais.

Essa abordagem é adequada para o cenário da GEEK Commerce, especialmente devido às limitações de largura de banda dos links via satélite presentes em algumas filiais.

# Questão 2 – Justificativa Técnica das Escolhas

## Cliente – Fragmentação Horizontal

A tabela Cliente foi distribuída utilizando fragmentação horizontal porque possui um grande volume de registros e cada filial tende a acessar principalmente os clientes de sua própria região. Como os sites de Seropédica/RJ e Teofilândia/BA possuem links de baixa velocidade (512 kb e 256 kb via satélite), manter os dados dos clientes localmente reduz o tráfego de rede e melhora o desempenho das consultas. Além disso, a atualização dos dados ocorre com frequência moderada, tornando a fragmentação mais eficiente do que a replicação completa, que geraria um custo elevado de sincronização entre os sites.

## Produto – Replicação Total

A tabela Produto foi definida com replicação total porque o catálogo de produtos precisa estar disponível em todas as filiais para consultas e operações de venda. Embora existam atualizações ocasionais de preços, descrições ou estoque, a frequência dessas alterações é menor que a frequência de consultas realizadas pelos usuários. Como o volume de dados é relativamente controlado, o custo de replicação é aceitável e garante alta disponibilidade das informações, evitando dependência constante dos links de rede entre as unidades.

## Sexo – Replicação Total

A tabela Sexo é pequena, possui poucos registros e sofre raríssimas atualizações. Dessa forma, a replicação total apresenta um custo praticamente irrelevante de armazenamento e sincronização. Além disso, a disponibilidade dos dados em todos os sites facilita a execução de relatórios, análises estatísticas e campanhas de marketing sem necessidade de acesso remoto a outras filiais, melhorando a eficiência do sistema distribuído.

## Nota Fiscal – Fragmentação Horizontal


A tabela Nota Fiscal foi distribuída por fragmentação horizontal devido ao seu alto volume de registros e à grande frequência de inserções realizadas diariamente pelas filiais de vendas. Cada unidade gera suas próprias notas fiscais, tornando mais eficiente armazená-las localmente. Essa estratégia reduz significativamente o tráfego de dados nos links de menor largura de banda, especialmente nos sites conectados por satélite. Além disso, evita o alto custo de processamento e replicação que seria necessário caso todas as notas fiscais fossem mantidas sincronizadas em todos os locais.

# Questão 3 – Acesso Distribuído entre Bancos PostgreSQL

## a) Conceitos de dblink e postgres_fdw

O **dblink** é uma extensão do PostgreSQL que permite executar consultas em um banco de dados remoto por meio de uma conexão direta. As consultas são realizadas através de funções específicas e o resultado é retornado para o banco local. Embora seja simples de configurar, exige que o desenvolvedor escreva consultas mais complexas e trate explicitamente a conexão remota.

O **postgres_fdw** (Foreign Data Wrapper) é uma solução mais moderna que permite acessar tabelas remotas como se fossem tabelas locais. Ele cria tabelas estrangeiras (*foreign tables*) no banco local, possibilitando o uso de comandos SQL normais, incluindo JOINs, filtros e consultas otimizadas pelo planejador de execução do PostgreSQL. Dessa forma, oferece melhor desempenho e maior transparência para o usuário.

---

## b) Passo a passo para acessar a tabela nota_fiscal de Seropédica a partir de Porto Velho

### 1. Habilitar a extensão postgres_fdw

```sql
CREATE EXTENSION IF NOT EXISTS postgres_fdw;
```

### 2. Criar o servidor remoto

```sql
CREATE SERVER servidor_seropedica
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (
    host 'ip_seropedica',
    dbname 'geek_commerce',
    port '5432'
);
```

### 3. Criar o mapeamento de usuário

```sql
CREATE USER MAPPING FOR CURRENT_USER
SERVER servidor_seropedica
OPTIONS (
    user 'usuario_remoto',
    password 'senha_remota'
);
```

### 4. Criar um schema para as tabelas remotas

```sql
CREATE SCHEMA remoto;
```

### 5. Importar a tabela nota_fiscal

```sql
IMPORT FOREIGN SCHEMA public
LIMIT TO (nota_fiscal)
FROM SERVER servidor_seropedica
INTO remoto;
```

### 6. Realizar consultas como se a tabela fosse local

```sql
SELECT *
FROM remoto.nota_fiscal;
```

### 7. Exemplo de JOIN entre produto (local) e nota_fiscal (remota)

```sql
SELECT
    p.id_produto,
    p.nome,
    nf.id_nota,
    nf.valor_total
FROM produto p
INNER JOIN remoto.nota_fiscal nf
    ON p.id_produto = nf.id_produto;
```

---

## c) Método recomendado para consultas frequentes

Para consultas frequentes, a melhor opção é o **postgres_fdw**. Como existe uma diferença significativa entre os links de rede (2 MB em Porto Velho e 512 kb em Seropédica), é importante minimizar a quantidade de dados trafegados. O postgres_fdw permite que o PostgreSQL otimize as consultas, enviando filtros e condições para serem executados diretamente no servidor remoto (*push-down de consultas*), reduzindo o volume de dados transmitidos pela rede. Além disso, ele oferece maior transparência, facilidade de manutenção e melhor desempenho quando comparado ao dblink, tornando-se a solução mais adequada para o ambiente distribuído da GEEK Commerce.

# Questão 4 – Controle de Concorrência no PostgreSQL

## a) Principais modos de bloqueio de tabela

### ACCESS SHARE

É o bloqueio mais leve do PostgreSQL e é adquirido automaticamente durante operações de leitura (SELECT). Permite que múltiplas consultas leiam a tabela simultaneamente, bloqueando apenas operações que exijam exclusividade total sobre a tabela.

### ROW SHARE

É utilizado em comandos como SELECT ... FOR UPDATE ou SELECT ... FOR SHARE. Permite a leitura dos dados enquanto reserva determinadas linhas para futuras atualizações, evitando conflitos entre transações concorrentes.

### ROW EXCLUSIVE

É adquirido automaticamente por operações que modificam dados, como INSERT, UPDATE e DELETE. Permite que outras transações realizem leituras na tabela, mas controla conflitos com outras operações de escrita.

### SHARE

É utilizado por operações que precisam ler a tabela garantindo que sua estrutura não seja alterada, como a criação de alguns índices. Permite consultas de leitura, mas restringe determinadas operações de modificação.

### ACCESS EXCLUSIVE

É o bloqueio mais restritivo do PostgreSQL. Nenhuma outra transação pode ler ou modificar a tabela enquanto esse bloqueio estiver ativo. É utilizado em operações como DROP TABLE, TRUNCATE e algumas alterações estruturais realizadas com ALTER TABLE.

---

## b) Diferença entre bloqueio de linha e bloqueio de tabela

O bloqueio de linha afeta apenas os registros específicos que estão sendo manipulados por uma transação. Dessa forma, outros usuários podem continuar acessando e modificando linhas diferentes da mesma tabela, aumentando a concorrência e o desempenho do sistema. Já o bloqueio de tabela afeta a tabela inteira, impedindo ou restringindo que outras transações realizem determinadas operações sobre qualquer registro da tabela. Em ambientes com muitos usuários simultâneos, o bloqueio de linha é geralmente mais eficiente por permitir maior paralelismo, enquanto o bloqueio de tabela é utilizado quando é necessário garantir consistência em operações que afetam toda a estrutura ou grande parte dos dados.

# Questão 5 – Deadlock no PostgreSQL

## a) O que é um deadlock em banco de dados?

Um deadlock ocorre quando duas ou mais transações ficam bloqueadas indefinidamente, esperando que a outra libere um recurso. Por exemplo, a Transação A bloqueia a Tabela 1 e tenta acessar a Tabela 2, enquanto a Transação B bloqueia a Tabela 2 e tenta acessar a Tabela 1. Como ambas aguardam a liberação do recurso pela outra transação, nenhuma consegue continuar sua execução.

---

## b) Como o PostgreSQL detecta e resolve um deadlock?

O PostgreSQL monitora constantemente as dependências entre transações que estão aguardando bloqueios. Quando identifica um ciclo de espera entre duas ou mais transações, caracteriza a situação como um deadlock. Para resolver o problema, o sistema escolhe uma das transações envolvidas como vítima, cancela sua execução e realiza um rollback das alterações ainda não confirmadas. Após a liberação dos recursos bloqueados, as demais transações podem continuar normalmente. O erro retornado ao usuário é geralmente **"deadlock detected"**.

---

## c) Boa prática para evitar deadlocks

Uma das principais boas práticas é garantir que todas as transações acessem tabelas e registros sempre na mesma ordem. Por exemplo, se uma aplicação precisa acessar as tabelas **produto** e **nota_fiscal**, todas as transações devem primeiro acessar **produto** e depois **nota_fiscal**. Essa padronização reduz significativamente a possibilidade de ciclos de espera entre transações e, consequentemente, a ocorrência de deadlocks.

# Questão 6 – Uso do EXPLAIN no PostgreSQL

## a) O que é o comando EXPLAIN e qual sua finalidade?

O comando **EXPLAIN** é utilizado para exibir o plano de execução que o PostgreSQL pretende utilizar para executar uma consulta SQL. Sua principal finalidade é auxiliar na análise de desempenho e na otimização de consultas, permitindo identificar como o banco acessa os dados, quais métodos de varredura são utilizados (Seq Scan, Index Scan, Hash Join, entre outros) e qual o custo estimado de cada operação. Com essas informações, o administrador pode detectar gargalos e criar índices ou reescrever consultas para melhorar a performance.

---

## b) Interpretação do plano de execução

### Plano apresentado

```sql
QUERY PLAN
->  Hash Join  (cost=1.14..114.20 rows=1000 width=32)
      Hash Cond: (vendas.produto_id = produto.id)
      ->  Seq Scan on vendas  (cost=0.00..45.00 rows=3000 width=16)
      ->  Hash  (cost=1.10..1.10 rows=10 width=16)
            ->  Seq Scan on produto  (cost=0.00..1.10 rows=10 width=16)
                  Filter: (categoria = 'Eletrônicos')
```

### Passo a passo da execução

1. O PostgreSQL inicia realizando um **Seq Scan** (varredura sequencial) na tabela **produto**.

2. Durante essa leitura, aplica o filtro:

```sql
categoria = 'Eletrônicos'
```

Selecionando apenas os produtos pertencentes à categoria "Eletrônicos".

3. Os registros encontrados são armazenados em uma estrutura de memória chamada **Hash**, criando uma tabela hash baseada no campo **produto.id**.

4. Em seguida, o PostgreSQL executa um **Seq Scan** na tabela **vendas**, percorrendo aproximadamente 3.000 registros.

5. Para cada registro da tabela **vendas**, o sistema consulta a tabela hash criada anteriormente e verifica a condição:

```sql
vendas.produto_id = produto.id
```

6. Quando encontra correspondência entre os valores, realiza o **Hash Join**, combinando os dados das tabelas **vendas** e **produto**.

7. O resultado final é retornado com uma estimativa de aproximadamente 1.000 linhas.

### Análise do plano

O PostgreSQL escolheu o **Hash Join** porque a tabela **produto**, após a aplicação do filtro, possui poucos registros (cerca de 10 linhas), tornando eficiente a criação da tabela hash em memória. Já a tabela **vendas** é percorrida integralmente através de um **Seq Scan**, possivelmente porque não existe um índice adequado ou porque o otimizador considerou a leitura completa mais vantajosa para a quantidade de dados existente.

