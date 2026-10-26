# APRAU Group 17 — Report Source Materials

This editorial archive preserves the original detailed Markdown, including numerical details, references, attribution, and methodological qualifications for the report. It is not a completed report or an independent validation. Original wording and language are retained. No models were rerun during this revision.

---

<!-- Original notebook cell 1 (one-based) -->

# APRAU Project — Group 17

## 1. Objectives and Organisation

This project analyses a dataset of 2,000 patients and 35 columns with two objectives: predicting the continuous target `target_regression` and classifying patients as DI, HT_HY, or DI_HT_HY using `target_class`. Neither target is used to predict the other.

The analysis includes:

- Exploratory data analysis (EDA) and preprocessing
- Simple and multiple linear regression
- Logistic regression and LDA
- Resampling and class weighting
- Regularisation and feature selection
- Test-set evaluation and discussion of limitations

---

<!-- Original notebook cell 7 (one-based) -->

## 2. Exploratory Data Analysis (EDA)

### 2.1 Data Inspection and Quality

---

<!-- Original notebook cell 12 (one-based) -->

- `calcium_baseline` apresenta o mesmo valor em todas as observações, sendo uma feature constante e potencialmente não informativa.

- `plasma_value` apresenta um valor distinto para cada observação. Isto pode resultar da precisão da medição e não significa necessariamente que seja um identificador. A sua relevância será investigada posteriormente.

---

<!-- Original notebook cell 14 (one-based) -->

### 2.2 Descriptive Statistics

---

<!-- Original notebook cell 17 (one-based) -->

`systolic_index` exactly matches `systolic_bp` and is therefore redundant.

`ldl_alternative` has the same sorted values as `ldl`, but they are assigned to different patients, with almost no correlation between the columns. This is consistent with a permutation but does not establish how the variable was constructed. It is not a patient-level copy; its contribution will be assessed through associations with the targets and model comparisons.

---

<!-- Original notebook cell 19 (one-based) -->

- No unexpected categories or spelling inconsistencies were identified.

- Most categorical features are relatively balanced. The main exceptions are `age_level`, where `Young` is less frequent, and `bmi_level`, where `Obese` is the most frequent category.

- `target_class` is highly imbalanced: `DI` accounts for 90.05% of cases, compared with 5.25% for `HT_HY` and 4.70% for `DI_HT_HY`. Accuracy should therefore not be used alone to assess classification performance.

---

<!-- Original notebook cell 22 (one-based) -->

- `age_level` categorises `age` into the observed ranges 9–23 (`Young`), 24–45 (`Middle`), 46–67 (`Mature`), and 68–89 (`Senior`). These are dataset-specific ranges, not universal clinical categories.

- `bmi_level` categorises `bmi` using thresholds consistent with the Underweight, Normal, Overweight, and Obese categories provided in the exercise.

- These numerical and categorical variables represent related information differently: numerical values retain exact measurements, whereas categories group values into intervals and discard within-group differences. Their joint use will be evaluated during modelling.

---

<!-- Original notebook cell 25 (one-based) -->

- Patients with `low_hdl_cholesterol = Yes` have lower mean HDL values than the remaining patients. However, the groups overlap, so the indicator does not correspond to a perfect separation at a single HDL threshold.

- Patients with `high_ldl_cholesterol = Yes` have higher mean LDL values, although the groups also overlap.

- These indicators are closely related to `hdl` and `ldl`, respectively, but are not exact copies. Their potential redundancy will be assessed during modelling.

---

<!-- Original notebook cell 26 (one-based) -->

### 2.2.1 Blood Pressure and Lipid Consistency

These checks assess consistency between related measurements. Auxiliary variables are kept separately and are not used as model features.

Pulse pressure is the difference between systolic and diastolic pressure. Mean arterial pressure is approximated by:

$$
\mathrm{MAP} \approx \frac{\mathrm{systolic} + 2\,\mathrm{diastolic}}{3}
$$

The dataset description does not confirm whether `blood_pressure` represents this quantity or comes from the same measurement.

Non-HDL cholesterol is calculated as `cholesterol - hdl`. With comparable units and measurements, total cholesterol is expected to be at least HDL + LDL, allowing for analytical uncertainty. The Friedewald approximation, in mg/dL, is:

$$
\mathrm{LDL} \approx \mathrm{total\ cholesterol} - \mathrm{HDL} - \frac{\mathrm{triglycerides}}{5}
$$

The classical formula is unsuitable for triglycerides ≥ 400 mg/dL. As the dataset's units and measurement methods are unknown, these checks identify potential inconsistencies rather than establish rules for correcting or removing observations.

---

<!-- Original notebook cell 28 (one-based) -->

**Consistency Check Results**

- Systolic pressure is below diastolic pressure in 114 observations and equal in seven. Confirming errors requires knowing whether both refer to the same measurement.
- `blood_pressure` correlates weakly with estimated mean arterial pressure (−0.021); its definition remains unclear.
- Total cholesterol is below HDL + LDL in 469 observations, including 51 where it is also below LDL. These counts overlap.
- Observed LDL and the Friedewald estimate are almost uncorrelated (0.003).

These alerts do not establish measurement errors, as variable definitions, units, and methods are incomplete. No values are reconstructed or patients excluded: all models retain 1,600 training and 400 test patients.

Auxiliary variables are used only for EDA. Including them alongside their original variables in linear regression would introduce linear dependencies.

---

<!-- Original notebook cell 29 (one-based) -->

### 2.3 Univariate Analysis

---

<!-- Original notebook cell 32 (one-based) -->

- Most numerical variables are relatively symmetric. `bmi` and `target_regression` show greater right skew, with some high values.

- `HbA1c_level`, `blood_pressure`, `crp_level`, and `homocysteine_level` show a central concentration with smaller groups of distant values. This pattern alone does not establish errors; these values are retained for modelling.

- `age_normalized` has a similar distribution to `age`. `systolic_index` and `ldl_alternative` reproduce the marginal distributions of `systolic_bp` and `ldl`, respectively.

- Most categorical features are relatively balanced, with exceptions including `age_level`, `bmi_level`, and `low_hdl_cholesterol`. The target `target_class` is highly imbalanced, with `DI` strongly predominant.

---

<!-- Original notebook cell 33 (one-based) -->

### 2.4 Bivariate Analysis — Regression

---

<!-- Original notebook cell 35 (one-based) -->

- Linear correlations with `target_regression` are weak. The highest are `bmi` (0.110), `triglycerides` (0.074), and `HbA1c_level` (0.066).

- `age` and `age_normalized` have identical target correlations, consistent with their equivalent information on different scales. The same holds for `systolic_bp` and its exact copy, `systolic_index`.

- `ldl_alternative` has almost zero linear correlation with `target_regression`; this does not establish its relevance for classification.

- The correlation for `calcium_baseline` is `NaN` because a constant variable has zero variance, making Pearson correlation undefined.

- These results suggest limited predictive performance from individual linear predictors, which will be assessed using simple linear regression.

---

<!-- Original notebook cell 38 (one-based) -->

- `target_regression` distributions overlap substantially across all categorical features examined.

- Median differences across `age_level`, `bmi_level`, `physical_activity`, and `employment_status` are small relative to within-group variability.

- Extreme target values near 260, 280, and 300 occur across several categories rather than exclusively within one group.

- No categorical feature shows a strong individual association with the target visually. Their predictive contribution will be assessed through modelling; these plots alone do not establish irrelevance.

---

<!-- Original notebook cell 41 (one-based) -->

- A classe `HT_HY` apresenta valores de `target_regression` geralmente inferiores, com média de 125,12 e mediana de 126.

- As classes `DI` e `DI_HT_HY` apresentam a mesma mediana, igual a 142. A classe `DI_HT_HY` tem uma média ligeiramente superior, mas também apresenta maior dispersão.

- Apesar destas diferenças, existe uma sobreposição considerável entre as distribuições das três classes. Assim, `target_regression` não permite separar claramente os diagnósticos.

