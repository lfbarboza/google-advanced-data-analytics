# PACE Strategy Document — Automatidata (NYC TLC)

**Curso 1 — Foundations of Data Science**
**Autor:** Lucas Barboza
**Notebook de referência:** [`automatidata_c1.ipynb`](../notebooks/automatidata_c1.ipynb)

Respostas às perguntas de reflexão do PACE Strategy Document. As respostas das
tasks (Task 1, 2b, 2c e resumo executivo) ficam no notebook; estas, aqui.

## Plan

### Como você pode se preparar melhor para entender e organizar as informações fornecidas?

A preparação começa fora do código: ler o dicionário de dados da NYC TLC e os e-mails dos stakeholders antes do `read_csv()`. Foi isso que permitiu tratar `RatecodeID = 99` como código fora do dicionário oficial — um achado a reportar — e reconhecer que as gorjetas zeradas em dinheiro são limitação do sistema de captura, não erro de preenchimento. Sem a documentação, ambos seriam apenas números.

A segunda parte é definir o critério de leitura antes de olhar os dados. Adotei a regra de nunca reportar um problema como adjetivo — nada de "há valores estranhos" — e sim quantificado: 148 corridas com distância zero, 33 sem passageiros, 14 registros negativos. Isso muda o que se procura: `describe()` deixa de ser uma tabela para admirar e vira um mapa de hipóteses a testar com filtros explícitos. A terceira é validar contra o mundo real — a distância máxima de 33,96 milhas só é plausível porque bate com a geografia da região metropolitana de Nova York — e aceitar que a resposta nem sempre está nas colunas entregues: `trip_duration_min`, derivada do par embarque/desembarque, expôs inconsistências que nenhuma das duas colunas mostrava isoladamente.

### Quais codebooks de acompanhamento e autorrevisão ajudarão nesse trabalho?

O **codebook de acompanhamento** deste projeto é o próprio notebook fornecido pelo curso, já estruturado com as tasks na ordem do PACE. Ele funciona como roteiro: define a sequência carregar → inspecionar (`head`/`info`/`describe`) → investigar variável a variável, e é essa estrutura que impede pular direto para a conclusão. Os labs guiados de Python e pandas do Curso 2 cobrirão a parte mecânica que aqui foi apoiada por orientação recebida durante o processo.

O **codebook de autorrevisão** é o exemplar que o curso disponibiliza logo após a entrega, como o próprio notebook anuncia. A comparação útil não é conferir se o número bate, mas se a abordagem foi equivalente ou se usei um caminho mais frágil — por exemplo, se a conversão com `pd.to_datetime()` e `format` explícito é o padrão esperado, e se derivar a duração via `.dt.total_seconds() / 60` é a forma canônica.

Além desses, as referências que efetivamente sustentaram o trabalho foram o **dicionário de dados da TLC**, os **e-mails dos stakeholders** — que definiram o que investigar a fundo — e o **próprio notebook como registro vivo**, com cada achado em markdown acompanhado do número que o comprova, de modo que a revisão do DeShawn reconstitua o caminho sem me perguntar nada.

### Que atividades adicionais um estudante engenhoso faria antes de começar a codar?

Conferir a expectativa contra a entrega. A documentação descreve 408.294 registros e o arquivo tem 22.699 — um fato estrutural que limita qualquer conclusão e precisa ser comunicado no início, não descoberto no fim. Em seguida, mapear os pedidos dos stakeholders: ler os e-mails da Luana e do DeShawn antes de codar transforma uma inspeção genérica em dirigida, porque já se sabe quais variáveis precisam sustentar o modelo, em vez de espalhar esforço igualmente por 18 colunas.

Estudar o domínio também ajuda, mas com uma ressalva que vale registrar: ele nem sempre desfaz a ambiguidade — às vezes a aumenta. A corrida de $1.200,29 tem `RatecodeID = 5` (tarifa negociada), o que torna um valor alto tecnicamente possível — ainda que $999,99 de tarifa por 2,60 milhas siga desproporcional. Conhecer a composição tarifária não fecha o caso; mostra por que ele precisa ser classificado como **provável erro de lançamento, a validar com a TLC**, em vez de afirmado. Por fim, uma checagem ética antes de começar: dados de corridas são dados de deslocamento de pessoas, e vale saber desde o início o que pode circular externamente.

---

## Analyze

### As informações disponíveis serão suficientes para atingir o objetivo?

Para o objetivo imediato — inspecionar, tipar e identificar variáveis relevantes — sim. São 22.699 corridas com 18 colunas, sem valores nulos, cobrindo os doze meses de 2017, e a intuição sobre as variáveis confirma coerência: mediana de 1,61 milha e $11,80 por corrida são números típicos de deslocamento urbano em Manhattan, não de um dataset corrompido.

