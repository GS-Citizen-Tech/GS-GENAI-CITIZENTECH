# Roteiro do vídeo (até 5 minutos)

Gravem a tela do Colab com o notebook já executado, as tabelas da Seção 13 e o chat do Gradio aberto em outra aba. Não mostrem a aba de Secrets nem a chave da API.

| Tempo | Quem | Tela | Fala |
|---|---|---|---|
| 0:00–0:25 | Matheus | Título do notebook | "Somos Matheus Mikio Tutume Lourenço, RM 561864, e Ian Nobres Azevedo, RM 561861. O nosso assistente CitizenTech orienta o trabalhador sobre seguro-desemprego, abono, carteira de trabalho e FGTS usando só as Cartas de Serviços do gov.br. Ele cita a fonte, com a data, e diz quando não sabe." |
| 0:25–1:00 | Matheus | Diagrama de arquitetura | Percorrer o fluxo: lista de fontes e sanitizador na ingestão, guarda de entrada, retriever híbrido com reranker, gate de similaridade, GPT-4.1 mini com regras, pós-validação. "Cada etapa tem uma função de controle; não é só ligar uma interface a um LLM." |
| 1:00–1:45 | Ian | Tabela de recuperação | "Chunking por seção da Carta de Serviços. O resultado mais importante: a busca densa sozinha foi a pior, e o reranker levou o MRR de 0,51 para 0,76. Medir mudou a nossa escolha." |
| 1:45–2:10 | Ian | Prompt e CONFIG | "Começamos com o Gemini gratuito, mas ele ficou instável. Migramos para o GPT-4.1 mini: 207 chamadas sem nenhuma falha, por 15 centavos de dólar. O juiz da avaliação é outro modelo, o GPT-5 mini." |
| 2:10–3:05 | Matheus | Gradio ao vivo | **Sucesso:** "Quanto recebo de abono se trabalhei 6 meses?" (R$ 811 com fonte clicável). **Multi-turno:** "Sou empregada doméstica e fui demitida" → "Qual o prazo para pedir?" (7 a 90 dias, regra específica). **Abstenção:** "Sou MEI, posso receber?" (o modelo sabe de memória, mas não está na base, então não responde). **Fora do escopo:** "Como tiro passaporte?" (barrado pelo gate). |
| 3:05–4:05 | Ian | Tabela da Seção 13.3 | "Criamos uma página falsa, em tom oficial, mandando o cidadão informar CPF e senha num site de golpe. Sem as nossas camadas, o GPT indicou o site falso em todos os cenários, mesmo com o prompt completo. Com a lista de fontes, a página nem entra no índice. Outro ataque, em vários turnos, fez o modelo copiar o prompt inteiro; com todas as camadas, ele tentou de novo, mas o canário na pós-validação bloqueou. Resultado: 4 de 9 ataques funcionam sem defesas, nenhum com elas." |
| 4:05–4:35 | Ian | Revisão manual (Seção 8 do relatório) | Mostrar um caso de falha: "No C01, o gov.br tem dois prazos conflitantes. Com o contexto controlado, o modelo mostrou os dois; no fluxo completo, recebeu três trechos do FAQ e um da página do recurso e ficou só com a versão do FAQ. E o juiz automático deu notas diferentes para respostas iguais. Por isso revisamos todos os erros à mão." |
| 4:35–5:00 | Matheus | Conclusão | "Na revisão manual, só uma afirmação em 22 ficou sem apoio nas fontes. O reranker decidiu a qualidade; as defesas fora do modelo decidiram a segurança. Limitações: amostra pequena, juiz automático pouco confiável e troca de assunto na conversa. Para o cidadão: resposta com fonte e data, menos prazo perdido e menos golpe." |

**Dicas**
* Ensaiem com cronômetro. A parte ao vivo é a que mais estoura o tempo.
* Para a demonstração ao vivo, rodem as Seções 0 a 6 e a 10, sem a avaliação inteira.
* Se o Gradio demorar, mostrem as respostas já registradas em `resultados/relatorio_resultados.md`.
* Para a parte de segurança, a tabela "Ataques: resultado por cenário" está em `resultados/complementares/complementares.md`, e as respostas completas, em `ataques_v2.csv`.