- As classes têm dimensões muito diferentes, pelo que as estatísticas das classes minoritárias são estimadas com menos observações e devem ser interpretadas com maior cautela.

- Esta análise serve para compreender a relação entre os targets. `target_class` não será utilizada como predictor do modelo de regressão.

---

<!-- Original notebook cell 42 (one-based) -->

### 2.5 Análise bivariada — Classificação

---

<!-- Original notebook cell 44 (one-based) -->

- `age` apresenta valores geralmente mais elevados nas classes `HT_HY` e `DI_HT_HY` do que na classe `DI`. Como esperado, `age_normalized` reproduz o mesmo padrão.

- `HbA1c_level` apresenta uma diferença clara: a classe `HT_HY` possui valores geralmente inferiores, enquanto `DI` e `DI_HT_HY` apresentam valores mais elevados e semelhantes.

- `triglycerides` apresenta valores geralmente superiores nas classes `HT_HY` e `DI_HT_HY`.

- Em várias features, como `systolic_bp`, `diastolic_bp`, `alcohol_intake`, `salt_intake`, `heart_rate`, `hdl` e `ldl`, as medianas são semelhantes, mas a classe `DI` apresenta uma dispersão muito superior.

- `plasma_value` e `ldl_alternative` apresentam distribuições muito semelhantes entre as classes, sugerindo reduzida capacidade discriminativa individual.

- `systolic_index` reproduz o comportamento de `systolic_bp`, como esperado devido à redundância já identificada.

- As diferenças de dispersão entre as classes poderão ser relevantes para a escolha do classificador, dado que a LDA assume uma matriz de covariância comum. Esta hipótese será avaliada durante a modelação.

---

<!-- Original notebook cell 45 (one-based) -->

- Em várias features, os valores de algumas classes estão fortemente concentrados num intervalo reduzido. Como consequência, o intervalo interquartil é pequeno e valores moderadamente afastados são apresentados como outliers.

- Estes valores não serão removidos automaticamente. A posição num boxplot não confirma erro de medição; os modelos mantêm estes pacientes, dadas as definições incompletas das variáveis.

---

<!-- Original notebook cell 47 (one-based) -->

- A classe `DI` é predominante em todas as categorias analisadas, confirmando o forte desbalanceamento de `target_class`.

---

<!-- Original notebook cell 48 (one-based) -->

## 3. Protocolo de avaliação e preparação dos dados

Mantemos a divisão original 80/20, com seed 42 e sem estratificação: 1 600 pacientes de treino e 400 de teste. No treino existem 1 443 casos DI, 89 HT_HY e 68 DI_HT_HY; no teste, 358, 16 e 26, respetivamente. Não alteramos a divisão após observar os resultados. O holdout interno e os folds de classificação são estratificados; a estratificação preserva proporções, mas não equilibra as classes.

As regressões são comparadas nos mesmos cinco folds, usando a RMSE média como critério principal. Na classificação, usamos CV5 estratificada com três repetições (15 avaliações) para comparar configurações. Holdout, CV10, LOOCV e bootstrap complementam a avaliação e a discussão de variabilidade, sem escolher o protocolo pelo resultado mais favorável. Não é criado um terceiro conjunto fixo: as validações internas utilizam partes do treino.

A função `prepare_features` aprende categorias e escala apenas no conjunto de ajuste e aplica a transformação à avaliação. Regressão linear sem penalização usa `scaling='none'`; as referências e experiências manuais da classificação usam `numeric`; a comparação de regularização usa `all`, incluindo dummies. Dentro de cada comparação controlada, a preparação é a mesma. Os efeitos de experiências com preparações diferentes não são atribuídos exclusivamente a uma alteração de features ou penalização.

A EDA global, a consulta histórica do teste e a escolha sucessiva de hipóteses no mesmo treino limitam a independência da avaliação. Os melhores scores de CV serviram para seleção e podem ser otimistas; não são uma avaliação independente do procedimento completo.


---

<!-- Original notebook cell 50 (one-based) -->

### Auditoria de coerência no treino

Verificamos pressões invertidas e incompatibilidades entre idade/IMC e as categorias fornecidas. A célula seguinte identifica 84 casos de pressões invertidas no treino e não encontra exclusões adicionais pelas regras de idade/IMC. Todos os 84 casos assinalados pertencem a DI.

A célula constrói `coherent_train_indices`, com 1 516 pacientes, como registo auxiliar da auditoria. Esse índice **não é usado nos modelos apresentados**: a regressão, a classificação e a seleção mantêm `train_indices`, com 1 600 pacientes. O teste mantém os seus 400 casos.

Não removemos pacientes apenas por aparecerem como outliers, por correlações fracas ou pelos alertas lipídicos e de pressão agregada cuja interpretação permanece incerta. Uma eventual exclusão exigiria uma justificação sobre a qualidade dos dados, e não apenas uma melhoria da métrica.


---

<!-- Original notebook cell 52 (one-based) -->

### Inspeção automática de redundâncias no treino

Listamos os pares de features numéricas com correlação de Pearson absoluta ≥ 0,90. O cálculo usa apenas as 1 600 observações de treino e exclui ambos os targets, antes das exclusões de features. Uma coluna constante terá correlação indefinida e não aparecerá nesta tabela.

Esta é uma sinalização para investigar, não uma regra de remoção automática. As verificações anteriores de igualdade e de valores ordenados são mantidas: correlação elevada não prova que duas colunas sejam cópias, e uma permutação de valores pode não ter correlação elevada.


---

<!-- Original notebook cell 54 (one-based) -->

### 3.1 Features e preparação comum

Retiramos os dois targets de X e três colunas justificadas pela EDA: `calcium_baseline` (constante), `systolic_index` (cópia de `systolic_bp`) e `age_normalized` (outra escala da idade). Ficam 30 features originais, que nesta execução originam 40 colunas após codificação. Não confundimos features originais com dummies individuais.

`ldl_alternative`, categorias de idade/IMC e indicadores lipídicos permanecem inicialmente candidatos. As comparações posteriores avaliam a sua contribuição; uma correlação fraca com um target não é, isoladamente, uma regra de exclusão.

A codificação utiliza as categorias presentes no treino de cada ajuste e omite a primeira como referência. Categorias desconhecidas na avaliação ficam com todas as dummies dessa variável a zero, partilhando a representação da categoria de referência. É uma limitação explícita desta convenção, não uma categoria nova aprendida pelo modelo. O alinhamento de colunas e as verificações numéricas asseguram entradas consistentes entre ajuste e avaliação.


---

<!-- Original notebook cell 56 (one-based) -->

## 4. Modelos de regressão

### 4.1 Regressão linear simples

Ajustamos uma regressão linear por feature numérica, usando as mesmas divisões de CV5 nos 1 600 pacientes de treino. Cada modelo utiliza apenas uma variável explicativa. A referência prevê a média do target calculada no treino de cada fold.

Reportamos as médias de R², MAE e RMSE e o desvio-padrão da RMSE entre folds. A variável é escolhida pela menor RMSE média; o teste original não participa nesta escolha. Não aplicamos padronização à regressão linear sem penalização. A dispersão entre folds é descritiva, não um intervalo de confiança.


---

<!-- Original notebook cell 58 (one-based) -->

A variável `bmi` apresentou a menor RMSE média na validação cruzada
de 5 folds, sendo selecionada para a regressão linear simples.

Contudo, a melhoria face à referência da média do treino foi reduzida:
a RMSE passou de 46,1036 para 46,0110. As variáveis `triglycerides`,
`age` e `HbA1c_level` apresentaram resultados próximos.

O R² médio de `bmi` foi ligeiramente negativo (−0,0025), indicando
capacidade preditiva linear individual muito limitada nesta avaliação.
A pequena melhoria de RMSE face à baseline pode coexistir com R² médio
negativo, porque as métricas são calculadas por fold e depois agregadas.

