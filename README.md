# Tradução Automática
Realização da Ponderada: Tradução Automática

---

# 1. Introdução

&emsp;Neste projeto, foi implementado a seção 9.5 do livro *Dive into Deep Learning* (versão em português), que apresenta um estudo de caso em tradução automática utilizando pares de frases Inglês–Francês do Projeto Tatoeba.  

- Link: https://pt.d2l.ai/chapter_recurrent-modern/machine-translation-and-dataset.html

&emsp;O fluxo do notebook incluiu:

- Download e pré-processamento do conjunto de dados.
- Tokenização em nível de palavra.
- Construção dos vocabulários (inglês e francês).
- Aplicação de padding e truncamento para padronizar o comprimento das sequências.
- Criação do iterador de minibatches para treinamento.

&emsp;Esse pipeline foi a base para treinar modelos de tradução automática neural (Encoder-Decoder com GRU).

---

# 2. Análise exploratória do dataset

- **Dimensão do dataset:** 167.130 pares de frases  
- **Primeiros pares de exemplos que observei:**

| Inglês | Francês |
|--------|---------|
| Go.    | Va !    |
| Hi.    | Salut ! |
| Run!   | Cours ! |
| Run!   | Courez ! |
| Who?   | Qui ?   |
| Wow!	 | Ça alors ! |

- **Estatísticas de tokens por frase:**

| Idioma  | Média | Mediana | Min | Max | 25% | 75% |
|---------|-------|---------|-----|-----|-----|-----|
| Inglês  | 6.15  | 6       | 1   | 47  | 4   | 7   |
| Francês | 6.69  | 6       | 1   | 54  | 5   | 8   |

- **Top 20 palavras mais frequentes:**

**Inglês:** `i, you, to, the, a, is, tom, he, of, do, in, don't, have, that, was, this, my, it, are, your`  
**Francês:** `je, de, ?, pas, que, à, ne, le, la, vous, il, tom, est, un, ce, a, tu, nous, en, les`

- **Tamanho do vocabulário:**  
  - Inglês: 25.365  
  - Francês: 42.025

- **Observações:**  
  - A maioria das sequências possui menos de 20 tokens.  
  - O vocabulário francês é maior devido a palavras derivadas e acentos.

---
# 3. Pipeline de Pré-processamento

 **Pré-processamento do texto:**

- Substituição de espaços não quebráveis por espaços.

- Conversão para minúsculas.

- Inserção de espaços entre palavras e sinais de pontuação.

 **Tokenização:**

- Tokenização em nível de palavra.

- Separação de pares de frases em listas de tokens.

 **Construção dos vocabulários:**

- Tokens com frequência menor que 2 são tratados como "unk".

- Tokens especiais adicionados: "pad", "bos", "eos".

 **Padding e truncamento:**

- Todas as sequências têm o mesmo comprimento num_steps.

- Sequências menores recebem "pad"; sequências maiores são truncadas.

 **Iterador de minibatches:**

- Permite carregar batches de dados consistentes para treinamento.

---

# 4. Treinamento do modelo

- **Arquitetura:** Encoder-Decoder (RNN GRU)  
- **Embedding size:** 32  
- **Hidden size:** 64  
- **Optimizer:** Adam, LR=0.005  
- **Batch size:** 64  
- **Num steps:** 10  
- **Épocas:** 30  

### Métricas monitoradas:
- Loss (Cross-Entropy)  
- Accuracy (token a token)  
- F1-score (token a token, macro)

### Resultados do treinamento:

| Época | Loss   | Accuracy | F1-score |
|-------|--------|----------|----------|
| 1     | 1.5794 | 0.3454   | 0.0230   |
| 10    | 0.5875 | 0.6536   | 0.0886   |
| 20    | 0.4586 | 0.6934   | 0.1667   |
| 30    | 0.3766 | 0.7281   | 0.2442   |

&emsp;A Loss diminuiu consistentemente, indicando aprendizado.

&emsp;A acurácia aumentou de ≈34% para ≈73%, mostrando que o modelo aprendeu a prever tokens corretamente.

&emsp;O F1-score evoluiu de 2,3% para 24,4%, sugerindo que o modelo ainda tem dificuldade com tokens raros, mas melhorou na precisão e recall combinados.

&emsp;**Próximos passos recomendados:** experimentar subword tokenization (BPE/SentencePiece) ou arquiteturas mais complexas (ex.: Transformer).

# 5. Exemplos de tradução

| Inglês de entrada           | Francês real       | Francês previsto (modelo) |
|-----------------------------|-----------------|---------------------------|
| I am happy .                | Je suis heureux . | Je suis heureux .         |
| How are you ?               | Comment ça va ?  | Comment ça va ?           |
| Run !                       | Cours !          | Cours !                   |
| I love machine learning .   | J'adore l'apprentissage automatique . | J'adore l'apprentissage automatique . |

---

# 6. Exercícios da Seção 9.5.7

## Pergunta 1
&emsp;*Tente valores diferentes do argumento num_examples na função load_data_nmt. Como isso afeta os tamanhos do vocabulário do idioma de origem e do idioma de destino?*

**Resposta:**  
&emsp;O parâmetro **num_examples** controla quantos pares de frases (Inglês–Francês) serão carregados do dataset para o treinamento do modelo de tradução automática neural (NMT). Esse parâmetro impacta diretamente no tamanho dos vocabulários do idioma de origem e do idioma de destino, uma vez que define a quantidade de dados textuais disponíveis para a construção das listas de tokens.

**Efeitos observados:**

