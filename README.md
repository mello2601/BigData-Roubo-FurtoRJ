# BigData-Roubo-FurtoRJ

Trabalho da disciplina de **Big Data Analytics** sobre a evolução de **roubos e furtos no município do Rio de Janeiro entre 2005 e 2015**, usando os dados abertos do Instituto de Segurança Pública (ISP-RJ).

**Autores:** Matheus Pepe e Rafael Dias

---

## Objetivo

Em vez de só listar estatísticas, o trabalho foi organizado em torno de quatro perguntas:

1. **Os roubos caíram?** Como os roubos evoluíram de 2005 a 2015, e a queda se manteve?
2. **Existe época do ano?** Algum mês concentra mais furtos do que os outros?
3. **Mais crime ou mais gente?** Os furtos cresceram mais do que a população da cidade?
4. **Dá para medir?** Quanto os crimes mudam por mês, e quanto pesa o Carnaval? A regressão linear responde.

## Dados

| Fonte | Conteúdo |
|---|---|
| [ISP-RJ](https://www.ispdados.rj.gov.br/) – `BaseDPEvolucaoMensalCisp.csv` | Ocorrências registradas por delegacia (CISP) e por mês |
| IBGE/SIDRA – tabelas 6579, 793 e 202 | População residente do município (estimativas anuais, Contagem 2007 e Censo 2010) |

Recorte usado: **município do Rio de Janeiro, 2005–2015**, 42 delegacias (CISPs), 132 meses e cerca de 5,3 mil linhas (delegacia × mês). Foram mantidas as colunas de identificação e todas as variáveis de roubo e furto (transeunte, celular, coletivo, veículo, carga, comércio, residência, bicicleta, totais etc.).

## Como o trabalho foi feito

Todo o projeto está em um único notebook do Google Colab ([`code.ipynb`](code.ipynb)), dividido em três etapas.

### Etapa 1 – Pipeline de dados + banco SQLite

- Download da base direto do site do ISP, filtro por município e período, seleção de colunas e limpeza.
- Roubo e furto de **bicicleta só passaram a ser registrados em 2014**; antes disso o valor é tratado como vazio (`NA`/`NULL`), e não como zero, para não distorcer médias e desvios.
- Os dados tratados vão para um banco **SQLite** (`seguranca_rj.db`) com três tabelas:
  - `delegacias` – uma linha por CISP;
  - `ocorrencias` – uma linha por CISP × mês, com as contagens de crimes (a AISP fica aqui porque várias CISPs mudaram de AISP na reorganização de 2011);
  - `populacao` – população do município por ano (IBGE), usada nas taxas.
- Conferência de que o banco tem exatamente os mesmos dados do DataFrame e consultas SQL de exemplo.

### Etapa 2 – Análise descritiva, outliers e regressão

- **Estatística descritiva** (média, mediana, desvio padrão etc.) em dois níveis: delegacia × mês e total mensal do município.
- **Taxa por 100 mil habitantes**, calculada no banco com `JOIN` entre ocorrências e população.
- **Composição** dos crimes (quais tipos pesam mais) e **concentração** (quanto do total está em poucas delegacias).
- **Correlação** de Pearson e Spearman entre os totais mensais.
- **Detecção de outliers** por **IQR** (método principal, pois contagens de crimes são assimétricas) e por **z-score** (comparação). Além do IQR global, também é calculado o IQR dentro de cada delegacia, para achar meses atípicos em relação ao próprio histórico dela. Os outliers foram identificados e mantidos na base.
- **Gráficos**: série mensal, boxplots por ano e histogramas.
- **Regressão linear** sobre os totais mensais do município:
  - Modelo 1 – tendência simples (`y = b0 + b1·t`), em que `b1` é a variação média de ocorrências por mês;
  - Modelo 2 – roubos em três fases (regressão segmentada, com quebras no pico e no vale);
  - Modelo 3 – furtos com tendência curva e variável de Carnaval (o mês do Carnaval muda de ano para ano).

### Etapa 3 – Apresentação

- Todos os resultados são salvos em uma **planilha Excel** única (`projeto_seguranca_rj_2005_2015.xlsx`, uma aba por tabela, com aba "Leia-me" e gráficos) e também gravados no banco SQLite.
- A **apresentação em PowerPoint** é gerada automaticamente pelo próprio código com `python-pptx`: todos os números dos slides vêm das tabelas calculadas, então, se a base mudar, a apresentação acompanha.

## Principais conclusões

- Os **roubos caíram** do pico até o ponto mais baixo do período e **depois voltaram a subir** — uma reta única esconde esse padrão, por isso o modelo em três fases descreve melhor a série.
- Os **furtos têm um pico que se repete todo ano no mês do Carnaval** (inclusive em 2011 e 2014, quando o Carnaval caiu em março).
- Desde meados do período a cidade passou a registrar **mais furtos do que roubos**.
- A **taxa de furtos por habitante cresceu**, ou seja, o aumento vai além do crescimento da população.

## Limitações

- **Registro não é crime:** a base conta boletins de ocorrência. Parte da alta dos furtos pode ser mais gente registrando (seguro, bloqueio de celular), e não necessariamente mais crime.
- **Delegacias não são comparáveis diretamente:** os números por delegacia são absolutos (o Centro, por exemplo, recebe muita gente que não mora lá). Por isso a taxa por habitante foi calculada só para a cidade.
- **Bicicleta só desde 2014:** antes disso o dado é vazio e ficou fora dos cálculos.
- **Regressão não prova causa:** os modelos descrevem o passado; não explicam por que os crimes mudaram nem foram feitos para prever o futuro.

## Como executar

1. Abra o [`code.ipynb`](code.ipynb) no [Google Colab](https://colab.research.google.com/).
2. Execute a célula. A base é baixada direto do site do ISP.
3. Ao final são gerados o banco `seguranca_rj.db`, a planilha `.xlsx` e a apresentação `.pptx`.

> No Colab os arquivos somem quando a sessão reinicia — baixe-os ao final ou salve no Google Drive.

## Ferramentas

Python · pandas · NumPy · SQLite · statsmodels · matplotlib · openpyxl · python-pptx · Google Colab