Embora `ldl` apresente o menor MAE médio, mantemos a RMSE como critério
de seleção previamente definido.

Estes resultados não justificam excluir as restantes variáveis:
a sua contribuição em conjunto será avaliada na regressão múltipla.

---

<!-- Original notebook cell 59 (one-based) -->

### 4.2 Regressão linear múltipla

Ajustamos uma regressão linear múltipla com todas as features
disponíveis após as exclusões justificadas na EDA.

As variáveis categóricas são codificadas dentro de cada fold.
Não aplicamos padronização, pois o modelo não usa penalização.

Utilizamos as mesmas divisões de CV5 da regressão simples e avaliamos
R², MAE e RMSE. Este modelo serve como referência para as experiências
de exclusão de variáveis da secção seguinte.

---

<!-- Original notebook cell 61 (one-based) -->

A regressão múltipla apresentou uma RMSE média de 45,4885,
inferior à regressão simples com `bmi` (46,0110) e à baseline
(46,1036).

O R² médio passou para 0,0207, embora a capacidade preditiva
continue reduzida.

O MAE aumentou para 36,2222, face a 35,7264 na regressão simples
e 35,6718 na baseline. Assim, a melhoria não é comum a todas
as métricas: a RMSE penaliza mais os erros grandes, enquanto
o MAE mede o erro absoluto médio.

Mantemos a RMSE como critério de comparação previamente definido.
Na secção seguinte, avaliamos se conjuntos reduzidos de variáveis
permitem manter ou melhorar o desempenho do modelo múltiplo.

---

<!-- Original notebook cell 62 (one-based) -->

### 4.3 Comparação de conjuntos de variáveis

Comparamos a regressão múltipla completa com exclusões individuais de `age_level`, `bmi_level`, `low_hdl_cholesterol`, `high_ldl_cholesterol`, `ldl_alternative` e `plasma_value`. As quatro primeiras representam informação relacionada com medidas numéricas; as duas últimas apresentam indícios de reduzido sinal individual na EDA. Relações entre variáveis motivam candidatos, mas não provam que devam ser eliminados.

Mantemos os mesmos pacientes, folds, modelo e preparação. A diferença de RMSE é calculada relativamente ao conjunto completo: valores negativos favorecem a exclusão. Observamos também as diferenças por fold para não interpretar apenas a média.

Após as experiências individuais, avaliamos a exclusão conjunta de `low_hdl_cholesterol` e `plasma_value`, e dessas duas juntamente com `ldl_alternative`. Esta segunda etapa depende dos resultados preliminares na mesma CV. Os efeitos das exclusões não são necessariamente aditivos; a avaliação é exploratória e não utiliza o teste para escolher os conjuntos.


---

<!-- Original notebook cell 69 (one-based) -->

#### Resultados das exclusões na regressão

A exclusão individual de `low_hdl_cholesterol` e `plasma_value` reduziu a RMSE nos cinco folds, embora com diferenças pequenas. Retirar `ldl_alternative` apresentou uma melhoria média menor e não consistente entre folds. As exclusões individuais de `age_level`, `bmi_level` e `high_ldl_cholesterol` aumentaram a RMSE média; essas colunas foram mantidas.

As duas exclusões conjuntas reduziram a RMSE nos cinco folds face ao conjunto completo. O conjunto sem `low_hdl_cholesterol`, `plasma_value` e `ldl_alternative` apresentou a menor RMSE média entre os candidatos: 45,3691, face a 45,4885 na referência, uma melhoria aproximada de 0,26%.

Retirar adicionalmente `ldl_alternative` melhorou apenas dois dos cinco folds face à exclusão das outras duas variáveis. A vantagem adicional é pequena e inconsistente. Selecionamos o conjunto sem as três pela RMSE média, sem demonstrar que as colunas sejam universalmente dispensáveis.

As combinações foram definidas após observar as exclusões individuais na mesma CV, tornando esta seleção exploratória. As conclusões aplicam-se à regressão; a classificação é estudada separadamente.


---

<!-- Original notebook cell 70 (one-based) -->

### 4.4 Avaliação no conjunto de teste

Reajustamos os modelos selecionados nas 1 600 observações de treino
e avaliamos as suas previsões nos mesmos 400 pacientes de teste.

Comparamos a referência da média do treino, a regressão simples
com a variável escolhida por CV5 e a regressão múltipla sem
`low_hdl_cholesterol`, `plasma_value` e `ldl_alternative`.

A codificação das categorias é aprendida apenas no treino.
Esta avaliação não é utilizada para alterar as escolhas anteriores.

O conjunto de teste já foi consultado em versões anteriores
do projeto, pelo que os resultados são exploratórios e não
constituem uma validação final independente.

---

<!-- Original notebook cell 73 (one-based) -->

### 4.5 Discussão dos resultados

No conjunto de teste, ambos os modelos melhoraram as três métricas
face à referência que prevê a média do treino.

A regressão múltipla apresentou a menor RMSE (47,7733) e o maior
R² (0,0570), sendo a opção preferida pelo critério de RMSE definido
anteriormente. A redução de RMSE face à baseline foi de cerca de 3%.

A regressão simples com `bmi` apresentou o menor MAE (37,1257),
ligeiramente inferior ao da múltipla (37,2831). Assim, a múltipla
não foi superior em todas as métricas.

Apesar da melhoria, o R² da regressão múltipla permanece baixo,
indicando capacidade limitada para explicar a variação do target
nestes pacientes.

As exclusões de variáveis foram escolhidas pela validação cruzada
no treino. O resultado de teste não demonstra, por si só, que cada
variável retirada era dispensável, pois não inclui uma comparação
direta com o modelo múltiplo completo.

Estes resultados são exploratórios, dado que o conjunto de teste
já tinha sido consultado em versões anteriores do projeto.

---

<!-- Original notebook cell 74 (one-based) -->

## 5. Modelos de classificação

Pretendemos prever `target_class` através de regressão logística
e análise discriminante linear (LDA).

Como as classes estão fortemente desequilibradas, usamos o F1 macro
como critério principal, acompanhado de accuracy e precision,
recall e F1 por classe.

### 5.1 Comparação inicial por validação cruzada

Comparamos os dois modelos usando validação cruzada estratificada
de 5 folds, com três repetições. Os modelos usam exatamente
as mesmas divisões dos dados de treino.

Incluímos uma referência que prevê sempre a classe mais frequente
no treino de cada fold.

Usamos todas as features disponíveis após as exclusões iniciais
da EDA. A codificação e a padronização das variáveis numéricas
são aprendidas apenas no treino de cada fold.

Nesta comparação não aplicamos pesos de classes nem fazemos
novas exclusões de variáveis. A regressão logística usa
a penalização L2 predefinida, com C=1.

---

<!-- Original notebook cell 78 (one-based) -->

A regressão logística apresentou o maior F1 macro médio (0,5675),
ligeiramente superior ao da LDA (0,5658). A diferença é pequena,
pelo que a logística fica como referência provisória, sem evidência
de superioridade robusta.

Ambos os modelos reconhecem bem a classe maioritária DI,
com precision próxima de 94% e recall superior a 97%.

Para HT_HY, o recall médio foi igual nos dois modelos (64,38%),
mas a logística apresentou maior precision: 79,93%, face a
74,63% na LDA.

A maior dificuldade foi DI_HT_HY. A logística reconheceu apenas
1,90% dos casos, com precision média de 13,33%. A LDA apresentou
maior recall (3,41%), mas menor precision (10,29%). Assim, ambos
identificam poucos casos desta classe e muitas das suas previsões
positivas estão erradas.

A baseline obteve accuracy de 90,19% por prever sempre DI,
sem reconhecer nenhuma das classes minoritárias. Isto demonstra
por que a accuracy não deve ser interpretada isoladamente.

As métricas apresentadas são médias das 15 avaliações de CV.
Os valores de precision igual a zero nos folds sem previsões
de uma classe seguem a opção `zero_division=0`.

---

<!-- Original notebook cell 79 (one-based) -->

