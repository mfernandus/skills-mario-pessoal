---
name: "avaliacao-modelo-skills"
description: "Depois de criar ou alterar qualquer skill, avalia se ela roda num modelo mais leve (Sonnet ou Haiku em vez de Opus) para gastar menos do plano. Também sob pedido e numa revisão semanal das skills e tarefas agendadas."
---

# Avaliação de modelo das skills

Objetivo: toda skill da Next e de Mário sai com um modelo recomendado, o mais leve que entrega o mesmo resultado, para economizar o limite do plano. Uma skill criada no Opus não precisa rodar no Opus para sempre.

## Quando rodar

- Logo depois de criar ou alterar uma skill (skill-creator, propose_skills, edição no GitHub), antes de dar a entrega por encerrada. A avaliação faz parte da entrega da skill.
- Quando Mário pedir para avaliar o modelo, o consumo ou o custo de uma skill ou de uma tarefa agendada.
- Na revisão geral ("avaliar todas as skills"), conforme a seção própria abaixo.
- Uma vez por semana, pela tarefa agendada "Revisão semanal de modelos das skills", conforme a seção Revisão semanal.

## Os níveis

Usar sempre a versão mais recente de cada nível disponível na conta. IDs em nov/2026: claude-haiku-5-5, claude-sonnet-5-5, claude-opus-5-5 e, acima do Opus, claude-fable-5-2.

- Haiku, o mais leve: tarefa mecânica de regra fixa. Conferir, copiar, sincronizar, mover, checar lista, preencher modelo com dados prontos, rotação de destaques, contagens.
- Sonnet, o padrão das rotinas: apuração em fontes indicadas pela skill, redação jornalística no padrão da casa, publicação via NX Feed, boletins, resumos, adaptações (ex.: Comunhão na ES Brasil), SEO, triagem.
- Opus (ou Fable): só quando há julgamento pesado. Síntese de muitas fontes em conflito, números a cruzar e projetar (simulação de faturamento, análises financeiras), textos longos e autorais de alto risco (Retrospectiva, cartas comerciais estratégicas), criação de código e apps, decisões editoriais sensíveis sem regra escrita.

## Como avaliar

Ler o SKILL.md inteiro e os arquivos de referência da skill. Dar nota de 0 a 2 em seis critérios:

1. Raciocínio: segue regras explícitas (0) · julga com critérios escritos (1) · julgamento aberto, sem regra (2)
2. Fontes: uma fonte fixa ou dados prontos (0) · poucas fontes indicadas (1) · muitas fontes, conflito, apuração aberta (2)
3. Texto: nenhum ou curto, em molde (0) · matéria padrão (1) · texto longo, autoral ou analítico (2)
4. Números: nenhum ou só copiar (0) · conferir e comparar (1) · cruzar, calcular, projetar (2)
5. Fluxo: poucas chamadas em linha reta (0) · fluxo médio com ramos (1) · longo, com código, muitos ramos ou recuperação de erro (2)
6. Risco do erro: interno e fácil de corrigir (0) · publicado, mas corrigível (1) · risco à reputação, jurídico, financeiro ou com cliente e parceiro (2)

Soma de 0 a 3: Haiku. De 4 a 8: Sonnet. De 9 a 12: Opus.

Travas que valem acima da soma:

- Skill que publica direto, sem revisão humana, nunca vai para o Haiku.
- Risco 2 e raciocínio 2 ao mesmo tempo: Opus.
- Skill que escreve código ou mexe em app (Lovable, Supabase, scripts): no mínimo Sonnet.
- Se só uma etapa for pesada, propor dividir a skill ou a tarefa agendada em duas: a etapa pesada no nível que ela pede, o resto no mais leve (o mesmo princípio do pacote de economia de 07/11/2026).

## Teste antes de trocar

Teste obrigatório quando a recomendação é o Haiku ou quando a soma fica a um ponto do corte. Nos outros casos, a primeira execução real no modelo novo vale como teste, com conferência pós-publicação.