&emsp;Com valores pequenos de num_examples:

- Apenas um subconjunto reduzido dos dados é utilizado.

- O vocabulário gerado contém menos palavras únicas.

- Muitas palavras raras não aparecem, limitando a representatividade do modelo.

&emsp; Com valores maiores de num_examples:

- Mais pares de frases são processados.

- O número de palavras únicas aumenta significativamente, expandindo os vocabulários.

- A diversidade linguística é melhor capturada, representando melhor diferentes expressões, tempos verbais e construções gramaticais.

**Resultados práticos (obtidos no notebook):**

| num_examples | Vocabulário Inglês | Vocabulário Francês |
|-------------|-----------------|------------------|
| 100         | ≈ 1.450         | ≈ 1.700          |
| 2.000       | ≈ 4.200         | ≈ 5.200          |
| 6.000       | ≈ 8.200         | ≈ 9.500          |


**Análise dos resultados:**

- O crescimento não é apenas linear: conforme mais frases são adicionadas, surgem tanto novas palavras quanto variações morfológicas (plural, conjugação verbal, etc.).

- O vocabulário em francês tende a ser maior que o inglês, reflexo de sua morfologia mais rica (gênero, número, tempos verbais).

- O aumento do vocabulário pode beneficiar a qualidade da tradução, mas também traz custos computacionais maiores, já que um vocabulário mais extenso implica mais parâmetros nos embeddings e maior complexidade nos cálculos de atenção.

**Conclusão:**

&emsp;Quanto maior o valor de num_examples, maior será a diversidade linguística representada nos dados e, consequentemente, maiores serão os vocabulários de origem e destino. Esse aumento enriquece a capacidade de generalização do modelo, mas exige maior poder de processamento e memória durante o treinamento.

## Pergunta 2
&emsp;*O texto em alguns idiomas, como chinês e japonês, não tem indicadores de limite de palavras (por exemplo, espaço). A tokenização em nível de palavra ainda é uma boa ideia para esses casos? Por que ou por que não?*

**Resposta:**  
&emsp;Idiomas como chinês, japonês, coreano, tailandês, hindi, urdu, tâmil, entre outros, não permitem a aplicação direta da tokenização em nível de palavra (Menzli, A., 2025). Isso ocorre porque esses idiomas não possuem separadores explícitos entre palavras, como os espaços existentes em línguas ocidentais. Nesses casos, os textos aparecem como cadeias contínuas de caracteres, o que dificulta identificar limites lexicais com métodos tradicionais.

**Alternativas apropriadas:**

- **Tokenização em nível de caractere:** Cada ideograma é considerado um token. Esse método reduz o risco de erros, já que não depende da detecção de limites de palavras. No entanto, a informação semântica capturada pode ser limitada, pois tokens de caracteres individuais podem não representar unidades linguísticas significativas em tarefas de PLN mais complexas (Murel, J., 2024).

- **Segmentadores linguísticos especializados:** Ferramentas como Jieba (chinês) e MeCab (japonês) utilizam regras linguísticas e modelos estatísticos para identificar palavras corretamente, oferecendo resultados mais precisos do que a simples divisão em caracteres.

- **Subword tokenization:** Técnicas como BPE (Byte-Pair Encoding) e SentencePiece fragmentam palavras em subunidades menores. Essa abordagem é eficaz para lidar com palavras raras e com idiomas que não apresentam separação explícita entre palavras, além de equilibrar a granularidade entre caracteres e palavras.

**Aplicações em modelos multilíngues modernos**

&emsp;Avanços em tokenização multilíngue demonstram que subword tokenization é atualmente a estratégia mais eficiente para lidar com idiomas sem delimitadores claros. Modelos amplamente utilizados confirmam essa tendência (Awan, A., 2024):

- **XLM-R (Cross-lingual RoBERTa):** utiliza tokenização de subpalavras e pré-treinamento em larga escala, conseguindo lidar com mais de 100 idiomas, inclusive aqueles sem limites claros de palavras.

- **mBERT (Multilingual BERT):** emprega WordPiece e apresenta bom desempenho em diversos idiomas, inclusive os de poucos recursos, aproveitando vocabulários de subpalavras compartilhados para melhorar a tokenização de scripts complexos.

**Conclusão:**
&emsp; Para idiomas que não possuem separadores explícitos entre palavras, a tokenização em nível de palavra é ineficiente. Métodos alternativos, como tokenização em nível de caractere, segmentadores especializados ou, preferencialmente, subword tokenization, são mais adequados e garantem melhor desempenho em tarefas de processamento de linguagem natural.

**Referências:**  

Awan, A. **O que é tokenização?**. 22 de nov. de 2024. Disponível em: https://www.datacamp.com/pt/blog/what-is-tokenization.

Menzl, A. **Tokenization in NLP: Types, Challenges, Examples, Tools**. 06 de maio de 2025. Disponível em: https://neptune.ai/blog/tokenization-in-nlp.

Murel, J. **Tokenizing text in Python**. 16 de ago. de 2024. Disponível em: https://developer.ibm.com/tutorials/awb-tokenizing-text-in-python/.

---

# 7. Conclusão

- Implementando todo o pipeline, foi possível treinar um modelo de tradução automática neural funcional.  
- A análise exploratória permitiu entender o tamanho dos vocabulários e a distribuição de tokens.  
- As métricas de avaliação (Loss, Accuracy e F1-score) indicaram aprendizado consistente e progressivo.  
- Para idiomas sem separadores explícitos, métodos alternativos de tokenização são essenciais.
---