### 5.2 Holdout interno

Dividimos o conjunto de treino em 80% para ajuste e 20% para
validação, preservando aproximadamente as proporções das classes.

Comparamos a baseline, a regressão logística e a LDA na mesma
divisão, mantendo as features e a preparação da secção anterior.

O conjunto de teste original de 400 pacientes não participa.
Como usamos uma única divisão, não estimamos um desvio-padrão
para este método.

---

<!-- Original notebook cell 83 (one-based) -->

No holdout interno, a regressão logística apresentou o maior
F1 macro (0,5871), seguida da LDA (0,5075) e da baseline (0,3158).

Para HT_HY, a logística apresentou precision de 1,0000 e recall
de 0,6667: todas as previsões desta classe estavam corretas,
mas cerca de um terço dos casos reais não foi identificado.
A LDA apresentou precision de 0,8000 e recall de 0,4444.

Ambos os modelos tiveram recall e F1 iguais a zero para DI_HT_HY,
não reconhecendo corretamente nenhum caso dessa classe.

Apesar da accuracy elevada, persistem dificuldades nas classes
minoritárias. Estes resultados dependem de uma única divisão
de 320 pacientes para validação e não demonstram superioridade
geral da logística.

Não apresentamos desvio-padrão para o holdout, pois foi realizada
apenas uma divisão.

---

<!-- Original notebook cell 84 (one-based) -->

### 5.3 Validação cruzada com 10 folds

Avaliamos os mesmos modelos por validação cruzada estratificada
de 10 folds, com três repetições.

Mantemos as features, os parâmetros e a preparação utilizados
na CV5. Em cada fold, a codificação e a padronização são
aprendidas apenas na parte de treino.

Comparamos as médias e a dispersão das métricas com a CV5.
Os desvios-padrão descrevem a variabilidade entre avaliações,
que partilham observações e não são independentes.

---

<!-- Original notebook cell 88 (one-based) -->

Os valores médios de F1 macro foram próximos entre CV5 e CV10.
Na CV5, a logística apresentou o maior valor; na CV10, a LDA
ficou ligeiramente à frente. Estas diferenças pequenas não
demonstram superioridade robusta de um dos modelos.

A CV10 apresentou maior desvio-padrão do F1 macro e da accuracy
nos dois modelos. Cada fold de validação contém 160 pacientes,
face a 320 na CV5, incluindo menos casos das classes minoritárias.
Isso pode contribuir para uma maior oscilação das métricas.

Esta dispersão descreve os scores entre folds; não corresponde
diretamente à variância da estimativa média de desempenho.
Não concluímos que a CV10 tenha sempre maior variância.

O reconhecimento de DI_HT_HY continuou muito reduzido em ambos
os métodos. Alterar o número de folds não resolveu essa dificuldade.

Mantemos a CV5 como critério de seleção previamente definido.
A CV10 é utilizada como análise complementar de estabilidade.

---

<!-- Original notebook cell 89 (one-based) -->

### 5.4 Leave-One-Out Cross-Validation (LOOCV)

Em cada iteração, ajustamos os modelos com 1 599 pacientes
e prevemos a classe do único paciente deixado de fora.
Repetimos o procedimento para os 1 600 pacientes do treino.

A codificação e a padronização são aprendidas apenas nos dados
de ajuste de cada iteração.

Reunimos as previsões fora do treino e calculamos as métricas
nesse conjunto. Não calculamos F1 macro por fold, pois cada fold
de avaliação contém apenas um paciente.

O conjunto de teste original não participa nesta avaliação.

---

<!-- Original notebook cell 93 (one-based) -->

No LOOCV, a regressão logística apresentou accuracy de 0,9300
e F1 macro de 0,5758, face a 0,9250 e 0,5669 na LDA.

Para HT_HY, a logística apresentou maior precision (0,8310),
enquanto a LDA teve recall ligeiramente superior: 0,6742,
face a 0,6629 na logística.

Para DI_HT_HY, ambos os modelos reconheceram apenas um dos
68 casos, correspondendo a recall de 0,0147. A precision
também foi reduzida: 0,1429 na logística e 0,0909 na LDA.

Os resultados reforçam a dificuldade de reconhecimento desta
classe, apesar da accuracy global elevada.

As métricas foram calculadas sobre as 1 600 previsões reunidas,
cada uma obtida sem utilizar o respetivo paciente no ajuste.
Não apresentamos desvio-padrão de F1 macro por fold, porque
cada fold contém apenas uma observação.

---

<!-- Original notebook cell 94 (one-based) -->

### 5.5 Bootstrap com avaliação out-of-bag

Em cada iteração, sorteamos 1 600 observações de treino com
reposição. Alguns pacientes aparecem várias vezes e outros
não são selecionados. Ajustamos os modelos na amostra sorteada
e avaliamos apenas nos pacientes não selecionados (out-of-bag).

Os modelos usam as mesmas amostras em cada iteração.
A codificação e a padronização são aprendidas apenas na
amostra de ajuste.

Apresentamos as médias das métricas e a sua dispersão entre
iterações. Este procedimento avalia os modelos; não é uma
técnica de equilíbrio das classes. O teste original não participa.

---

<!-- Original notebook cell 98 (one-based) -->

Nas 200 iterações de bootstrap, a LDA apresentou F1 macro médio
de 0,5743, ligeiramente superior ao da logística (0,5725).
A diferença é pequena e não demonstra superioridade robusta.

A logística apresentou maior accuracy média (0,9250 contra
0,9182) e melhores precision e recall para HT_HY.

Para DI_HT_HY, a LDA apresentou maior recall médio:
0,0747, face a 0,0399 na logística. Contudo, ambos continuam
a reconhecer uma proporção reduzida dos casos desta classe.
A precision média foi próxima de 0,22 nos dois modelos.

O desvio-padrão do F1 macro foi 0,0327 na LDA e 0,0298
na logística. Estes valores descrevem a dispersão entre
amostras bootstrap, que partilham pacientes.

Os resultados não são diretamente equivalentes aos da CV:
o bootstrap contém pacientes repetidos no ajuste e um número
variável de pacientes distintos na avaliação out-of-bag.

---

<!-- Original notebook cell 99 (one-based) -->

### 5.6 Comparação dos métodos de reamostragem e discussão da variabilidade

Comparamos os modelos de referência sem pesos e com todas as features disponíveis após as exclusões iniciais da EDA. As experiências seguintes de exclusão e pesos não fazem parte desta comparação. Todos os métodos usam apenas os 1 600 pacientes de treino; o teste original permanece fora destes procedimentos.

A tabela resume os resultados guardados nesta execução. Em CV5, CV10 e bootstrap, apresentamos a média e o desvio-padrão (DP) dos scores. No holdout, apresentamos o resultado de uma divisão; no LOOCV, as métricas são calculadas nas 1 600 previsões reunidas.

| Método | Modelo | Accuracy | DP accuracy | F1 macro | DP F1 macro | Recall DI_HT_HY |
|---|---|---:|---:|---:|---:|---:|
| Holdout interno | Logística | 0,9281 | — | 0,5871 | — | 0,0000 |
| Holdout interno | LDA | 0,9062 | — | 0,5075 | — | 0,0000 |
| CV5, 3 repetições | Logística | 0,9277 | 0,0067 | 0,5675 | 0,0283 | 0,0190 |
| CV5, 3 repetições | LDA | 0,9210 | 0,0079 | 0,5658 | 0,0309 | 0,0341 |
| CV10, 3 repetições | Logística | 0,9294 | 0,0095 | 0,5680 | 0,0482 | 0,0143 |
| CV10, 3 repetições | LDA | 0,9244 | 0,0136 | 0,5703 | 0,0616 | 0,0286 |
| LOOCV | Logística | 0,9300 | — | 0,5758 | — | 0,0147 |
| LOOCV | LDA | 0,9250 | — | 0,5669 | — | 0,0147 |
| Bootstrap OOB, 200 iterações | Logística | 0,9250 | 0,0091 | 0,5725 | 0,0298 | 0,0399 |
| Bootstrap OOB, 200 iterações | LDA | 0,9182 | 0,0098 | 0,5743 | 0,0327 | 0,0747 |

