# Conteúdo Educacional — BNCC Matemática (Anos Finais)

Este diretório contém a referência educacional da aplicação, organizada segundo a **Base Nacional Comum Curricular (BNCC)** — componente curricular Matemática, etapa Ensino Fundamental, Anos Finais (6º ao 9º ano).

## Codificação das Habilidades

Cada habilidade é identificada por um código alfanumérico, conforme convenção da BNCC:

```
EFyyMAzz
│  │  │└─ número sequencial da habilidade
│  │  └── componente curricular (MA = Matemática)
│  └───── ano escolar (06, 07, 08, 09 ou bloco 67, 68, 69)
└──────── etapa (EF = Ensino Fundamental)
```

**Exemplo:** `EF07MA13` → 13ª habilidade de Matemática do 7º ano do Ensino Fundamental.

## Unidades Temáticas

A BNCC organiza a Matemática em 5 unidades temáticas. Cada arquivo abaixo detalha uma unidade, percorrendo do 6º ao 9º ano, com os **objetos de conhecimento** e a lista completa de **habilidades** correspondentes.

| # | Unidade Temática | Arquivo | Ideia-chave articuladora |
|---|---|---|---|
| 1 | **Números** | [`numeros.md`](numeros.md) | Pensamento numérico: quantificar, julgar e interpretar quantidades. |
| 2 | **Álgebra** | [`algebra.md`](algebra.md) | Pensamento algébrico: generalizar padrões, modelar relações. |
| 3 | **Geometria** | [`geometria.md`](geometria.md) | Pensamento geométrico: formas, posições, transformações. |
| 4 | **Grandezas e Medidas** | [`grandezas-medidas.md`](grandezas-medidas.md) | Mensurar, comparar grandezas, estimar. |
| 5 | **Probabilidade e Estatística** | [`probabilidade-estatistica.md`](probabilidade-estatistica.md) | Tratamento da informação, incerteza, leitura crítica. |

## Ideias Fundamentais Articuladoras

Presentes em todas as unidades, atravessam os 4 anos:

- **Equivalência** — diferentes representações para um mesmo objeto matemático.
- **Ordem** — comparação, classificação, seriação.
- **Proporcionalidade** — relação multiplicativa entre grandezas.
- **Interdependência** — relações entre conceitos (ex.: geometria × medidas).
- **Representação** — registros, símbolos, linguagens.
- **Variação** — taxas, funções, regularidades.
- **Aproximação** — estimativas, arredondamentos, erros.

## Mapa de Quantidade de Habilidades por Ano e Unidade

| Ano | Números | Álgebra | Geometria | G&M | P&E | Total |
|---|---|---|---|---|---|---|
| 6º  | 13 | 2 | 9 | 7 | 3 | 34 |
| 7º  | 12 | 4 | 8 | 6 | 4 | 34 |
| 8º  | 5  | 7 | 6 | 4 | 5 | 27 |
| 9º  | 5  | 8 | 5 | 2 | 4 | 24 |

> A quantidade é aproximada e reflete o quantitativo de habilidades descritas oficialmente na BNCC. Detalhes em cada arquivo de unidade.

## Uso na Aplicação

- **Tags das questões:** cada questão do banco recebe tags correspondentes ao seu `ano`, `unidade_tematica`, `objeto_conhecimento` e `habilidade_bncc` (código `EFyyMAzz`).
- **Filtros:** o usuário pode filtrar questões por ano escolar, unidade temática, objeto de conhecimento ou habilidade específica.
- **Progresso:** o sistema acompanha o desempenho do aluno por unidade temática, alimentando badges e rankings segmentados.
