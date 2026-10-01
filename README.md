# Cinéfilo Fuzzy

Controlador fuzzy Mamdani, em [scikit-fuzzy](https://pythonhosted.org/scikit-fuzzy/), que dá uma nota
de **adequação (0 a 10)** a um filme para uma sessão específica.

Mini-Projeto 2 de Sistemas Baseados em Conhecimento (UFPB), dá continuidade ao
[Cinéfilo](https://github.com/Nicolas-Kleiton/sbc-cinefilo) (Mini-Projeto 1).

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Nicolas-Kleiton/sbc-cinefilo-fuzzy/blob/main/Cinefilo_Fuzzy.ipynb)

## Domínio

O Mini-Projeto 1 usa regras IF–THEN nítidas (`experta`) para decidir **o que procurar** na TMDB.
Este projeto decide **quão bom é cada filme encontrado** para a sessão que o usuário tem em mãos.

Escolher um filme envolve julgamentos sem fronteira nítida. Um filme de 115 minutos numa sessão de
120 "cabe, mas apertado"; não existe um minuto exato em que ele deixa de caber. No MP1 esse tipo de
julgamento precisou virar um corte rígido (a regra R11 mudava de comportamento em 150 minutos).
Com lógica fuzzy, a transição fica gradual.

### Variáveis

| Variável | Tipo | Universo | Termos |
|---|---|---|---|
| `folga` | entrada | −90 a 120 min (tempo disponível − duração) | `estoura`, `justa`, `sobra` |
| `nota` | entrada | 0 a 10 (média TMDB) | `fraca`, `media`, `alta` |
| `consenso` | entrada | 0 a 5000 votos (saturado) | `baixo`, `medio`, `alto` |
| `adequacao` | saída | 0 a 10 | `baixa`, `media`, `alta` |

- **`folga` não é monotônica.** Se for negativa, o filme não cabe. Pequena e positiva é o ideal.
  Muito grande volta a ser pior, porque a sessão fica subaproveitada.
- **`consenso` separa nota de confiança.** Um 9,0 com 40 votos não vale o mesmo que um 8,5 com
  30 mil votos. Os pontos de quebra (200, 500 e 1000 votos) reaproveitam os patamares de
  `vote_count.gte` do MP1.

Cada ponto de quebra está justificado nos comentários do notebook (seção 3).

### Base de regras

São 27 regras, uma para cada combinação de termos das três entradas (3 × 3 × 3). A cobertura
completa é verificada por `assert` no próprio notebook. A base segue três princípios:

1. **O tempo é restrição dura.** Com `folga = estoura`, a adequação é `baixa`, qualquer que
   seja a nota.
2. **Nota fraca não se recupera.** Um filme mal avaliado sai `baixa` mesmo cabendo bem e tendo
   muitos votos.
3. **Consenso modula, não decide.** Poucos votos rebaixam a adequação em no máximo um nível.

A inferência é Mamdani: implicação por mínimo, agregação por máximo e defuzzificação por
centroide.

## Como executar

### Opção 1: Google Colab (recomendado)

1. Clique no botão **Abrir no Colab** acima.
2. Execute **Ambiente de execução → Executar tudo**.

A primeira célula instala as dependências. Nada precisa ser configurado antes.

### Opção 2: Jupyter local

Requer Python 3.10 ou mais recente.

```bash
git clone https://github.com/Nicolas-Kleiton/sbc-cinefilo-fuzzy.git
cd sbc-cinefilo-fuzzy

python -m venv .venv
# Windows:      .venv\Scripts\activate
# Linux/macOS:  source .venv/bin/activate

pip install scikit-fuzzy numpy scipy networkx packaging matplotlib requests jupyter
jupyter notebook Cinefilo_Fuzzy.ipynb
```

No Jupyter, use **Run → Run All Cells**. `networkx` e `packaging` aparecem na lista porque o
scikit-fuzzy 0.5.0 os importa sem declará-los como dependência.

### Chave da TMDB (opcional)

Somente a seção 8, que pontua filmes reais da TMDB, precisa de uma chave. Ela aceita tanto a
API Key (v3) quanto o Read Access Token (v4). O notebook procura a chave nesta ordem:

1. o secret `TMDB_API_KEY` do Colab (ícone de chave na barra lateral);
2. a variável de ambiente `TMDB_API_KEY`;
3. um campo para digitá-la durante a execução.

**Para rodar sem chave, aperte Enter nesse campo.** A seção 8 é pulada, e todo o restante
(controlador, testes e gráficos) funciona normalmente.

> Se o notebook for executado sem interação (por exemplo, com `jupyter nbconvert --execute`),
> defina a variável `TMDB_API_KEY` antes. O campo de digitação não funciona nesse modo.

## O que o notebook contém

| Seção | Conteúdo |
|---|---|
| 1 | Instalação e importações |
| 2 | Descrição do domínio |
| 3 | Variáveis linguísticas, funções de pertinência e seus gráficos |
| 4 | Base de 27 regras e a verificação de cobertura |
| 5 | Inferência com explicação: fuzzificação, regras ativadas e agregação |
| 6 | Casos de teste comentados e verificações globais |
| 7 | Superfície de controle |
| 8 | Ranqueamento de filmes reais da TMDB (opcional) |
| 9 | Comparação entre fuzzy e as regras nítidas do MP1 |

### Casos de teste

| Caso | folga | nota | votos | Adequação | Interpretação |
|---|---|---|---|---|---|
| 1. Ideal | +20 min | 8,4 | 12 000 | **8,42** | Aproveita bem a sessão e tem qualidade confirmada |
| 2. Não cabe | −60 min | 8,4 | 12 000 | **1,58** | Um ótimo filme que estoura a sessão não é recomendado |
| 3. Consenso frágil | +20 min | 8,8 | 90 | **5,00** | Nota alta com poucos votos vale menos que nota confirmada |

O notebook também traz um caso ambíguo (folga +35, nota 7,2, 800 votos) em que quatro regras
disparam com forças diferentes e o centroide as concilia, resultando em 5,75.

Pela defuzzificação por centroide, a saída útil vai de cerca de 1,5 a 8,5, e não de 0 a 10.

## Licença

[MIT](LICENSE)