Os relatórios anteriores apresentam também precision e recall das restantes classes. A baseline mantém F1 macro próximo de 0,316 e recall zero nas classes minoritárias, apesar da accuracy próxima de 90%. Os dois classificadores superam essa referência em F1 macro, mas o reconhecimento de DI_HT_HY permanece reduzido em todos os métodos.

#### Efeito do método na variabilidade observada

**Holdout.** O ajuste utiliza 1 280 pacientes e a validação 320. O resultado depende de uma única divisão. Não há repetições que permitam estimar a dispersão entre divisões; o traço na tabela significa que não foi estimada, não que a variância seja zero. A diferença de F1 macro entre logística e LDA é maior neste holdout do que nas avaliações por CV, ilustrando a sensibilidade à divisão considerada.

**CV5 e CV10.** As médias de F1 macro são próximas, mas a CV10 apresenta maior dispersão dos scores nesta experiência. Na logística, o DP aumenta de 0,0283 para 0,0482; na LDA, de 0,0309 para 0,0616. A variância descritiva entre scores é o quadrado do DP: aproximadamente 0,000801 e 0,002323 na logística, e 0,000955 e 0,003795 na LDA, respetivamente, usando os valores arredondados da tabela.

Cada avaliação da CV10 contém 160 pacientes, face a 320 na CV5, incluindo menos casos minoritários; isso pode contribuir para maior oscilação de precision, recall e F1. Em contrapartida, a CV10 ajusta cada modelo com 1 440 pacientes, face a 1 280 na CV5. Estes resultados não demonstram que a CV10 tenha sempre maior variância nem que a CV5 seja universalmente melhor.

**LOOCV.** Cada modelo é ajustado com 1 599 pacientes, mas os conjuntos de ajuste diferem em apenas uma observação. As previsões e os modelos resultantes são, por isso, fortemente dependentes. Usar quase todo o treino em cada ajuste não garante menor variância da estimativa de desempenho. Como cada avaliação individual contém um paciente, calculamos precision, recall e F1 sobre as previsões reunidas; não estimamos um DP de F1 macro por fold, nem atribuímos variância zero ao método.

**Bootstrap OOB.** Cada ajuste usa 1 600 posições sorteadas com reposição, com aproximadamente 63,2% de pacientes distintos em média. Os restantes, cerca de 36,8%, são avaliados out-of-bag; o número varia por iteração. O DP do F1 macro foi 0,0298 na logística e 0,0327 na LDA, próximo do observado em CV5 e inferior ao de CV10 nesta execução. Contudo, os tamanhos efetivos de ajuste e avaliação diferem dos da CV e as iterações partilham pacientes. Não concluímos que o bootstrap seja, em geral, mais estável ou tenha menor variância.

#### Conclusão da comparação

Os desvios-padrão e as variâncias entre scores descrevem a dispersão das avaliações realizadas; não são diretamente a variância do estimador médio de desempenho nem intervalos de confiança. Folds, repetições e amostras bootstrap partilham dados, pelo que não podemos tratá-los como experiências independentes ou calcular a incerteza da média simplesmente dividindo o DP pela raiz do número de avaliações.

A logística apresenta F1 macro ligeiramente superior na CV5 e no LOOCV, enquanto a LDA fica ligeiramente à frente na CV10 e no bootstrap. As diferenças são pequenas e não demonstram superioridade geral de um modelo. O maior recall de DI_HT_HY no bootstrap também não prova que o método de reamostragem melhore o classificador final: são avaliações com composições e tamanhos distintos.

Mantemos a CV5 estratificada com três repetições como protocolo previamente adotado para comparar configurações nas experiências seguintes. Esta decisão assegura consistência e um custo computacional viável, sem afirmar que a CV5 é o melhor método em todos os problemas. O baixo reconhecimento das classes minoritárias motiva as experiências separadas de exclusão, pesos e regularização.

Procedimentos de referência: materiais da semana 4, em particular `T4_EvaluationResampling.pdf`, `TP4_Cross_validation_solutions.ipynb` e `Bootstrap_by_Hand.ipynb`. A discussão numérica refere-se à execução atual e deve ser revista se os resultados forem recalculados ou alterados.

---

<!-- Original notebook cell 100 (one-based) -->

### 5.7 Comparação de conjuntos de variáveis na classificação

Investigamos se as três variáveis retiradas na regressão também
podem ser dispensadas na classificação.

Comparamos todas as features disponíveis com a exclusão individual
de `low_hdl_cholesterol`, `plasma_value` e `ldl_alternative`,
e com a exclusão das três em conjunto.

Mantemos logística e LDA sem pesos, com os mesmos parâmetros,
preparação e divisões de CV5 estratificada com três repetições.
A codificação e a padronização são aprendidas dentro de cada fold.

O critério principal é o F1 macro, acompanhado de precision
e recall por classe. O teste original não participa.
As candidatas foram motivadas por análises anteriores neste
dataset, pelo que a comparação é exploratória.

---

<!-- Original notebook cell 104 (one-based) -->

A exclusão conjunta de `low_hdl_cholesterol`, `plasma_value` e `ldl_alternative` apresentou o maior F1 macro médio entre os conjuntos avaliados nos dois classificadores: 0,5792 na LDA e 0,5761 na logística, face a 0,5658 e 0,5675 nas referências completas.

Mantemos esse conjunto para a experiência de pesos e para a referência LDA. A LDA ficou ligeiramente à frente da logística nesta comparação, sem evidência de superioridade robusta. A logística é utilizada na experiência seguinte por disponibilizar diretamente `class_weight`, não por ser comprovadamente superior à LDA. As melhorias são modestas e o recall de DI_HT_HY continua baixo.

Na secção 6 regressamos às 30 features disponíveis após as exclusões iniciais, para comparar regularização e seleção com uma referência completa. Assim, as exclusões manuais não são impostas ao seletor L1. A escolha destes conjuntos na mesma CV permanece exploratória.


---

<!-- Original notebook cell 105 (one-based) -->

### 5.8 Pesos de classes na regressão logística

Comparamos a regressão logística sem pesos com a opção
`class_weight='balanced'`, que atribui pesos inversamente
proporcionais às frequências das classes no treino de cada fold.

Usamos o conjunto sem `low_hdl_cholesterol`, `plasma_value`
e `ldl_alternative`, mantendo os mesmos parâmetros, preparação
e divisões de CV5 com três repetições.

Avaliamos se os pesos melhoram o F1 macro e o reconhecimento
das classes minoritárias, acompanhando a precision para identificar
um eventual aumento dos falsos positivos.

Esta comparação é exploratória, pois o conjunto de features
foi selecionado anteriormente com os mesmos dados de treino.

---

<!-- Original notebook cell 109 (one-based) -->

#### Pesos baseados nas frequências: compensação parcial e normalização

As experiências anteriores com pesos manuais motivaram uma abordagem sistemática baseada nas frequências das classes. Esta extensão foi definida após análises preliminares e é exploratória; não corresponde a uma fórmula exigida no enunciado.

A opção `balanced` atribui a cada classe o peso:

$$
w_c^{balanced}=\frac{N}{K n_c},
$$

onde $N$ é o número de pacientes no treino, $K$ o número de classes e $n_c$ a frequência da classe. Nos 1 600 pacientes de treino, existem 1 443 casos de DI, 89 de HT_HY e 68 de DI_HT_HY. Os pesos seriam aproximadamente 0,370; 5,993; 7,843, respetivamente, com proporções relativas de 1:16,21:21,22. Estes cálculos ilustram o treino completo; na CV, as frequências são recalculadas apenas na parte de ajuste de cada fold.

Para explorar compensações menos intensas, definimos pesos relativos:

