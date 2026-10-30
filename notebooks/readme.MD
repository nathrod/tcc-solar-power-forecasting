### `00_data_preparation_integration.ipynb`

1. **Análise inicial dos dados meteorológicos do INMET**
   - Carrega e organiza os dados da estação de Dracena.
   - Investiga falhas e valores ausentes nas variáveis meteorológicas.
   - Analisa possíveis anomalias de temperatura por meio de variações horárias.
   - Verifica a consistência física da radiação solar e seu comportamento ao longo do dia.

2. **Validação e integração da radiação solar**
   - Carrega os dados de radiação GHI provenientes do CAMS.
   - Padroniza timestamps e unidades para permitir a comparação com os dados do INMET.
   - Compara a radiação observada em Dracena com a radiação do CAMS.
   - Avalia a aderência entre as fontes por meio de métricas estatísticas.
   - Utiliza os dados do CAMS como fonte de radiação solar para a série consolidada.

3. **Comparação com estações meteorológicas auxiliares**
   - Compara os dados de Dracena com as estações de Valparaíso e Presidente Prudente.
   - Avalia temperatura, umidade relativa, pressão atmosférica, vento e precipitação entre as estações.
   - Calcula as distâncias entre as estações e os respectivos pesos pelo método de ponderação pelo inverso da distância (IDW).
   - Utiliza a comparação para definir as fontes auxiliares empregadas nos períodos com indisponibilidade da estação de Dracena.

4. **Preenchimento do período de outubro a dezembro**
   - Utiliza IDW entre Valparaíso e Presidente Prudente para temperatura, umidade relativa e velocidade do vento.
   - Substitui os valores de pressão atmosférica pela série da estação de Valparaíso.
   - Integra os valores obtidos à série principal de Dracena.

5. **Obtenção e preparação da precipitação**
   - Carrega os dados de precipitação total provenientes do ERA5.
   - Seleciona o ponto de grade mais próximo das coordenadas de Dracena.
   - Mantém o período correspondente ao ano de 2025.
   - Converte a precipitação de metros para milímetros.
   - Organiza a série em resolução horária e timestamps UTC.

6. **Construção do dataset meteorológico consolidado**
   - Integra temperatura, umidade relativa, pressão atmosférica, radiação solar e precipitação.
   - Padroniza as séries pela coluna `data_hora_utc`.
   - Verifica período, valores ausentes e duplicidades.
   - Exporta o dataset meteorológico consolidado utilizado nas etapas seguintes do trabalho.

### `01_target_dataset_construction.ipynb`

1. **Padronização temporal dos dados do ONS**
   - Carrega os dados de geração horária.
   - Converte o horário da geração de `America/Sao_Paulo` para UTC.
   - Ajusta o recorte temporal para manter o ano de 2025 completo.
   - Realiza o mesmo processo para os dados de constrained-off, originalmente disponibilizados em intervalos de 30 minutos.

2. **Auditoria dos dados meteorológicos**
   - Verifica integridade temporal, timestamps duplicados, frequência horária e período.
   - Analisa valores ausentes, blocos consecutivos de dados ausentes e simultaneidade das lacunas entre variáveis.
   - Documenta os tratamentos realizados para umidade, temperatura e pressão atmosférica.
   - Remove a velocidade do vento devido à elevada quantidade de dados ausentes.

3. **Tratamento dos dados de constrained-off**
   - Identifica a ocorrência de restrição a partir da variável `val_geracaolimitada`.
   - Converte os patamares de 30 minutos para resolução horária.
   - Calcula a geração de referência horária pela média dos dois patamares de 30 minutos.

4. **Integração ONS + dados meteorológicos e construção das bases**
   - `df_sem_restricao`: exclui as horas em que houve constrained-off e mantém a geração efetivamente observada.
   - `df_com_restricao`: mantém as 8.760 horas de 2025 e, nas horas em que houve constrained-off, utiliza a geração de referência do ONS como variável-alvo.
   - Cria a variável `alvo_referencia_ons` para identificar as observações em que o alvo foi substituído pela geração de referência do ONS.