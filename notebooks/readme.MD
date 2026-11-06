## Notebooks

O pipeline experimental foi organizado em seis notebooks sequenciais, cobrindo desde a preparação dos dados meteorológicos até a avaliação final dos modelos de previsão em múltiplos horizontes.

### `00_meteorological_data_preparation_2025.ipynb`

1. **Preparação inicial dos dados meteorológicos do INMET**

   - Carrega e organiza os dados da estação automática de Dracena.
   - Padroniza nomes de colunas, timestamps e tipos de dados.
   - Investiga falhas e valores ausentes nas variáveis meteorológicas.
   - Verifica a consistência temporal e física das séries.

2. **Validação e integração da radiação solar**

   - Carrega os dados de radiação GHI provenientes do CAMS.
   - Padroniza timestamps e unidades para integração com os dados meteorológicos.
   - Compara a radiação disponível nas diferentes fontes.
   - Utiliza os dados do CAMS como fonte de radiação solar para a base consolidada.

3. **Comparação com estações meteorológicas auxiliares**

   - Utiliza dados das estações de Valparaíso e Presidente Prudente como suporte à reconstrução da série de Dracena.
   - Compara temperatura, umidade relativa, pressão atmosférica e outras variáveis meteorológicas entre as estações.
   - Calcula as distâncias entre Dracena e as estações auxiliares.
   - Calcula pesos utilizando o método de ponderação pelo inverso da distância (IDW).

4. **Reconstrução de períodos com indisponibilidade**

   - Utiliza IDW entre Valparaíso e Presidente Prudente para reconstrução de variáveis selecionadas.
   - Utiliza a estação de Valparaíso como fonte auxiliar para pressão atmosférica quando necessário.
   - Integra os valores reconstruídos à série meteorológica principal.

5. **Preparação da precipitação**

   - Carrega dados de precipitação provenientes do ERA5.
   - Seleciona o ponto de grade correspondente à região de Dracena.
   - Converte a precipitação para milímetros.
   - Organiza a série em resolução horária e timestamps UTC.

6. **Construção da base meteorológica**

   - Integra temperatura do ar, umidade relativa, pressão atmosférica, radiação solar e precipitação.
   - Padroniza todas as fontes pela coluna `data_hora_utc`.
   - Verifica período, duplicidades e consistência temporal.
   - Exporta a base meteorológica utilizada nas etapas seguintes.

---

### `01_missing_data_treatment_2025.ipynb`

1. **Auditoria dos valores ausentes**

   - Analisa a quantidade e distribuição temporal dos valores ausentes.
   - Identifica blocos consecutivos de dados faltantes.
   - Verifica a ocorrência simultânea de lacunas entre diferentes variáveis meteorológicas.

2. **Tratamento das variáveis meteorológicas**

   - Reconstrói valores ausentes de temperatura do ar.
   - Reconstrói valores ausentes de umidade relativa.
   - Reconstrói valores ausentes de pressão atmosférica.
   - Utiliza informações provenientes das estações auxiliares quando necessário.

3. **Avaliação das variáveis**

   - Verifica a consistência dos valores após o preenchimento.
   - Remove a velocidade do vento do conjunto final devido à elevada quantidade de dados ausentes e à baixa confiabilidade de sua reconstrução.

4. **Exportação**

   - Gera o dataset meteorológico tratado de 2025 utilizado na construção da variável-alvo.

---

### `02_target_dataset_construction_2025.ipynb`

1. **Preparação dos dados de geração do ONS**

   - Carrega os dados horários de geração fotovoltaica.
   - Seleciona o Conjunto Fotovoltaico de Dracena.
   - Padroniza os timestamps para UTC.
   - Mantém o período correspondente ao ano de 2025.

2. **Preparação dos dados de constrained-off**

   - Carrega os registros de restrição de geração disponibilizados pelo ONS.
   - Padroniza os timestamps.
   - Identifica os períodos em que houve limitação da geração fotovoltaica.
   - Converte os registros originalmente disponíveis em resolução de 30 minutos para resolução horária.

3. **Construção da geração de referência**

   - Calcula a geração de referência horária a partir dos registros de constrained-off.
   - Utiliza essa informação para evitar que períodos de limitação operacional sejam tratados como redução natural do recurso solar.

4. **Integração com os dados meteorológicos**

   - Integra geração, constrained-off e variáveis meteorológicas pela coluna `data_hora_utc`.
   - Mantém consistência temporal entre todas as fontes.

5. **Construção das bases-alvo**

   - `df_sem_restricao`: remove os períodos em que houve constrained-off.
   - `df_com_restricao`: mantém todas as horas e substitui a geração observada pela geração de referência nos períodos de restrição.
   - Cria a variável `alvo_referencia_ons` para identificar os registros cujo alvo foi corrigido.

---

### `03_exploratory_data_analysis_2025.ipynb`

1. **Análise descritiva**

   - Avalia estatísticas das variáveis meteorológicas e da geração fotovoltaica.
   - Analisa distribuições, amplitude e comportamento temporal.

2. **Análise das relações entre variáveis**

   - Calcula correlações de Pearson e Spearman.
   - Avalia a relação entre radiação solar e geração fotovoltaica.
   - Investiga a associação entre geração e variáveis meteorológicas.

3. **Análise temporal da geração**

   - Avalia o perfil horário da geração.
   - Analisa padrões sazonais.
   - Investiga a autocorrelação da série.

4. **Seleção de defasagens**

   - Analisa a dependência temporal da geração.
   - Dá suporte à escolha das defasagens utilizadas posteriormente na modelagem:
     - `t-1`;
     - `t-2`;
     - `t-3`;
     - `t-24`.