$$
u_c(\alpha)=\left(\frac{n_{DI}}{n_c}\right)^\alpha.
$$

Com $\alpha=0$, todas as classes têm peso 1. Com $\alpha=1$, as proporções são inversamente proporcionais às frequências. Valores intermédios suavizam a compensação; não preservam exatamente as proporções de balanced. Testamos $\alpha\in\{0,25;0,50;0,75;1,00\}$, além da referência sem pesos e de `balanced`. Neste contexto, alpha controla os pesos e não é o parâmetro de penalização de Ridge/Lasso.

**Normalização dos pesos.** Calculamos a média dos pesos por paciente e dividimos cada peso por essa média:

$$
\bar{u}(\alpha)=\frac{\sum_c n_c u_c(\alpha)}{N},
\qquad
w_c(\alpha)=\frac{u_c(\alpha)}{\bar{u}(\alpha)}.
$$

Assim, $\sum_c n_c w_c=N$: o peso médio por paciente é 1. A normalização preserva as proporções entre classes para cada alpha, mantendo constante o peso total no ajuste. Com `C=1` fixo, evita-se que uma multiplicação global dos pesos altere também a força relativa da regularização. Para $\alpha=1$, os pesos normalizados coincidem com `balanced`.

Mantemos as features selecionadas anteriormente, a escala apenas nas variáveis numéricas e os mesmos 15 folds da CV5 estratificada com três repetições. Categorias, escala, frequências e pesos são aprendidos apenas no treino de cada fold. O teste original não participa na escolha.

Comparamos precision e recall de todas as classes e usamos o F1 macro médio como critério de seleção entre os valores desta grelha. Não são impostos custos clínicos nem mínimos de desempenho validados. A utilização da mesma CV para selecionar features e pesos pode tornar os resultados otimistas.

---

<!-- Original notebook cell 113 (one-based) -->

#### Resultados e escolha da intensidade de compensação

Na execução apresentada, `alpha=1.00` e `balanced` produziram as mesmas métricas, como esperado pela equivalência dos pesos normalizados.

Entre os valores avaliados, **alpha=0,50 apresentou o maior F1 macro médio (0,6048)**, face a 0,5761 sem pesos. O recall de HT_HY aumentou de 0,6586 para 0,7743 e o de DI_HT_HY de 0,0242 para 0,1938. O recall de DI diminuiu de 0,9898 para 0,9381. As precisions foram 0,9536 para DI, 0,6035 para HT_HY e 0,2000 para DI_HT_HY.

A compensação superior com alpha=0,75 elevou o recall de DI_HT_HY para 0,4289, mas reduziu a sua precision para 0,1479 e o recall de DI para 0,8443. Com balanced, o recall de DI_HT_HY atingiu 0,4883, mas a precision caiu para 0,0845 e o recall de DI para 0,7015. Assim, aumentar a compensação não melhorou todas as métricas simultaneamente.

Selecionamos **alpha=0,50** para as experiências seguintes pelo maior F1 macro médio dentro desta grelha e pelo compromisso observado entre reconhecimento minoritário e preservação de DI. A escolha não significa superioridade em todas as métricas, nem demonstra vantagem robusta face às experiências anteriores com pesos manuais. A formulação baseada nas frequências fornece uma regra reproduzível; por si só, não prova melhor desempenho.

**Limitações.** O desvio-padrão do F1 macro foi elevado (0,0652) e os folds partilham observações. A precision de 0,2000 e o recall de 0,1938 para DI_HT_HY mostram que persistem muitos falsos positivos e casos não identificados. A pesquisa foi desenvolvida após experiências preliminares e a mesma CV serviu para seleção; não é uma avaliação independente nem uma validação clínica. Não foi demonstrado um ótimo global de alpha.

Nas próximas avaliações internas, alpha fica fixo em 0,50, mas os valores numéricos dos pesos são recalculados e normalizados no treino de cada fold. Num reajuste final, serão calculados apenas nos 1 600 pacientes de treino, sem utilizar o target do teste.

---

<!-- Original notebook cell 114 (one-based) -->

## 6. Regularização e seleção de features na classificação

### 6.1 Comparação de Ridge e Lasso com C=1

Seguimos a comparação apresentada na parte final de `TP5_Ridge_Lasso_Elasticnet.ipynb` e nas páginas 40–46 de T5: ajustar regressão logística com penalização L2 (Ridge) e L1 (Lasso), fixando **C=1** em ambos os modelos, e comparar desempenho e coeficientes ativos. Menor C significa maior regularização, mas nesta experiência não pesquisamos C. Elastic Net, também ilustrado na TP5, não é incluído por ser complementar ao requisito de Ridge/Lasso.

Usamos as 30 features disponíveis após as exclusões iniciais da EDA, antes das três exclusões manuais adicionais. A referência completa permite investigar a seleção produzida pela regularização.

**Adaptações ao projeto.** Em vez de escolher configurações pelo teste, avaliamos os dois modelos nos mesmos 15 folds da CV5 estratificada com três repetições. Além da accuracy usada no exemplo da TP5, calculamos F1 macro, precision e recall por classe, devido ao forte desequilíbrio. A accuracy de cada avaliação fica guardada em `regularization_fold_results`. Mantemos alpha=0,50 para os pesos de classes, calculados e normalizados apenas no treino de cada fold. Esse alpha controla a compensação de frequências; C controla a penalização.

Padronizamos todas as colunas codificadas, incluindo dummies, dentro de cada fold, para comparar coeficientes em escalas comparáveis. Esta opção é uma adaptação metodológica e difere da escala apenas numérica da experiência anterior; as duas penalizações são comparadas com a mesma preparação nesta secção. Para o problema multiclasse, uma coluna é considerada ativa se pelo menos um coeficiente das três classes tiver módulo superior a 0,0001. O exemplo binário da TP5 inspeciona apenas `coef_[0]`; aqui verificamos todas as classes.

Usamos solver SAGA, como na TP5, com `penalty='l1'` ou `penalty='l2'` explícitos para compatibilidade com a versão instalada. A estabilização numérica divide todos os pesos pelo seu máximo k e multiplica C interno por k, preservando o objetivo correspondente ao **C nominal de 1**, apresentado na tabela. Mantemos tol=0,0001 e max_iter=50000; a execução é interrompida se algum ajuste não convergir.

L1 pode anular coeficientes; L2 encolhe os coeficientes sem eliminar automaticamente colunas. A comparação explícita entre conjuntos completos e reduzidos é apresentada na secção 6.2. Os resultados continuam exploratórios, porque os pesos e outras decisões já foram estudados neste treino.

---

<!-- Original notebook cell 118 (one-based) -->

#### Interpretação da comparação com C=1

Na execução apresentada, Ridge obteve F1 macro médio de 0,6073 e Lasso de 0,6057. A diferença de 0,0016 é pequena e não demonstra superioridade robusta. O recall de DI_HT_HY foi 0,2381 e 0,2282, respetivamente, mantendo-se dificuldades nessa classe.

Ridge manteve as 40 colunas ativas. Lasso manteve, em média, 39,13: C=1 produziu pouca simplificação. Estas contagens referem-se a colunas após codificação, e não necessariamente a features originais completas. Os desvios-padrão do F1 macro (0,0666 e 0,0635) descrevem dispersão entre avaliações dependentes e não são intervalos de confiança. 


---

<!-- Original notebook cell 119 (one-based) -->

### 6.2 Comparação com conjuntos reduzidos

Comparamos a logística L2 completa com L2 reajustada em dois conjuntos reduzidos. Mantemos C nominal igual a 1, os mesmos pesos com alpha=0,50, a mesma padronização e os mesmos folds. Incluímos também L1 completa com C=1 como referência da secção anterior. Assim, a comparação de L2 completa com L2 reduzida altera apenas as colunas utilizadas, incluindo o procedimento que as seleciona.

