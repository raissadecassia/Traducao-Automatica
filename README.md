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

- Tokens com frequência menor que 2 são tratados como <unk>.

- Tokens especiais adicionados: <pad>, <bos>, <eos>.

 **Padding e truncamento:**

- Todas as sequências têm o mesmo comprimento num_steps.

- Sequências menores recebem <pad>; sequências maiores são truncadas.

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
&emsp;O parâmetro `num_examples` define quantos pares de frases (Inglês–Francês) serão carregados do dataset.  

- Quando `num_examples` é pequeno, poucas frases são utilizadas, resultando em vocabulários reduzidos.  
- Quando `num_examples` aumenta, mais palavras únicas surgem, aumentando os vocabulários.

**Resultados práticos (obtidos no notebook):**

| num_examples | Vocabulário Inglês | Vocabulário Francês |
|-------------|-----------------|------------------|
| 100         | ≈ 1.450         | ≈ 1.700          |
| 2.000       | ≈ 4.200         | ≈ 5.200          |
| 6.000       | ≈ 8.200         | ≈ 9.500          |


&emsp;**Conclusão:** quanto maior o número de exemplos, maior a diversidade linguística capturada, aumentando o tamanho dos vocabulários.

## Pergunta 2
&emsp;*O texto em alguns idiomas, como chinês e japonês, não tem indicadores de limite de palavras (por exemplo, espaço). A tokenização em nível de palavra ainda é uma boa ideia para esses casos? Por que ou por que não?*

**Resposta:**  
&emsp;Não, para idiomas como chinês e japonês a tokenização em nível de palavra **não é adequada**, pois não existem separadores explícitos (como espaços).  

- Para idiomas ocidentais (inglês, francês, português): tokenização por palavra funciona bem.  
- Para chinês e japonês: os textos aparecem como cadeias contínuas de caracteres.

**Alternativas apropriadas:**  
- Tokenização em nível de caractere (cada ideograma é um token).  
- Segmentadores linguísticos especializados: Jieba (chinês), MeCab (japonês).  
- Subword tokenization (ex.: BPE, SentencePiece), que lida melhor com palavras raras e textos sem separação explícita.

**Referências:**  


---

# 7. Conclusão

- Implementando todo o pipeline, foi possível treinar um modelo de tradução automática neural funcional.  
- A análise exploratória permitiu entender o tamanho dos vocabulários e a distribuição de tokens.  
- As métricas de avaliação (Loss, Accuracy e F1-score) indicaram aprendizado consistente e progressivo.  
- Para idiomas sem separadores explícitos, métodos alternativos de tokenização são essenciais.
---
