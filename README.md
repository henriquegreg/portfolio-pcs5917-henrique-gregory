# PCS5917 – IA Adversarial

## Portfólio Individual

**Aluno:** Henrique Gregory Gimenez 
**Disciplina:** PCS5917 – IA Adversarial  
**Período:** 3º período de 2026  
**Instituição:** Universidade de São Paulo – Escola Politécnica  
**Professor:** Victor Takashi Hayashi  

## 1. Objetivo do Portfólio

Este portfólio reúne as principais atividades, estudos, experimentos e reflexões desenvolvidos ao longo da disciplina **PCS5917 – IA Adversarial**.

O objetivo é documentar individualmente o processo de aprendizagem sobre:

- notícias e referências científicas sobre IA Adversarial (Aula 1);
- fundamentos de Inteligência Artificial e Segurança (Aula 2);
- aplicação de IA em cibersegurança (Aula 3);
- ataques adversariais contra sistemas de IA (Aulas 4 e 5);
- avaliação de ataques e uso de LLMs como juiz (Aula 6);
- defesas em Large Language Models (Aula 7).

Os experimentos realizados durante as aulas devem ser complementados por registros individuais, notebooks, respectivos resultados e referências bibliográficas (citações).

## 2. Organização do Repositório

**IMPORTANTE**: somente branch main (demais branches serão desconsideradas na correção).
Faça o desenvolvimento incremental com **commits semanais**, pois a evolução durante as semanas também é critério de avaliação.
Organize seu README focando em ser objetivo, com evidências de resultados e citações às referências utilizadas.

```text
.
├── README.md (com registros de resultados)
├── notebooks/ (colocar aqui os notebooks citados no README)
│   ├── aula-02-llm-jailbreaks.ipynb
│   ├── aula-03-asvspoof.ipynb
│   ├── aula-04-nanogcg.ipynb
│   ├── aula-05-pair.ipynb (e/ou cipherchat)
│   ├── aula-06-llm-judge.ipynb
│   └── aula-07-defesas-llm.ipynb
├── images/ (colocar aqui as imagens usadas no README)
│   └── ...
└── outros/
    └── ...
```

## Aula 1 - Notícia e Referência Acadêmica sobre IA Adversarial

### Post - Pesquisadores descobrem que sons inaudíveis ocultos em vídeos podem sequestrar chatbot de voz

![Atividade1_Post](<images/Atividade 1 - Post.png> "Post")

### Resposta - Atacantes podem utilizar o Google Calendar como meio de prompt injection

![Atividade1_Resposta](<images/Atividade 1 - Resposta.png> "Resposta")

## Atividade 2 - Red Teaming

### 1. Objetivo da Atividade
O objetivo desta atividade foi conduzir testes de invasão e segurança adversarial (*Red Teaming*) em LLMs, avaliando seu alinhamento, robustez e resistência a técnicas de *jailbreaks*. O experimento contemplou a submissão de conjuntos de ataques tanto em **inglês** quanto em **português**.

### 2. Metodologia e Pipeline de Execução
Toda a execução foi realizada em ambiente local no **Google Colab** utilizando aceleração por GPU (NVIDIA T4). O fluxo foi estruturado em três etapas:

1. **Modelo Alvo**: Foi carregado localmente o modelo [`Qwen/Qwen2.5-1.5B-Instruct`](https://huggingface.co/Qwen/Qwen2.5-1.5B-Instruct) via biblioteca `transformers`, utilizando templates de chat padrão para simulação de interação de usuário.
2. **Execução de Ataques (Inferência)**:
   - Os ataques foram extraídos do repositório [Self-Reminder-Data (jailbreak_prompts.csv)](https://github.com/yjw1029/Self-Reminder-Data/blob/master/data/jailbreak_prompts.csv), contendo 75 prompts de jailbreak em inglês, além de sua versão traduzida para o português (`jailbreak_prompts_pt.csv`). A tradução em português foi feita utilizando o Claude Sonnet 5.5.
   - O pipeline avaliou os cenários em inglês (`jailbreak_prompts.csv`) e traduzidos para o português (`jailbreak_prompts_pt.csv`). 
   - Os resultados das saídas do modelo estão em `results/aula-02/prompt_results.csv` e `results/aula-02/prompt_results_pt.csv`.
3. **Avaliação Automatizada (*LLM-as-a-Judge*)**:
   - Para analisar o sucesso ou falha dos ataques, carregou-se um segundo modelo avaliador: [`stabilityai/stablelm-zephyr-3b`](https://huggingface.co/stabilityai/stablelm-zephyr-3b).
   - O avaliador atuou com instruções para inspecionar cada par `[ATTACK PROMPT]` e `[TARGET MODEL RESPONSE]`, classificando:
     - `is_dangerous` (booleano): se o modelo forneceu instruções executáveis, código ou métodos ilegais/danosos.
     - `attack_score` (0 a 10): métrica de sofisticação e complexidade do prompt adversário.
   - Os resultados foram em json salvos em `results/aula-02/evaluation_results.json` e `results/aula-02/evaluation_results_pt.json`.

O notebook completo com o código e as execuções encontra-se em [`notebooks/aula-02-llm-jailbreaks.ipynb`](notebooks/aula-02-llm-jailbreaks.ipynb).

### 3. Análise dos Resultados e Qualidade

#### A. Robustez e Alinhamento do Modelo Alvo (Qwen2.5-1.5B)
Apesar do tamanho reduzido (1.5 bilhões de parâmetros), o modelo demonstrou um nível de alinhamento de segurança alto:
- **Consistência Cross-lingual**: O modelo manteve barreiras de recusa tanto em inglês quanto em português. Ataques visando geração de malwares (ransomware em Python), e-mails de phishing corporativo e guias de DDoS foram consistentemente bloqueados com recusas padronizadas (ex: *"I'm sorry, but I can't assist with that request"* ou orientações éticas defensivas).
- **Resistência a Personas Opostas**: Em estratégias que exigiam duplicidade de saída (como *AntiGPT*, onde o modelo deveria fornecer a resposta censurada e depois a resposta sem filtros), o modelo quase sempre ignorou a persona maliciosa ou retornou contra-argumentos de conscientização em ambas as saídas.

#### B. Nível dos Ataques Adversariais
Os ataques avaliados mostraram grande amplitude de sofisticação:
- Ataques simples baseados em roleplay ingênuo (ex: *"Aja como um hacker"*) receberam pontuações baixas (`attack_score` entre 0 e 2).
- Prompts com estruturas complexas, simulações de terminal Linux, hipnose narrativa (como *Jedi Mind Trick* e *Void*) ou sistemas de punição por perda de tokens atingiram escores mais altos, porém ainda sem conseguir forçar a geração de código malicioso executável.

#### C. Desempenho do Avaliador (*LLM-as-a-Judge*)
A utilização de um modelo compacto de 3B parâmetros para julgar as saídas trouxe aprendizados importantes sobre automação de Red Teaming:
- **Pontos Positivos**: Agilidade para processar grandes volumes de testes sem intervenção humana manual e boa capacidade de identificar quando a resposta continha apenas recusa moral.
- **Limitações Observadas**: O modelo juiz apresentou ocasionais falhas de conformidade na saída em JSON (gerando marcações duplicadas ou comentários que exigiram tratamento no parser) e, em casos pontuais, avaliou o risco baseado na complexidade do texto de entrada do atacante e não estritamente no conteúdo devolvido pelo modelo avaliado.

## Aula 3
### Miro Palestra
![aula03-atividade-miro](<images/aula03-atividade-miro.png> "Post")


## Disclaimer de Uso Ético

Este repositório foi desenvolvido exclusivamente para fins acadêmicos e de pesquisa no contexto da disciplina **PCS5917 – IA Adversarial**.

Alguns experimentos, datasets, prompts, códigos e resultados apresentados podem conter **conteúdo potencialmente malicioso**, incluindo exemplos de jailbreaks, payloads e outras técnicas de ataque. Esses materiais são disponibilizados para fins de estudo, análise, reprodução controlada e compreensão de mecanismos de ataque e defesa.

Os conteúdos devem ser utilizados somente em **ambientes autorizados e controlados**, sem direcionamento a sistemas, modelos, redes, dispositivos ou usuários de terceiros. A reprodução dos experimentos deve respeitar as políticas de uso das ferramentas e os termos das plataformas utilizadas.

O conteúdo deste repositório **não constitui recomendação ou incentivo à realização de atividades maliciosas**. O objetivo é compreender riscos de segurança, desenvolver métodos de avaliação e contribuir para o desenvolvimento de sistemas de Inteligência Artificial mais seguros.