**Seleção L1.** Ajustamos um seletor logístico L1 com C=0,1 apenas no treino de cada fold. Conservamos uma coluna quando pelo menos um coeficiente das três classes tem módulo superior a 0,0001. Depois ajustamos uma nova logística L2 com C=1 nessas colunas. O C=0,1 do seletor é distinto do C=1 do modelo avaliado. Foi escolhido como experiência de redução mais forte após a pesquisa preliminar; não é uma regra fornecida pela TP5 nem uma escolha independente dos dados já explorados.

**Seleção baseada em L2.** Ridge não anula coeficientes automaticamente. Acrescentamos uma regra heurística explícita: ajustar L2 com C=1 no treino do fold, calcular a norma dos coeficientes de cada coluna entre as três classes e conservar as 50% de colunas com maior norma. Reajustamos L2 com C=1 no conjunto reduzido. A fração de 50% é uma escolha exploratória para estudar redução, não uma propriedade de Ridge nem um ótimo demonstrado.

As categorias, escalas, frequências, pesos e seleções são aprendidos apenas no treino de cada fold. Não usamos uma seleção feita nos 1 600 pacientes para avaliar esses mesmos pacientes por CV. Caso L1 não selecione colunas, avaliamos um modelo só com intercepto, representado por uma entrada constante e contabilizado com zero predictors.

Guardamos os nomes das colunas selecionadas em cada fold. A seleção de uma dummy não implica conservar todas as categorias da variável original. Reportamos a dimensão média e os seus limites entre folds, precision e recall por classe, F1 macro e diferenças emparelhadas face à L2 completa. O teste original continua fora desta comparação. Esta etapa adapta os princípios de coeficientes esparsos da TP5 ao requisito de comparar conjuntos completos e reduzidos; não é uma transcrição do exemplo das aulas.


---

<!-- Original notebook cell 123 (one-based) -->

### 6.3 Discussão da seleção de features

O modelo L2 completo apresentou F1 macro médio de 0,6073. O reajuste L2 nas colunas selecionadas por L1 atingiu 0,6163, com 25,6 colunas de entrada em média (entre 24 e 30), face a 40 na referência. A diferença emparelhada média foi +0,0091 e houve melhoria em oito dos 15 folds. O desvio-padrão do F1 macro foi 0,0590, face a 0,0666 na L2 completa.

No conjunto selecionado por L1, precision/recall foram 0,9560/0,9307 para DI, 0,6129/0,7861 para HT_HY e 0,2088/0,2432 para DI_HT_HY. A redução não prejudicou o recall minoritário médio nesta comparação, embora a classe DI_HT_HY continue mal reconhecida.

O conjunto top50% de L2 utilizou 20 colunas e obteve F1 macro de 0,5982, diferença média de −0,0091 face à referência, com melhoria em cinco dos 15 folds. O recall de DI_HT_HY diminuiu para 0,1788. A maior simplificação teve, portanto, um custo observado no reconhecimento dessa classe.

Selecionamos o procedimento L1 seguido de reajuste L2 como candidato principal entre estes quatro procedimentos, pelo maior F1 macro médio e pela redução de dimensão. A melhoria não é consistente em todos os folds e não demonstra superioridade estatística. A contagem de folds favoráveis é descritiva, pois os folds e as repetições partilham observações; o DP não é um intervalo de confiança.

L1 pode selecionar colunas diferentes entre amostras, sobretudo na presença de predictors relacionados. Uma coluna anulada não é clinicamente irrelevante em absoluto. A regra top50% é uma seleção externa a Ridge, não uma propriedade de eliminação automática de L2. Uma diferença média pequena também não demonstra equivalência entre modelos.

O C do seletor e os pesos resultam de exploração anterior deste treino. A CV não estima independentemente todo o procedimento de seleção. No reajuste da secção 7, a lista final de colunas é reaprendida nos 1 600 pacientes; não usamos a união dos folds nem o fold com maior score, e não escolhemos pelas métricas de teste.


---

<!-- Original notebook cell 124 (one-based) -->

## 7. Avaliação dos classificadores no conjunto de teste

Após as comparações no treino, reajustamos os procedimentos escolhidos nos 1 600 pacientes e avaliamos nos 400 pacientes de teste originais. A tabela de teste não é utilizada para alterar features, pesos, C ou regras de seleção.

Comparamos a baseline da classe mais frequente, a LDA com o conjunto reduzido manualmente, a logística L2 completa e a logística L2 com seleção L1. A LDA é uma referência do percurso anterior, não uma comparação controlada do efeito da regularização. As duas logísticas usam C nominal de 1, padronização de todas as colunas e os mesmos pesos normalizados com alpha=0,50, calculados exclusivamente no treino.

Para o candidato selecionado por L1, reajustamos o seletor com C=0,1 apenas no treino completo. Conservamos as colunas com algum coeficiente de classe de módulo superior a 0,0001 e ajustamos L2 com C=1 nesse conjunto. A seleção final não é a união dos folds nem a seleção do fold com melhor resultado. Apresentamos os pesos, as colunas conservadas, métricas por classe e matrizes de confusão.

O teste já foi observado em versões anteriores e a EDA utilizou o dataset completo. Estes resultados são exploratórios, não uma validação final independente nem uma demonstração de adequação clínica.


---

<!-- Original notebook cell 129 (one-based) -->

### 7.1 Interpretação da avaliação no teste

O teste contém 358 pacientes DI, 16 HT_HY e 26 DI_HT_HY. A baseline acerta os 358 pacientes DI, obtendo accuracy de 0,8950, mas não identifica nenhum caso minoritário. O seu F1 macro é 0,3149.

A LDA com redução manual apresentou a maior accuracy (0,9150), mas F1 macro de 0,5859. Reconheceu 354/358 pacientes DI, 11/16 HT_HY e apenas 1/26 DI_HT_HY. A elevada accuracy deve ser lida em conjunto com esse fraco reconhecimento da classe mais difícil.

A logística L2 com seleção L1 apresentou accuracy de 0,8750 e F1 macro de 0,5996. Reconheceu 334/358 casos DI (recall 0,9330), 12/16 HT_HY (0,7500) e 4/26 DI_HT_HY (0,1538). Para DI_HT_HY, quatro das 24 previsões positivas estavam corretas: precision de 0,1667. Vinte dos 26 casos reais dessa classe foram previstos como DI e dois como HT_HY.

**Efeito da redução de colunas.** A L2 completa teve F1 macro de 0,5981 e accuracy de 0,8725, com os mesmos acertos minoritários. A L2 reduzida acrescentou apenas um acerto em DI. A diferença de F1 macro de 0,0015 é mínima; a vantagem concreta é reduzir de 40 para 26 colunas (35%) com resultados próximos neste teste. Não foi demonstrada melhoria relevante nem equivalência estatística.

O seletor final conservou `low_hdl_cholesterol_Yes`, mas excluiu `plasma_value` e `ldl_alternative`, entre outras colunas. A seleção condicionada por penalização e restantes predictors não tem de coincidir com a exclusão manual. Remover uma dummy não implica retirar a variável original completa nem demonstrar irrelevância clínica.

Nas matrizes, as linhas são classes reais e as colunas classes previstas. A diagonal conta acertos; as restantes células identificam confusões. A comparação controlada da seleção é entre as duas logísticas, com pesos, escala e C iguais. A comparação com LDA altera também o modelo, a preparação e a utilização de pesos; não isola o efeito destes fatores.

O desempenho do candidato selecionado foi inferior ao seu F1 macro médio de CV (0,6163). A CV foi usada para selecionar hipóteses e pode ser otimista; o teste tem ainda poucos casos minoritários. Um único acerto altera o recall de HT_HY em 6,25 pontos percentuais e o de DI_HT_HY em cerca de 3,85 pontos. Não alteramos a configuração após observar estes resultados. A consulta histórica do teste mantém esta avaliação exploratória.


---

<!-- Original notebook cell 130 (one-based) -->

## 8. Conclusões integradas e limitações

### 8.1 Principais resultados

