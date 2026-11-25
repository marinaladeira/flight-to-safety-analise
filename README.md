# O título público como porto seguro em crises: EUA e Brasil (2003-2026)

Código e resultados para reproduzir as análises do artigo *O título público como porto seguro em crises: um estudo de caso comparativo entre Estados Unidos e Brasil (2003-2026)*.

O estudo compara a correlação diária entre ações e título público nos dois países, de 16/09/2003 a 30/09/2026: S&P 500 x Treasury de 10 anos nos EUA e Ibovespa x IMA-B (NTN-B) no Brasil, em crise e fora dela.

## Como reproduzir

Requer Python 3.12 (testado com 3.12.5).

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m ipykernel install --user --name replicacao
```

### 1. Obter os dados

Só as séries de fontes públicas estão no repositório: Treasury e TIPS (Federal Reserve, H.15) e PTAX (Banco Central). As demais pertencem a provedores que não permitem redistribuição (ver [Licença](#licença)) e precisam ser baixadas das fontes originais. O `01_coleta.ipynb` faz isso e grava tudo em `dados/brutos/<data de hoje>/`, pasta que o processamento passa a usar:

```bash
cd codigo
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.kernel_name=replicacao 01_coleta.ipynb
```

Baixadas hoje, as séries podem ter pequenas revisões em relação às de 21/10/2026, e os números podem diferir na última casa decimal. Para a reprodução exata, os arquivos originais podem ser pedidos aos autores; colocados em `dados/brutos/2026-10-21/` e `dados/brutos/complementares/`, eles podem ser conferidos pelo SHA-256 registrado em `manifesto.csv` e `manifesto_complementar.csv`.

### 2. Rodar a análise

Rode os notebooks em ordem, a partir da pasta `codigo/` (kernel `replicacao`):

```bash
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.kernel_name=replicacao 02_processamento.ipynb
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.kernel_name=replicacao 03_regimes.ipynb
jupyter nbconvert --to notebook --execute --inplace --ExecutePreprocessor.kernel_name=replicacao 04_analise.ipynb
cd ..
python codigo/figuras_pb.py . figuras
```

O `04_analise.ipynb` leva alguns minutos (*bootstrap* com 2.000 réplicas, semente fixa). Com os arquivos originais, rodada do zero em ambiente limpo, a sequência acima reproduz a base processada e todos os arquivos de `resultados/` sem nenhuma diferença.

## Estrutura

| Pasta / arquivo | Conteúdo |
|---|---|
| `dados/brutos/2026-10-21/` | Séries públicas como foram baixadas em 21/10/2026 (H.15 e PTAX) e `manifesto.csv` com fonte, URL, período e SHA-256 de todos os arquivos usados, inclusive os que não estão no repositório |
| `dados/brutos/complementares/` | Manifesto do EMBI+ Risco-Brasil (Ipeadata), baixado em 02/11/2026 |
| `dados/fontes.md` | O que é cada série e qual papel cumpre na análise |
| `dados/dicionario.md` | Descrição de cada coluna da base processada |
| `dados/processados/` | `log_processamento.md` (verificações e dias excluídos). A `base_diaria.csv` (5.526 dias úteis comuns) é gerada pelo `02_processamento.ipynb` e não fica no repositório |
| `codigo/01_coleta.ipynb` | Download das séries, inclusive o EMBI+ |
| `codigo/02_processamento.ipynb` | Alinhamento de datas, retornos, quedas (*drawdowns*) e indicadores de regime de crise |
| `codigo/03_regimes.ipynb` | Dias e episódios por regime (Tabela 1) |
| `codigo/04_analise.ipynb` | Correlações, testes das proposições P1 a P4, ano de 2022, robustez, EMBI+ e figuras |
| `codigo/figuras_pb.py` | Versão em preto e branco das Figuras 1 e 2 (a usada na submissão) |
| `resultados/` | Saídas em CSV citadas no artigo |
| `figuras/` | Figuras 1 e 2, em cores e em preto e branco |

## Onde está cada resultado do artigo

| No artigo | Arquivo | Gerado por |
|---|---|---|
| Tabela 1 – Dias e episódios por regime | `resultados/contagem_regimes.csv` | `03_regimes.ipynb` |
| Figura 1 – Correlação móvel de 63 pregões | `figuras/figura1_correlacao_movel.png`, `figuras/figura1_pb.png`; dados em `resultados/correlacoes_moveis.csv` | `04_analise.ipynb`, `figuras_pb.py` |
| Tabela 2 – Correlação por regime; P1, P3 e P4 (seções 4.2 e 4.3) | `resultados/testes_proposicoes.csv` | `04_analise.ipynb` |
| Tabela 3 – Regressão da correlação móvel sobre o VIX (P2) | `resultados/p2_regressoes_vix.csv` | `04_analise.ipynb` |
| P2 – Correlação média por faixa do VIX | `resultados/p2_faixas_vix.csv` | `04_analise.ipynb` |
| P2 – Diferença entre os coeficientes dos dois países | saída impressa da célula da P2 | `04_analise.ipynb` |
| P3 – Regressão da NTN-B sobre Ibovespa e câmbio | `resultados/p3_regressoes.csv` | `04_analise.ipynb` |
| Seção 4.4 e Figura 2 – Correlação por ano e o episódio de 2022 | `resultados/correlacao_por_ano.csv`, `figuras/figura2_*.png`; teste de 2022 em `resultados/testes_proposicoes.csv` | `04_analise.ipynb`, `figuras_pb.py` |
| Quadro 3 – Testes de robustez | `resultados/robustez.csv` | `04_analise.ipynb` |
| P4 com o EMBI+ Risco-Brasil | `resultados/p4_embi.csv` | `04_analise.ipynb` |
| Correlação móvel média por regime | `resultados/correlacao_movel_media_por_regime.csv` | `04_analise.ipynb` |

Nos CSVs de testes, `r_a` e `r_b` são as correlações nos dois regimes comparados, `dif` é a diferença, `ic95_dif_boot` e `p_boot_dif` vêm do *bootstrap* em blocos circular (21 pregões; 63 nas linhas "bloco 63"), `p_fisher` é o teste z de Fisher e `r_a_forbes_rigobon` é a correlação de crise com o ajuste de Forbes e Rigobon (2002).

## Fontes dos dados

| Série | Fonte |
|---|---|
| S&P 500 (^GSPC), Ibovespa (^BVSP), ETF IEF | Yahoo Finance |
| Treasury de 10 anos e TIPS de 10 anos | Federal Reserve, publicação H.15 |
| VIX | Cboe |
| IMA-B, IMA-B 5+, IDkA IPCA 10 anos, IRF-M | ANBIMA |
| Câmbio PTAX de venda | Banco Central do Brasil, SGS, série 1 |
| EMBI+ Risco-Brasil | J.P. Morgan, via Ipeadata (JPM366_EMBI366) |

O câmbio de fechamento do Yahoo Finance (BRL=X) é baixado porque entra no processamento, mas não é usado nas análises: foi descartado como controle por aparecer com data deslocada em relação à PTAX a partir de 2010 (seção 3.5 do artigo).

## Licença

| Parte | Licença |
|---|---|
| Código (`codigo/`) | [MIT](LICENSE) |
| Resultados e figuras (`resultados/`, `figuras/`) | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.pt_BR): uso livre, citando o artigo |
| Dados (`dados/`) | Não estão cobertos pelas licenças acima. Os direitos são de cada provedor, nas condições abaixo |

| Provedor | Séries | Condição | No repositório |
|---|---|---|---|
| Federal Reserve (H.15) | Treasury e TIPS de 10 anos | Dado público do governo dos EUA | Sim |
| Banco Central do Brasil (SGS) | PTAX | Dado aberto | Sim |
| ANBIMA | IMA-B, IMA-B 5+, IDkA IPCA 10A, IRF-M | Propriedade da ANBIMA; reprodução depende de autorização | Não |
| Cboe | VIX | Propriedade da Cboe; redistribuição depende de licença | Não |
| S&P Dow Jones Indices, B3, iShares, via Yahoo Finance | S&P 500, Ibovespa, IEF, BRL=X | Dados do Yahoo Finance licenciados para uso pessoal | Não |
| J.P. Morgan, via Ipeadata | EMBI+ Risco-Brasil | Índice proprietário da J.P. Morgan | Não |

Pelo mesmo motivo, a base processada e o VIX diário não são publicados. Os arquivos de `resultados/` trazem só correlações, coeficientes e outras estatísticas calculadas a partir das séries. Quem for usar os dados deve obtê-los nas fontes originais e seguir os termos de cada provedor.
