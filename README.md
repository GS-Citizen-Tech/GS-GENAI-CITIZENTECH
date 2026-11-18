# Assistente do Trabalhador CitizenTech (RAG sobre o gov.br)

**Global Solution 2026.2 · Generative AI & Advanced Nets · Turma 2TIAPF**

| Integrante | RM |
|---|---|
| Matheus Mikio Tutume Lourenço | 561864 |
| Ian Nobres Azevedo | 561861 |

Assistente conversacional que orienta o cidadão sobre **serviços de trabalho e benefícios** do portal gov.br (seguro-desemprego, recurso, empregado doméstico, abono salarial PIS/PASEP, carteira de trabalho e FGTS). Ele responde **somente** com base nas Cartas de Serviços oficiais, cita o trecho usado (URL, seção e data de atualização) e se abstém quando não há evidência.

**🎥 Vídeo de demonstração (até 5 min):** https://youtu.be/g1YSwIijkxs

## Entregáveis

| Entregável pedido | Onde está |
|---|---|
| Código e instruções de reprodução | [`GS_GenAI_CitizenTech_RAG.ipynb`](GS_GenAI_CitizenTech_RAG.ipynb) (já executado), [`requirements.txt`](requirements.txt) e a seção [Como reproduzir](#como-reproduzir) |
| Corpus e configuração (fontes, URLs, datas, chunking, retriever, embeddings, LLM, prompt, parâmetros) | seções [Corpus](#corpus) e [Configuração](#configuração), [`resultados/configuracao.json`](resultados/configuracao.json) |
| Relatório de avaliação (perguntas, evidências, respostas, métricas, análise por componente e end-to-end) | [`relatorio_avaliacao.md`](relatorio_avaliacao.md), dados brutos em [`resultados/`](resultados/) |
| Vídeo de até 5 minutos | link acima; roteiro em [`roteiro_video.md`](roteiro_video.md) |

### Correspondência com o barema

| Critério | Onde ver |
|---|---|
| Chunking | notebook Seção 2 e 13.1 · relatório: chunking e comparação de configurações |
| Retriever | notebook Seções 3 e 5 (BM25 × denso × híbrido × híbrido + rerank, top-k, gate) |
| Generator (LLM + prompt) | notebook Seção 6 e 13.2 (ablação do prompt) |
| Implementação e integração | pipeline completo nas Seções 1–6 e interface Gradio na Seção 10 |
| Avaliação dos componentes | Seções 5 (recuperação) e 7 (generator com contexto controlado) |
| Avaliação end-to-end | Seções 8 e 12 (típicas, ambíguas, sem resposta, fora do escopo, multi-turno) |
| Mitigação de erros e vulnerabilidades | Seções 2.1, 9 e 13.3 (13 ataques adversariais, ablação das defesas) |
| Interpretação dos resultados | `relatorio_avaliacao.md`, análise de erros, limitações e impacto para o cidadão |

## Como reproduzir

1. Abra `GS_GenAI_CitizenTech_RAG.ipynb` no Google Colab. Recomendado: *Ambiente de execução → Alterar o tipo → GPU T4*. O reranker roda bem mais rápido; em CPU a recuperação leva cerca de 10 s por pergunta.
2. Gere uma chave em https://platform.openai.com/api-keys (a conta precisa de saldo; a avaliação completa de 16/11/2026 custou US$ 0,15).
3. No Colab, clique no ícone de chave (Secrets), crie o segredo `OPENAI_API_KEY` com a chave e habilite o acesso do notebook. A chave nunca aparece no código nem nas saídas. `CONFIG["limite_gasto_usd"]` (padrão US$ 3) interrompe as chamadas se o custo estimado passar desse valor.
4. *Ambiente de execução → Executar tudo*. A execução completa leva poucos minutos em GPU.
5. Ao final, os resultados são baixados como `resultados_gs.zip` (Seções 0–11) e `resultados_complementares.zip` (Seção 13). A Seção 10 abre a interface de chat com link público temporário.

O notebook entregue já está executado, com as saídas da avaliação de 16/11/2026.

### Modelos

O provedor é escolhido em `CONFIG["provedor"]`. Os modelos de cada um ficam em `MODELOS_POR_PROVEDOR`, na mesma célula.

| Provedor | Gerador | Juiz | Reservas | Chave |
|---|---|---|---|---|
| **OpenAI** (padrão, pago) | `gpt-4.1-mini` | `gpt-5-mini` | `gpt-4o-mini`, `gpt-5-mini` | `OPENAI_API_KEY` |
| Groq (gratuito) | `meta-llama/llama-4-scout-17b-16e-instruct` | `openai/gpt-oss-120b` | `llama-3.3-70b-versatile`, `moonshotai/kimi-k2-instruct`, `qwen/qwen3-32b` | `GROQ_API_KEY` |
| Gemini (gratuito) | `gemini-3.1-flash-lite` | `gemini-3.8-flash` | `gemini-flash-lite-latest`, `gemini-3.8-flash` | `GEMINI_API_KEY` |

O juiz é sempre um modelo diferente do gerador, para reduzir a tendência de um modelo avaliar bem as próprias respostas. Na OpenAI, as respostas usam Structured Outputs (o JSON segue o schema exatamente) e o custo estimado aparece no cabeçalho de `relatorio_resultados.md`.

A terceira célula de código testa cada modelo configurado e mostra o erro real de cada um, se houver. Se um modelo esgotar a cota diária ou ficar indisponível, o notebook troca para uma reserva e registra a troca no cabeçalho de `relatorio_resultados.md`. Esperas por limite por minuto são automáticas. Perguntas que falharem por limite da API ficam marcadas com `falha_llm` e podem ser refeitas com `refazer_falhas()` na Seção 8.

Execução local (opcional): `pip install -r requirements.txt`, defina `OPENAI_API_KEY` como variável de ambiente e rode o notebook no Jupyter, removendo a primeira célula `!pip`.

## Experimentos complementares (Seção 13)

A Seção 13 fecha lacunas da avaliação sem alterar os resultados das Seções 0–12: chunking semântico (13.1), ablação do prompt (13.2), ataques mais difíceis contra um baseline sem defesas (13.3), variância entre execuções (13.4), latência real por etapa (13.5) e calibração do limiar do gate em um conjunto de validação separado (13.6). Os resultados vão para `resultados/complementares/` (CSVs e `complementares.md`) e, no Colab, são baixados como `resultados_complementares.zip`.

Há dois jeitos de rodar:

* **Notebook inteiro** (*Executar tudo*): 207 chamadas ao LLM na execução de 16/11/2026.
* **Só o necessário**: execute as Seções 0 a 6 em ordem, pule as Seções 7 a 12 e execute a Seção 13. São cerca de 100 chamadas (64 na 13.2 e até 36 na 13.3). A célula de configuração da Seção 13 traz as definições das Seções 7 e 8 de que ela precisa.

Flags na primeira célula de código da Seção 13:

| Flag | Padrão | Efeito |
|---|---|---|
| `RODAR_13_2` | `True` | ablação do prompt (até 64 chamadas) |
| `RODAR_13_3` | `True` | ataques novos em 4 cenários (até 36 chamadas) |
| `RODAR_REPETICOES` | `False` | repete o end-to-end completo `N_REPETICOES` vezes (custo alto) |
| `N_REPETICOES` | `2` | número de repetições novas |
| `JULGAR_REPETICOES` | `False` | `False` pula o juiz nas repetições para economizar cota |

Antes de cada subseção com API, o notebook imprime a estimativa de chamadas; `chamadas_api()` mostra o total já feito. As subseções 13.1, 13.5 e 13.6 não usam a API. Os documentos maliciosos da 13.3 ficam em um índice de teste separado e nunca entram no índice principal.

O `CONFIG` ganhou três flags usadas nesses experimentos, com padrão igual ao comportamento original: `mascarar_pii`, `usar_gate` e `system_prompt` (`None` usa o prompt da Seção 6).

## Corpus

Páginas oficiais do gov.br, baixadas pelo próprio notebook na primeira execução e salvas em `corpus/corpus.jsonl` com URL, data de consulta e a data de "Última Modificação" da página. O conteúdo do gov.br é publicado sob licença Creative Commons BY-ND 3.0.

| id | URL | Última modificação |
|---|---|---|
| seguro_desemprego | https://www.gov.br/pt-br/servicos/solicitar-o-seguro-desemprego | 25/09/2026 |
| faq_seguro | https://www.gov.br/pt-br/temas/perguntas-frequentes-seguro-desemprego | não informada |
| recurso_seguro | https://www.gov.br/pt-br/servicos/cadastrar-recurso-relativo-ao-seguro-desemprego | 25/09/2026 |
| seguro_domestico | https://www.gov.br/pt-br/servicos/solicitar-seguro-desemprego-empregado-domestico | 11/11/2026 |
| abono_salarial | https://www.gov.br/pt-br/servicos/receber-o-abono-salarial | 23/09/2026 |
| carteira_trabalho | https://www.gov.br/pt-br/servicos/obter-a-carteira-de-trabalho | 15/12/2025 |
| sacar_fgts | https://www.gov.br/pt-br/servicos/sacar-o-fgts | 15/12/2025 |

Data de consulta: 16/11/2026. As URLs, datas e a configuração usada também estão em `resultados/configuracao.json`.

O arquivo `corpus/corpus.jsonl` não acompanha o repositório: o notebook o gera na primeira execução, baixando as páginas acima. Nas execuções seguintes ele usa esse cache (`construir_corpus(forcar=True)` baixa de novo). Como o gov.br pode alterar as páginas, uma nova execução pode trazer um conteúdo diferente do avaliado em 16/11/2026; as datas de "Última modificação" acima permitem identificar essas mudanças.

Há também **documentos adversariais** criados pelo grupo, usados apenas nos testes de segurança: `fonte_comprometida` (Seção 9) e `portaria_falsa` e `atendimento_falso` (Seção 13.3). Eles ficam marcados como teste de segurança e são bloqueados pela lista de fontes permitidas (só domínios `gov.br`) e pelo sanitizador.

## Configuração

| Componente | Escolha |
|---|---|
| Limpeza | BeautifulSoup + markdownify, seções da Carta de Serviços promovidas a cabeçalhos, rodapés e menus removidos |
| Chunking | estrutural (por seção/pergunta do FAQ), até 1200 caracteres, overlap de 150 ao subdividir, cabeçalho contextual "Serviço / Seção", metadados completos |
| Embeddings | `intfloat/multilingual-e5-small` com prefixos `query:`/`passage:` |
| Retriever | híbrido BM25 + denso com Reciprocal Rank Fusion (k=60), reranker `cross-encoder/mmarco-mMiniLMv2-L12-H384-v1` sobre 20 candidatos, top-k = 4 |
| Filtros | lista de fontes permitidas (só `gov.br`), exclusão de trechos em quarentena, filtro opcional por categoria de serviço |
| Gate | abstenção sem chamar o LLM quando a similaridade máxima fica abaixo do limiar calibrado |
| LLM | `gpt-4.1-mini` (OpenAI), temperatura 0,1, Structured Outputs (JSON com schema), até 1024 tokens; juiz da avaliação `gpt-5-mini` |
| Prompt | papel, escopo fechado, regras de citação, abstenção, esclarecimento, conflito entre fontes e segurança; contexto em tags tratado como dado |
| Pós-validação | citações precisam existir no contexto, números da resposta precisam estar no contexto, links só para domínios `gov.br`, canário contra vazamento do prompt, máscara de CPF/e-mail/telefone |

Todos os parâmetros estão no dicionário `CONFIG` da segunda célula de código.

## Estrutura

```
GS_GenAI_CitizenTech_RAG.ipynb   notebook completo (ingestão → avaliação → interface)
README.md                        este arquivo
relatorio_avaliacao.md           relatório com justificativas, resultados e análise
resultados/                      resultados da execução de 16/11/2026 (GPT-4.1 mini)
resultados/complementares/       resultados da Seção 13 (experimentos complementares)
resultados_gemini_07-11/         execução anterior com Gemini, usada só na comparação do relatório
roteiro_video.md                 roteiro cronometrado do vídeo de 5 minutos
requirements.txt                 dependências
corpus/corpus.jsonl              gerado na primeira execução (cache do corpus; não versionado)
```

## Privacidade

Nenhuma credencial é salva no repositório. O assistente não armazena conversas; CPF, e-mail e telefone digitados são mascarados antes de chegar ao LLM e na resposta. Os testes usam apenas dados fictícios.