**Dados e preparação.** A auditoria não identificou valores em falta nem linhas totalmente duplicadas. Retirámos uma coluna constante e duas representações redundantes, mantendo 30 features candidatas. Os alertas de pressões e lípidos foram documentados sem impor correções clínicas não confirmadas. Todos os modelos apresentados mantêm os mesmos 1 600 pacientes de treino e 400 de teste.

**Regressão.** BMI foi a melhor variável individual pela RMSE média de CV5, mas a melhoria face à média foi reduzida. A regressão múltipla sem `low_hdl_cholesterol`, `plasma_value` e `ldl_alternative` apresentou RMSE de teste de 47,7733, contra 48,6671 na simples e 49,2433 na baseline. O seu R² foi 0,0570: combinar predictors ajudou modestamente, mas a capacidade preditiva linear continua limitada. A simples apresentou menor MAE do que a múltipla, pelo que não houve superioridade em todas as métricas.

**Classificação e reamostragem.** Logística e LDA superaram a baseline em F1 macro, sem superioridade robusta de um modelo em todos os métodos. Nesta execução, CV10 apresentou maior dispersão entre folds do que CV5; os tamanhos e dependências das avaliações impedem transformar essa observação numa regra universal. Holdout e LOOCV não fornecem aqui um DP comparável de F1 macro. A accuracy isolada ocultou o fraco recall de DI_HT_HY.

**Pesos.** No conjunto reduzido manualmente, alpha=0,50 normalizado aumentou o recall médio de DI_HT_HY em CV de 0,0242 para 0,1938 e o F1 macro de 0,5761 para 0,6048. Compensações mais intensas aumentaram o recall minoritário, com perda de precision e reconhecimento de DI. Selecionámos alpha=0,50 pelo maior F1 macro da grelha e pelo compromisso observado, sem o interpretar como um custo clínico.

**Seleção com regularização.** A seleção L1 seguida de reajuste L2 apresentou o maior F1 macro médio entre os quatro procedimentos finais de seleção (0,6163). No teste, utilizou 26 colunas em vez de 40 e obteve F1 macro de 0,5996, próximo de 0,5981 na L2 completa. O ganho principal foi a simplificação de 35% das colunas codificadas; não houve uma melhoria relevante no teste. Ridge não eliminou colunas automaticamente: a redução top50% foi uma regra adicional explícita.

### 8.2 Modelo escolhido e alcance da conclusão

Mantemos como candidato final de classificação a logística L2 com C nominal de 1, pesos normalizados com alpha=0,50 e seleção L1 com C=0,1, reajustada exclusivamente no treino. A escolha foi feita pela comparação interna, não pela maior accuracy no teste. A LDA permanece uma referência relevante, com maior accuracy, mas apenas um caso DI_HT_HY identificado no teste, face a quatro no candidato.

O candidato não resolve satisfatoriamente DI_HT_HY: no teste, falha 22 dos 26 casos e apresenta 20 falsos positivos dessa classe. A experiência demonstra os compromissos entre reconhecimento minoritário, falsos positivos e simplicidade, não um sistema pronto para decisão clínica.

### 8.3 Limitações e trabalho futuro

- A EDA global e a consulta do teste em versões anteriores impedem tratá-lo como validação independente. Manter a divisão assegura comparabilidade, não elimina essa limitação.
- Features, pesos e regras foram explorados sucessivamente na mesma CV. A seleção por essas médias introduz otimismo; novos dados ou CV aninhada que inclua todas as etapas seriam necessários para uma avaliação mais independente.
- O número reduzido de pacientes minoritários limita a estabilidade das métricas. Desvios-padrão de folds dependentes não são intervalos de confiança nem provas de significância.
- As definições incompletas de algumas medições limitam a interpretação dos alertas de coerência. Não removemos pacientes para obter uma métrica mais favorável.
- Os coeficientes e as seleções dependem das outras variáveis, da escala e da penalização. Não demonstram causalidade nem irrelevância clínica de colunas excluídas.

Uma extensão futura poderá avaliar estabilidade das seleções e procedimentos de avaliação independentes. Não é necessário adicionar algoritmos para ocultar resultados limitados: a interpretação crítica faz parte dos objetivos do trabalho.


---

<!-- Original notebook cell 131 (one-based) -->

## 9. Referências e apoio utilizado

### 9.1 Enunciado e materiais das aulas

- [Enunciado APRAU 2026/2027, v1.0](Trabalho_APRAU_2026_27_step1%20%281%29.pdf): requisitos de EDA, regressão, classificação, reamostragem e seleção de features.
- Semana 1: [Python for Data Analysis Lab](../week1/Python%20for%20Data%20Analysis_Lab.ipynb), [PL1 — Introdução a Python](../week1/PL1_APRAU_Intro_Python_Lab.ipynb) e [TP1 — Machine Learning in Python](../week1/TP1_APRAU_Machine%20Learning%20in%20Python.ipynb): manipulação e exploração dos dados, preparação e conceitos iniciais.
- Semana 2: [PL Aula 2](../week2/PL_Aula2.ipynb), [TP2 — Soluções](../week2/TP2_solutions.ipynb) e [T2 — Regressão linear](../week2/T2_LinearRegression_v0.pdf): regressão simples/múltipla, codificação e avaliação.
- Semana 3: [PL3 — Logística, LDA e QDA](../week3/PL3_LR_LDA_and_QDA.ipynb), [T3 — Classificação](../week3/Te%C3%B3ricas/T3_Classification_RL_LDA.pdf) e [Classification Metrics](../week3/Te%C3%B3ricas/ClassificationMetrics.pdf): classificadores e métricas por classe.
- Semana 4: [T4 — Evaluation and Resampling](../week4/T4_EvaluationResampling.pdf), [aula de reamostragem em Python](../week4/aula_T_Resampling_Python.ipynb), [TP4 — Cross-validation, soluções](../week4/TP4_Cross_validation_solutions.ipynb) e [Bootstrap by Hand](../week4/Bootstrap_by_Hand.ipynb): holdout, CV, LOOCV e avaliação bootstrap out-of-bag.
- Semana 5: [T5 — Ridge e Lasso](../week5/T5_RidgeLasso.pdf), em particular a parte de classificação nas páginas 40–46, e [TP5 — Ridge, Lasso e Elastic Net](../week5/TP5_Ridge_Lasso_Elasticnet.ipynb), em particular a comparação logística final com C=1 e a análise de coeficientes ativos.

### 9.2 Documentação e fontes complementares

- Documentação de `LogisticRegression`, `StandardScaler`, `classification_report`, validadores de CV e `ConfusionMatrixDisplay` do scikit-learn, consultada através dos materiais e da API utilizada. A sintaxe `penalty='l1'/'l2'` foi adaptada à versão instalada após verificar os avisos; a sintaxe da TP5 não foi assumida como universal.
- As fontes de apoio clínico às fórmulas da EDA estão ligadas na secção 2.2.1. Não substituem a confirmação das unidades, medições e regras de construção deste dataset.

### 9.3 Adaptações e apoio de IA

A função de preparação por fold, as comparações de exclusões, a compensação por potência alpha, a normalização dos pesos, a estabilização de SAGA e a regra top50% de L2 são implementações/adaptações específicas deste projeto. Não são apresentadas como transcrições dos notebooks das aulas. A fórmula com alpha e a regra top50% constituem escolhas exploratórias, não requisitos do enunciado nem métodos clinicamente validados.

Foi utilizado apoio de IA (Codex) na elaboração e revisão de código, organização do notebook, verificações e redação/interpretação. O grupo é responsável por compreender, rever e validar as decisões, os resultados e as limitações. Notebooks e registos anteriores no projeto documentam experiências preliminares; a presente versão consolida o percurso apresentado.

A identificação completa dos elementos e contribuições, a partilha do repositório GitHub com o docente e a submissão no formato pedido devem ser confirmadas pelo grupo. A execução deste notebook não comprova esses requisitos administrativos nem a preparação para a defesa individual.

