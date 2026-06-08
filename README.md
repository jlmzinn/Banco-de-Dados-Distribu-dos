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
