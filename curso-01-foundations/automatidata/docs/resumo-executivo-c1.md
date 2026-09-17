# Resumo executivo — Automatidata (NYC TLC)

**Curso 1 — Foundations of Data Science**
**Autor:** Lucas Barboza
**Destino:** uma página, template *Executive summaries* (retrato 8,5 × 11 pol)
**Layout sugerido:** slide 7 (só Título e Subtítulo, sem caixa de imagem — o que sobra mais espaço para texto). Se quiser um gráfico no rodapé, use o slide 5.
**Público:** DeShawn e Luana (técnicos); Uli King, Juliana Soto (Finanças) e Titus Nelson (Operações) — sem vocabulário técnico.

---

## Título e subtítulo

**Título:** Inspeção da base de corridas de táxi — NYC TLC, 2017
**Subtítulo:** Automatidata · Resumo executivo · o que a base já permite e o que precisa da TLC

---

## 1. O que foi feito

Recebemos a base de corridas de táxi amarelo de 2017 com a documentação oficial da TLC e conferimos uma contra a outra. Mapeamos as 18 informações registradas em cada corrida, verificamos preenchimento, formatos e faixas de valores, e quantificamos cada registro fora do padrão — nenhum problema foi anotado como impressão; todos têm volume medido. Criamos ainda uma medida de duração de cada corrida a partir dos horários de embarque e desembarque, que não existia na base e revelou inconsistências invisíveis nas colunas originais.

## 2. O que a avaliação das variáveis mostrou

A base é estruturalmente completa: todas as 22.699 corridas têm as 18 informações preenchidas, sem campos em branco. Completude, porém, não é validade — há registros preenchidos e impossíveis.

**A amostra corresponde a 5,6% do volume documentado.** São 22.699 corridas, contra 408.294 previstas na documentação, sem critério de seleção declarado. Até a TLC informar como o recorte foi feito, os resultados descrevem esta amostra, não o ano de 2017.

**Gorjetas em dinheiro não são registradas.** Um terço das corridas (7.267, ou 32%) foi pago em dinheiro, e todas aparecem com gorjeta zero. Não é ausência de gorjeta: é ausência de registro. Hoje qualquer leitura sobre gratificação enxerga apenas quem pagou com cartão.

**181 corridas trazem registro impossível — menos de 1% da base.** São 148 sem distância percorrida e 33 sem passageiros. O volume é pequeno, mas a causa precisa ser conhecida antes de decidir o tratamento: corrida cancelada com cobrança mínima é registro legítimo; falha de captura, não.

**Há um código de tarifa que não existe no dicionário.** Aparece o código 99, fora da lista oficial de 1 a 6. É o único achado que estatística nenhuma resolve — depende de resposta da TLC.

**Os valores extremos foram identificados e explicados.** As 14 cobranças negativas (menos de 0,1%) são estornos coerentes, de corridas sem cobrança ou em disputa. Uma corrida de US$ 1.200,29 por 2,6 milhas destoa de tudo o mais — o segundo maior valor da base é US$ 450,30 — e é provável erro de lançamento, a confirmar com a TLC.

## 3. Próximas etapas recomendadas

Cinco definições, em ordem de prioridade. A primeira condiciona todas as demais.

1. **Confirmar como a amostra foi selecionada.** Define se os resultados valem para 2017 inteiro ou apenas para este recorte. Enquanto não houver resposta, nenhuma conclusão pode ser generalizada.
2. **Definir o que significa uma corrida sem distância percorrida.** Cancelamento com cobrança mínima e falha de captura exigem tratamentos opostos, e a distância é a base do cálculo da tarifa.
3. **Verificar se cada estorno tem a corrida original na mesma base.** A resposta determina se o par deve ser removido em conjunto do treinamento do modelo.
4. **Identificar o código de tarifa 99.** Códigos de tarifa afetam diretamente o valor cobrado; uma categoria não documentada é uma influência que não sabemos interpretar.
5. **Registrar gorjeta não capturada como "não informado", e não como zero.** Hoje a base afirma que não houve gorjeta quando o correto é que não se sabe — e um modelo aprenderia com essa afirmação.

**Variáveis para o modelo preditivo.** Para estimar a tarifa no momento do embarque, as duas informações mais promissoras são a **distância da corrida** e o **horário de embarque**: a distância determina o preço base, e o horário permite derivar dia da semana e faixa horária como aproximação de trânsito e adicionais noturnos. O **código de tarifa** deve entrar como terceira variável de apoio, para as viagens de preço fixo, como as do aeroporto JFK. Recomendamos prever a **tarifa base**, e não o valor total cobrado, porque o total inclui a gorjeta — justamente a informação que a base registra de forma incompleta.

---

## O que ficou de fora, e por quê

Aplicado o critério "isso muda uma decisão de quem lê?", não entraram: ajustes de formato de colunas (resolvidos internamente, sem decisão de ninguém); a diferença entre média e mediana dos valores; a corrida mais longa da base, de 33,96 milhas, geograficamente plausível; as 46 corridas acima de três horas; e a duração mediana de 11,2 minutos, coerente com deslocamento urbano — todos achados que confirmam normalidade em vez de exigir ação.

Se o texto não couber na página, o corte seguinte é o quinto achado do bloco 2 (valores extremos): é o item que tranquiliza, não o que exige decisão.