- Rodar em modo ensaio: nada vai ao ar. Matéria fica em rascunho no NX Feed, sem WhatsApp, sem e-mail, sem gravar no Nextor nem no WordPress.
- Onde rodar: com a ferramenta de subagente disponível, num subagente com o modelo testado; se não, pedir a Mário que abra uma conversa nova com o modelo escolhido e chame a skill em modo ensaio.
- Conferir cada regra obrigatória da skill (ok ou falhou) e, se houver, comparar com a última saída do modelo atual.
- Aprovado: nenhuma regra obrigatória falhou e o texto tem qualidade equivalente. Reprovado: subir um nível e registrar o motivo.

## Onde registrar

1. No próprio SKILL.md, uma seção no fim (o claude.ai não guarda o campo model no cabeçalho, por isso o registro vai no corpo; a sincronização com o GitHub leva junto):

```
## Modelo recomendado
- Nível: Sonnet
- Nota: 5/12 (raciocínio 1, fontes 1, texto 1, números 0, fluxo 1, risco 1)
- Motivo: uma frase
- Teste: aprovado em dd/mm/aaaa (ou: não testado, primeira execução monitorada)
- Subir para Opus se: condição concreta
```

2. Tarefa agendada: se a skill roda numa tarefa agendada, achar a tarefa (list_triggers), mostrar o modelo atual e o recomendado e só trocar com o ok explícito de Mário nesta conversa (update_trigger, campo model). Tarefas que exigem o computador ou que ficam só no app desktop não aceitam troca de modelo por aqui: dizer a Mário para trocar nas configurações da tarefa.
3. Uso manual: dizer em qual modelo abrir a conversa para chamar a skill.

## Entrega

Mensagem curta: nível recomendado, nota em uma linha, motivo em uma frase, resultado do teste (se houve) e, quando houver tarefa agendada, a pergunta "troco o modelo da tarefa?". Não inventar percentuais de economia.

## Revisão geral

Quando pedida, montar uma tabela: skill · modelo atual (se conhecido) · recomendado · nota · teste. Ordenar pelo consumo provável, frequência vezes nível: rotinas que rodam várias vezes ao dia primeiro (Destaques, Política, Esporte, Panorama, Igreja, turno noturno). Propor as trocas em lote e aplicar só o que Mário aprovar.

## Revisão semanal

Roda toda segunda de manhã, pela tarefa agendada "Revisão semanal de modelos das skills". Objetivo: descobrir se alguma skill precisa de mudança mais séria, de modelo ou de estrutura. Só propõe; nada muda sem o ok de Mário.

1. O que mudou: listar as skills criadas ou alteradas nos últimos 7 dias, pelo histórico dos repositórios mfernandus/skills_nexteditorial e mfernandus/skills-mario-pessoal no GitHub (ou pelas datas dos arquivos de skill da sessão). Avaliar cada uma pela régua. Se a nota cruzou o corte, é mudança de nível.
2. Skills sem a seção Modelo recomendado: avaliar e propor a seção.
3. Saúde das tarefas agendadas (list_triggers): última execução com falha ou fora de SUCCEEDED, tarefa pausada sozinha. Separar falha de modelo (saída incompleta, regra obrigatória descumprida, tarefa que não termina) de falha de ferramenta ou conector, que não se resolve trocando o modelo.
4. Sinais para subir de nível: falha ligada ao modelo; correções registradas na semana (conferência pós-publicação, pedidos de Mário); skill que ganhou etapa pesada, passou a publicar direto ou a mexer em código.
5. Sinais para descer de nível: nota longe do corte, nenhuma falha nem correção na semana e teste aprovado no nível atual. No máximo duas sugestões de descida por semana, para não virar ruído.
6. Sinais para dividir: tarefa que ficou longa, com uma etapa pesada e o resto mecânico.

Mudança séria: subir de nível, dividir uma tarefa, falha repetida, skill nova rodando no Opus sem necessidade. Ajuste de nota que não muda o nível vai só no rodapé.

Relatório: mensagem curta, primeiro o que é sério. Cada item com skill, situação, proposta e a pergunta de ok. Se não houver nada sério, uma linha dizendo isso, com o número de skills e tarefas conferidas.

## Regras

- Nunca trocar o modelo de uma tarefa agendada sem o ok de Mário.
- Na dúvida entre dois níveis, recomendar o mais leve com teste; se reprovar, sobe.
- Não rebaixar de novo uma skill que já reprovou num nível, a menos que ela tenha mudado.
- Reavaliar quando a skill mudar de forma relevante: etapa nova, fonte nova, passou a publicar direto.