# Banco-de-Dados-Distribu-dos
Cenário: GEEK Commerce
Uma proposta adequada para a GEEK Commerce, considerando a localização das filiais, a largura de banda limitada dos links via satélite e o perfil de uso de cada tabela, seria:

Tabela	Técnica escolhida
Cliente	Fragmentação Horizontal
Produto	Replicação Total
Sexo	Replicação Total
Nota Fiscal	Fragmentação Horizontal
(alternativamente: Cliente pode usar Fragmentação Híbrida, dependendo do detalhamento exigido)	
Justificativas

Cliente – Fragmentação Horizontal

Os clientes podem ser distribuídos por região (Sudeste, Nordeste, Norte etc.).
Cada filial acessa principalmente os clientes de sua área de atuação.
Reduz o tráfego nos links de baixa velocidade e melhora o desempenho das consultas locais.

Produto – Replicação Total

O catálogo de produtos precisa estar disponível em todas as filiais para consultas e vendas.
As alterações são relativamente menos frequentes do que as consultas.
Garante alta disponibilidade e rapidez de acesso.

Sexo – Replicação Total

É uma tabela pequena e praticamente estática.
O custo de replicação é muito baixo.
Facilita consultas de marketing e relatórios em qualquer unidade.

Nota Fiscal – Fragmentação Horizontal

Cada filial gera suas próprias notas fiscais.
O volume de dados é elevado e cresce constantemente.
Manter as notas fiscais localmente reduz o tráfego de rede, especialmente nos links de 256 kb e 512 kb.
Consultas operacionais são realizadas principalmente pela própria filial que gerou a venda.
Resposta pronta para entrega
Tabela	Técnica escolhida
Cliente	Fragmentação Horizontal
Produto	Replicação Total
Sexo	Replicação Total
Nota Fiscal	Fragmentação Horizontal

Justificativa: As tabelas Produto e Sexo possuem poucos dados e são amplamente consultadas, tornando a replicação total a melhor opção. Já Cliente e Nota Fiscal possuem grande volume de registros e forte vínculo regional, sendo mais eficiente distribuí-las por fragmentação horizontal para reduzir tráfego de rede e melhorar o desempenho das operações locais.
