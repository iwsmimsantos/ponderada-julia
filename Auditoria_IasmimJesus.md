# Auditoria DesignOps — Blue Pen Pipe (G06)

## Parte 1 – Efficiency & Ability

**Evidência (print rastreável):** Pipeline `#38322`, job
`aggregate_suite #46769`
Link da evidência:
https://drive.google.com/file/d/13IA-nxbRm0hcazo8O7tHwGUQprm_3j5N/view?usp=sharing

**Processo analisado:** execução do script de validação de artefatos
(`scripts/ci/validate_artifacts.sh`) no estágio `analyze` da esteira
GitLab CI, job `aggregate_suite` #46769.

**Evidência:** o job foi disparado por Nicolas Ramon da Silva a partir
do commit `4ef60c33` (`feat(suite): agrega resultados da suite CD
[TASK-0408]`), na branch `feat/TASK-0407/quality-gates-suite`,
Pipeline #38322. Após instalar todas as dependências com sucesso
(`pip install -r requirements/base.txt`, `pip install -e .`) e baixar
artefatos dos jobs anteriores (`ci_fast_contracts` #46768,
`python_quality` #46767, `validate_commit` #46766), o job falhou na
linha:

    $ scripts/ci/validate_artifacts.sh --require-file metadata.json
    /usr/bin/bash: line 183: scripts/ci/validate_artifacts.sh: Permission denied
    ERROR: Job failed: exit code 1

A causa raiz é objetiva: o arquivo `scripts/ci/validate_artifacts.sh`
foi commitado sem permissão de execução. O histórico Git confirma que
o arquivo existe desde o commit `622a126` (`ci(gitlab): define pipeline
cd px4 gazebo [TASK-0311]`) e nunca recebeu `chmod +x`. A verificação
local do repositório confirma o mesmo estado: `-rw-r--r--`.

**Leitura crítica segundo REACH (Efficiency & Ability):** no eixo de
Efficiency, o gargalo não estava na lógica do processo, mas em um
detalhe de permissão de sistema de arquivos que bloqueou o estágio
final de uma cadeia de 23 segundos de execução bem-sucedida. Todo o
ambiente foi provisionado corretamente; o único ponto de falha foi a
ausência de `chmod +x` no commit do script. Do ponto de vista de
Ability, o time demonstrou capacidade de modelar e sequenciar o
pipeline (os três jobs anteriores passaram e entregaram artefatos),
mas falhou em uma verificação elementar de permissões — um indicador
de que o processo de revisão de PR não incluía checklist de
executabilidade para scripts de CI.

**Causa raiz:** o script foi criado e commitado sem que nenhum
membro do time verificasse se o Git preservava o bit de execução.
Não foi erro de conhecimento técnico profundo — foi ausência de um
critério mínimo no fluxo de revisão do MR.

**Como o time contornou:** a branch permaneceu isolada. O pipeline
principal em `develop` e `main` não foi bloqueado. O erro ficou
contido na branch de feature, o que demonstra que a separação de
ambientes funcionou como mecanismo de proteção — mas também significa
que o artefato de agregação da suite nunca chegou a ser validado
naquele ciclo.

**Conclusão:** a Ability do time de construir o processo estava
presente; a Efficiency foi significativamente comprometida por uma ausência simples no commit. Um hook de pre-push verificando permissões em `scripts/ci/*.sh`
teria evitado o problema sem custo operacional algum.

---

## Parte 2 – Clarity & Results

**Evidência (print rastreável):** Merge Request `!36`,
commit `5069ea7a`
(`docs(assessment): alinha evidências e referências pós-feedback`)
Link da evidência:
https://drive.google.com/file/d/1HMdA3DxbjJO_V7R2SaqjUP_WNaUZJZka/view?usp=sharing

**Processo analisado:** ciclo de resposta a feedback avaliativo
via documentação versionada no GitLab (MR !36).

**Evidência:** o commit `5069ea7a` (`docs(assessment): alinha
evidências e referências pós-feedback`) passou no Pipeline #23960
com todos os 9 checks verdes, cobrindo 6 arquivos alterados, 84
adições e 48 deleções. O diff inclui: correção de links de absolutos
para relativos (`/docs/` → `./docs/`), renomeação consistente de
"BP Project" para "Blue Pen Pipe" e "BP Group" para "Blueprint",
delimitação explícita entre protótipo acadêmico e recomendação
futura (Jira vs. GitLab Issues), criação de `docs/index.md` como
ponto único de entrada, e correção de referências bibliográficas
(IEEE Std 828-2012, Conventional Commits 1.0.0).

**Leitura crítica segundo REACH (Clarity & Results):** o MR !36
é a evidência mais direta de que a Clarity — no sentido REACH de
"parceiros externos e o time entendem o valor e as prioridades" —
estava comprometida antes da correção. Links quebrados (`/docs/`
absoluto não funciona no GitLab sem contexto de raiz), ambiguidade
entre o que era protótipo e o que era proposta de arquitetura, e
nomes inconsistentes de projeto e grupo são falhas de Clarity que
chegaram ao avaliador antes de serem corrigidas. O resultado (Results)
mensurável é o pipeline verde com 9 checks e a rastreabilidade completa
do ciclo: feedback externo → branch dedicada
(`fix/corrige-artefato-pos-feedback`) → MR → pipeline → merge.

**Causa raiz:** a documentação foi escrita ao longo das sprints por
múltiplos autores sem uma revisão final de consistência antes da
entrega avaliativa. A ausência de um checklist de entrega que incluísse
verificação de links e delimitação de escopo deixou ambiguidades que
só foram percebidas após o feedback.

**Limite desta evidência:** o MR !36 prova o ciclo interno de
resposta a feedback. Não há, nas evidências disponíveis, um registro
escrito da validação do parceiro Jacto após a correção — o que significa
que o eixo Results, no sentido de "a experiência melhorou para o
usuário final", permanece sem evidência direta nesta auditoria.

**Conclusão:** o MR !36 é o resultado mais limpo de operação em
malha fechada no repositório; a Clarity foi restaurada com
rastreabilidade ponta a ponta, mas a ausência de validação registrada
do parceiro pós-correção impede afirmar que o ciclo de Results foi
completado.

---

## Parte 3 – Health

**Evidência (print rastreável):** Conversa do grupo
“Módulo 10 G06” sobre continuidade do padrão de Conventional Commits e
expectativas do avaliador/professor quanto à integração dos repositórios
de aula.
Link da evidência:
https://drive.google.com/file/d/1p_sut-4q4foB4YKSxDtQ1XfM_lwMvQpJ/view?usp=sharing


**Processo analisado:** gestão de decisões estruturais de processo
e alinhamento com expectativas avaliativas, conduzidos via canal
informal (WhatsApp do grupo "Módulo 10 G06").

**Evidência:** na conversa capturada, o time discute abertamente se
vai manter o padrão de Conventional Commits adotado ao longo do
módulo. A frase "nn sei se a gente vai continuar com o esquema de
commit" é o indicador mais direto de desgaste: uma decisão de
arquitetura de processo — com impacto direto em rastreabilidade,
pipeline e critério de avaliação — sendo deliberada em canal sem
registro formal. Em paralelo, Nataly (monitora/Inteli) relata que
o professor passou 20 minutos questionando se o time integraria
material dos repositórios preparados para as aulas, e que o avaliador
Romualdo também espera essa integração. Milena compartilha o
repositório `https://github.com/milenacasttro/inteli-px4-cicd-demo`
como referência de resposta imediata.

**Leitura crítica segundo REACH (Health):** a saúde do processo apresenta sinais de fragilidade em dois planos simultâneos. Primeiro, o overhead
de manter Conventional Commits, branches por task, MRs rastreáveis,
pipeline multi-estágio e documentação versionada — tudo isso em paralelo
às entregas acadêmicas regulares — gerou fadiga visível: a dúvida sobre
continuar o padrão de commits não é uma questão técnica, é uma questão
de capacidade sustentável do time. Segundo, a expectativa do avaliador
chegou via conversa informal, não via critério documentado em issue ou
ADR — o que coloca o time em posição reativa, descobrindo tardiamente
o que deveria ser avaliado.

**Causa raiz:** o plano de DesignOps foi desenhado com práticas de
mercado aplicadas a um time universitário com capacidade operacional
limitada. O ritual de Conventional Commits teve bons frutos — como a
própria monitora Nataly reconhece na conversa: "teve bons frutos" —,
mas o acúmulo de rituais sem priorização gerou um ponto em que o time
questionou a sustentabilidade do conjunto.

**O que deveria ter sido cortado:** a assinatura de firmware em SITL
é o candidato mais evidente. O próprio `docs/documentacao.md` do projeto
reconhece explicitamente: "em SITL puro a verificação é tecnicamente
dispensável; o valor é manter o procedimento ativo." Um processo que
o próprio documento admite ser dispensável tecnicamente e que adiciona
complexidade operacional real não é DesignOps — é burocracia documentada
como feature.

**Conclusão:** o WhatsApp absorveu uma decisão crítica de processo que
deveria estar no GitLab; a saúde do time foi pressionada pela soma de
rituais de mercado aplicados sem filtro de realismo universitário, e o
sinal mais honesto disso está na dúvida expressa diretamente pelo time
sobre continuar o próprio processo que construiu.