5. **Suporte à engenharia de atributos**

   - Utiliza os resultados exploratórios para definir as variáveis e transformações incorporadas aos experimentos de Machine Learning.

---

### `04_dataset_extension_2026.ipynb`

1. **Extensão da série de geração**

   - Obtém e prepara os dados do ONS para janeiro a maio de 2026.
   - Seleciona o mesmo conjunto fotovoltaico utilizado em 2025.
   - Padroniza timestamps e resolução temporal.

2. **Tratamento de constrained-off**

   - Processa os dados de restrição de geração de 2026.
   - Calcula a geração de referência nos períodos afetados.
   - Constrói a variável-alvo corrigida de forma compatível com a metodologia utilizada em 2025.

3. **Preparação dos dados meteorológicos**

   - Integra dados do INMET para Dracena e estações auxiliares.
   - Utiliza IDW quando necessário.
   - Prepara temperatura, umidade relativa e pressão atmosférica.

4. **Integração da radiação solar**

   - Prepara os dados de GHI provenientes do CAMS.
   - Padroniza os valores para a mesma resolução temporal da geração.

5. **Integração da precipitação**

   - Utiliza os dados do ERA5.
   - Mantém a mesma metodologia empregada na preparação da base de 2025.

6. **Construção da extensão temporal**

   - Consolida geração e variáveis meteorológicas para janeiro a maio de 2026.
   - Prepara os dados para integração com a base de 2025 utilizada na modelagem.

---

### `05_multi_horizon_modeling_and_evaluation.ipynb`

1. **Consolidação da base de modelagem**

   - Integra os dados de 2025 com a extensão de janeiro a maio de 2026.
   - Ordena a série temporal.
   - Verifica duplicidades, continuidade temporal e valores ausentes.

2. **Engenharia de atributos**

   - Cria atributos históricos da geração:
     - `geracao_t`;
     - `geracao_t-1`;
     - `geracao_t-2`;
     - `geracao_t-3`;
     - `geracao_t-24`.
   - Cria atributos de radiação:
     - `radiacao_t`;
     - `radiacao_t-1`;
     - `radiacao_t-2`.
   - Mantém temperatura, umidade relativa, pressão atmosférica e precipitação disponíveis no instante atual.
   - Cria atributos temporais cíclicos com seno e cosseno da hora do dia e do dia do ano.

3. **Construção dos horizontes de previsão**

   - Formula o problema como previsão direta:
     - 1 hora à frente;
     - 12 horas à frente;
     - 24 horas à frente.
   - Utiliza os mesmos instantes-alvo entre os horizontes para garantir comparabilidade.

4. **Definição dos experimentos**

   - **E1:** histórico da geração e atributos temporais.
   - **E2:** E1 acrescido da radiação solar.
   - **E3:** E2 acrescido de temperatura do ar, umidade relativa, pressão atmosférica e precipitação.

5. **Divisão temporal**

   - Treinamento: janeiro a agosto de 2025.
   - Validação: setembro e outubro de 2025.
   - Teste independente: novembro de 2025 a maio de 2026.
   - Mantém a ordem temporal e evita embaralhamento das observações.

6. **Modelos de referência**

   - Persistência simples.
   - Persistência sazonal diária.

7. **Support Vector Regression**

   - Utiliza SVR com kernel RBF.
   - Padroniza os atributos utilizando `StandardScaler`.
   - Realiza seleção de hiperparâmetros utilizando `GridSearchCV`.
   - Utiliza `TimeSeriesSplit` com `gap` ajustado ao horizonte de previsão.

8. **Random Forest**

   - Realiza otimização dos principais hiperparâmetros.
   - Utiliza validação temporal consistente com os demais modelos.

9. **XGBoost**

   - Avalia um modelo de gradient boosting baseado em árvores.
   - Ajusta número de estimadores, profundidade, taxa de aprendizado, `subsample` e `colsample_bytree`.
   - Utiliza o mesmo protocolo temporal aplicado aos demais algoritmos.

10. **Treinamento final**

    - Após a definição dos hiperparâmetros, combina os conjuntos de treinamento e validação.
    - Treina os modelos finais sem utilizar o conjunto de teste.

11. **Avaliação no conjunto de teste**

    - Calcula:
      - MAE;
      - RMSE;
      - R².
    - Compara modelos, conjuntos de atributos e horizontes.

12. **Análise da contribuição das variáveis**

    - Compara E1, E2 e E3.
    - Quantifica o efeito da inclusão de radiação e demais variáveis meteorológicas sobre o erro de previsão.

13. **Análise da influência do horizonte**

    - Compara o desempenho entre 1h, 12h e 24h.
    - Avalia a degradação relativa das métricas conforme a antecedência da previsão aumenta.

14. **Análise de incerteza**

    - Constrói faixas empíricas de erro a partir dos resíduos observados no conjunto de validação.
    - Condiciona os intervalos à magnitude da geração prevista.
    - Avalia largura média, largura mediana e cobertura no conjunto de teste.

15. **Importância das variáveis**

    - Utiliza `permutation_importance` para analisar a contribuição dos atributos no Random Forest.
    - Compara a importância das variáveis entre os diferentes horizontes de previsão.

16. **Visualização dos resultados**

    - Gera gráficos de RMSE, MAE e R².
    - Compara os modelos de Machine Learning com os modelos de persistência.
    - Apresenta geração observada, previsão e faixa empírica de erro.
    - Analisa graficamente a influência do horizonte e dos conjuntos de atributos.