Para o objetivo final do cliente — prever tarifa —, não sem ressalvas. A documentação descreve 408.294 registros e o arquivo tem 22.699, sem qualquer informação sobre como a amostra foi extraída; plausível não é o mesmo que representativo. Somam-se três lacunas de validade a resolver antes de modelar: o significado das 148 corridas com distância zero, a origem do `RatecodeID = 99` (ausente do dicionário) e o pareamento dos 14 estornos com as corridas originais. A ausência de nulos não resolve nenhuma delas — completude não é validade.

### Como você construiria estatísticas-resumo do dataframe e avaliaria o intervalo mín./máx. dos dados?

Em três camadas. Para as variáveis contínuas, `df.describe()`, que entrega contagem, média, desvio-padrão e quartis de uma vez. Para as que são rótulo e não quantidade — `payment_type`, `VendorID`, `RatecodeID`, `PULocationID`, `DOLocationID` — a média não significa nada, e o resumo correto é `value_counts()`: foi assim que apareceram as 7.267 corridas em dinheiro e a ausência completa dos códigos 5 e 6 de pagamento, que constam do dicionário mas não ocorrem no arquivo. Para comparar grupos, `groupby()` sobre a categórica com a métrica de interesse.

O par mín./máx. é o teste de sanidade, e classifico cada um em três categorias. **Limites coerentes:** `tolls_amount`, de $0,00 a $19,10, compatível com os pedágios da região. **Limites impossíveis:** `fare_amount` mínimo de -$120,00, `trip_duration_min` mínimo de -16,98 minutos, `passenger_count` mínimo de 0 e `RatecodeID` máximo de 99, que sequer existe no dicionário. **Limites possíveis, mas suspeitos:** `trip_distance` máximo de 33,96 milhas (plausível pela geografia da região), `total_amount` de $1.200,29, `tip_amount` de $200,00 e duração máxima de 1.439,55 minutos — praticamente 24 horas.

Um detalhe do mín./máx. merece destaque porque sustenta a leitura dos estornos: o piso negativo não está só na tarifa. `extra` chega a -$1,00, `mta_tax` a -$0,50 e `improvement_surcharge` a -$0,30 — a reversão atinge a linha inteira, campo a campo, e não apenas o valor principal. Cada item das duas últimas categorias vira um filtro explícito que conta quantos registros existem, em vez de ficar na leitura do extremo isolado.

### As médias de alguma variável parecem incomuns? Você consegue descrever os dados intervalares?

A mais reveladora é a gorjeta média por forma de pagamento: $2,73 no cartão contra **$0,00 exatos** nas 7.267 corridas em dinheiro. Zero absoluto não é comportamento de passageiro, é ausência de captura — o sistema não registra gorjeta em dinheiro. Qualquer modelo que use `tip_amount` sem isolar a forma de pagamento aprende um artefato do sistema, não o comportamento real. Duas outras chamam atenção pelo motivo oposto: a média de `total_amount` é praticamente idêntica entre os vendors ($16,30 contra $16,32), ou seja, não há diferença operacional a explorar; e a gorjeta média por número de passageiros varia pouco, de $2,61 a $2,83, sugerindo pouca influência — mas a própria quebra revela 27 corridas no cartão com zero passageiros registrados, categoria que não deveria existir.

As variáveis contínuas — `trip_distance`, `fare_amount`, `tip_amount`, `tolls_amount`, `total_amount`, `trip_duration_min` e os dois timestamps — compartilham o mesmo formato: todas assimétricas à direita. A comparação entre média e mediana mostra isso de imediato — `total_amount` tem média $16,31 contra mediana $11,80, e `trip_duration_min`, média 17,0 contra mediana 11,2 minutos. Em ambos os casos poucos valores altos puxam a média, e é a mediana que descreve a corrida típica. O caso extremo é a dispersão da duração: desvio-padrão de 62,0 minutos contra mediana de 11,2, ou seja, a variabilidade retrata os outliers, não o comportamento comum.

Vale a distinção de escala ao interpretá-las: valores monetários e distância têm zero absoluto e admitem razão ("o dobro da tarifa" faz sentido), enquanto os timestamps são propriamente intervalares — a diferença entre dois é significativa, a razão entre eles não. A duração derivada deles, porém, volta a ser de razão: 20 minutos é o dobro de 10. É justamente essa mudança de escala que torna `trip_duration_min` mais informativa que as colunas de origem, e que justifica derivá-la em vez de trabalhar com os timestamps brutos.


---

## Execute

### O que recomendaria ao seu gerente investigar mais a fundo antes da EDA?

Em ordem de prioridade: o que bloqueia a modelagem vem antes do que apenas incomoda.

**1. Como a amostra foi extraída.** Esta é diferente das demais — não trata de um registro, trata do dataset inteiro. A documentação descreve 408.294 corridas e recebemos 22.699, sem critério declarado. Se o recorte não for aleatório (um mês, um vendor, uma faixa de valor), nenhuma conclusão daqui generaliza para 2017 e todo o resto da lista muda de alcance. É a primeira pergunta a fazer à TLC.

**2. O que significa distância zero.** São 148 corridas, e a resposta define tratamentos opostos: corrida cancelada com cobrança mínima é registro válido a manter; falha de captura do GPS é dado a descartar ou imputar. Como `trip_distance` é a preditora mais direta da tarifa, decidir isso no escuro contamina o modelo inteiro.

**3. Se os estornos têm par no arquivo.** São 14 registros negativos, todos com `payment_type` 3 ou 4, e com reversão campo a campo. Se cada um anula uma corrida que também está no dataset, o par precisa sair junto do treino; se a corrida original não está, são ruído de sinal invertido. Mais uma vez, dois tratamentos incompatíveis dependendo da resposta.

**4. O que é o `RatecodeID` 99.** Não consta no dicionário oficial. Importa porque o código de tarifa comprovadamente afeta o valor — o 5, de tarifa negociada, aparece de forma desproporcional entre as corridas mais caras —, então uma categoria não documentada é uma variável explicativa que não sabemos ler.

**5. A corrida de $1.200,29.** Deliberadamente por último: é um registro único, e a decisão sobre outliers extremos pode seguir regra estatística sem depender da TLC. Vale perguntar; não vale esperar a resposta para seguir.

### Quais dados se apresentam inicialmente contendo anomalias?

**Impossíveis pela regra do domínio.** 148 corridas com `trip_distance` igual a zero; 26 com duração zero, que são subconjunto exato das anteriores — toda corrida sem duração também está sem distância; uma corrida com duração negativa (-16,98 minutos), em que o desembarque antecede o embarque; 33 corridas com zero passageiros; e o `RatecodeID` 99, ausente do dicionário.

**Possíveis, mas suspeitas.** A corrida de $1.200,29 — tarifa de $999,99 mais $200,00 de gorjeta em 2,60 milhas, sob tarifa negociada; 46 corridas acima de três horas, com máximo de 1.439,55 minutos (praticamente 24 horas); e a concentração de valores altos em corridas com `RatecodeID` 5 e destino na zona 265, não identificada. Nenhuma viola uma regra física, todas destoam do padrão.

**Comportamento do sistema, que se confunde com anomalia.** Os 14 valores negativos não são erro: são estornos coerentes, com `payment_type` 3 ou 4 e reversão de `extra`, `mta_tax` e `improvement_surcharge` até o piso negativo de cada campo. E a gorjeta de $0,00 exatos nas 7.267 corridas em dinheiro não é ausência de gorjeta, é ausência de captura — a anomalia aqui está no instrumento, não no dado.

**Estrutura, não valor.** Duas colunas de data chegaram como texto e cinco variáveis categóricas vieram tipadas como inteiro, o que faz o `describe()` produzir médias sem significado para códigos de tarifa, vendor e zona.

### Que tipos adicionais de dados poderiam fortalecer este dataset?

*Premissa: considero como alvo `fare_amount`, não `total_amount` — este último embute `tip_amount`, justamente o campo com captura cega em dinheiro, e o alvo herdaria o artefato.*

**Contexto da corrida.** O taxímetro registra distância e tempo, mas não o que os explica. A rota efetivamente percorrida (e não apenas origem e destino), o número de paradas e o tempo parado em trânsito separariam uma corrida longa por distância de uma corrida longa por congestionamento — hoje as duas produzem a mesma duração. No mesmo espírito, a tabela de zonas da própria TLC, que traduz `PULocationID` e `DOLocationID` em nome e borough, transformaria dois inteiros opacos em geografia utilizável e daria sentido imediato à zona 265.

**Contexto externo.** Clima por hora (precipitação e temperatura), feriados e grandes eventos, e obras ou interdições de via são condições que alteram demanda e tempo de percurso, existem em bases públicas e se juntam ao dataset pelo timestamp de embarque. São exatamente as variáveis que explicariam a variação que hoje aparece como dispersão inexplicada na duração.

**Metadados de processo.** Esta é a direção que resolveria as perguntas que travaram o exercício inteiro. Um identificador de transação ligando o estorno à corrida original eliminaria a dúvida do pareamento. Um campo de status da corrida — concluída, cancelada, anulada — responderia sozinho o que significa distância zero. Um indicador de qualidade do sinal de GPS separaria falha de captura de corrida que realmente não saiu do lugar. E a versão do equipamento ou do software de registro explicaria códigos como o 99, que provavelmente nasce de uma implementação específica e não de uma regra de negócio.

O caso da gorjeta em dinheiro merece nota à parte, porque a solução não começa com dado novo: começa com representação honesta. Gorjeta não capturada deveria ser nula, não zero — a codificação atual afirma "não houve gorjeta" quando o correto é "não sabemos". Só isso já impediria um modelo de aprender que pagar em dinheiro reduz a gratificação. Para capturar o valor de fato, seria preciso uma fonte onde o dinheiro aparece: a conciliação de fechamento de turno do motorista, comparando a receita declarada com a soma das corridas do período.
