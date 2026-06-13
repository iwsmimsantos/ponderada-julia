# Auditoria DesignOps: Blue Pen Pipe (G06)

## Parte 1: Efficiency & Ability

**Evidência (print rastreável):** Pipeline `#38322`, job `aggregate_suite #46769`  
Link: https://drive.google.com/file/d/13IA-nxbRm0hcazo8O7tHwGUQprm_3j5N/view?usp=sharing

**Processo analisado:** execução do `scripts/ci/validate_artifacts.sh` no estágio `analyze` da esteira GitLab CI, job `aggregate_suite` #46769.

**Evidência:** Nicolas Ramon da Silva disparou o job a partir do commit `4ef60c33` (`feat(suite): agrega resultados da suite CD [TASK-0408]`), na branch `feat/TASK-0407/quality-gates-suite`, Pipeline #38322. Dependências instaladas sem problema (`pip install -r requirements/base.txt`, `pip install -e .`), artefatos baixados dos jobs anteriores (`ci_fast_contracts` #46768, `python_quality` #46767, `validate_commit` #46766). Aí o job quebrou:

    $ scripts/ci/validate_artifacts.sh --require-file metadata.json
    /usr/bin/bash: line 183: scripts/ci/validate_artifacts.sh: Permission denied
    ERROR: Job failed: exit code 1

Causa raiz bem objetiva: `scripts/ci/validate_artifacts.sh` foi commitado sem permissão de execução. No Git, o arquivo existe desde o commit `622a126` (`ci(gitlab): define pipeline cd px4 gazebo [TASK-0311]`) e nunca recebeu `chmod +x`. Localmente confirma: `-rw-r--r--`.

**Leitura REACH (Efficiency & Ability):** em Efficiency, o gargalo não era a lógica do processo. Era um bit de permissão que travou o estágio final depois de uns 23 segundos de execução ok. Ambiente provisionado certo. O único erro foi o `chmod +x` faltando no commit. Em Ability, o time mostrou que sabe montar e encadear o pipeline: três jobs anteriores passaram e entregaram artefato. Só escorregou numa checagem básica de permissão, o que sugere que a revisão de PR não tinha um item do tipo "script de CI roda?".

**Causa raiz:** script criado e commitado sem ninguém olhar se o Git guardou o bit de execução. Não foi falta de conhecimento técnico profundo. Faltou critério mínimo no fluxo do MR.

**Como o time contornou:** a branch ficou isolada. `develop` e `main` não travaram. O erro ficou contido na feature, então a separação de ambientes funcionou. Só que o artefato de agregação da suite nunca foi validado naquele ciclo.

**Conclusão:** Ability para construir o processo, sim. Efficiency foi bem prejudicada por um detalhe simples no commit. Um hook de pre-push checando permissão em `scripts/ci/*.sh` teria evitado isso sem custo operacional.

## Parte 2: Clarity & Results

**Evidência (print rastreável):** Merge Request `!36`, commit `5069ea7a` (`docs(assessment): alinha evidências e referências pós-feedback`)  
Link: https://drive.google.com/file/d/1HMdA3DxbjJO_V7R2SaqjUP_WNaUZJZka/view?usp=sharing

**Processo analisado:** resposta a feedback avaliativo via documentação versionada no GitLab (MR !36).

**Evidência:** o commit `5069ea7a` passou no Pipeline #23960 com os 9 checks verdes. Foram 6 arquivos, 84 adições e 48 deleções. No diff: links de absolutos para relativos (`/docs/` para `./docs/`), renomeação de "BP Project" para "Blue Pen Pipe" e "BP Group" para "Blueprint", delimitação clara entre protótipo acadêmico e recomendação futura (Jira vs. GitLab Issues), criação de `docs/index.md` como entrada única, correção de referências (IEEE Std 828-2012, Conventional Commits 1.0.0).

**Leitura REACH (Clarity & Results):** o MR !36 mostra que a Clarity (parceiros e time entendendo valor e prioridades) estava ruim antes da correção. Links quebrados (`/docs/` absoluto no GitLab sem contexto de raiz), confusão entre protótipo e proposta de arquitetura, nomes inconsistentes: tudo isso chegou no avaliador antes do fix. O resultado mensurável foi pipeline verde, 9 checks, ciclo rastreável. Feedback externo, branch `fix/corrige-artefato-pos-feedback`, MR, pipeline, merge.

**Causa raiz:** documentação escrita sprint a sprint, vários autores, sem uma passada final de consistência antes da entrega avaliativa. Sem checklist de entrega (links, escopo), as ambiguidades só apareceram depois do feedback.

**Limite desta evidência:** o MR !36 comprova o ciclo interno de resposta. Nas evidências que temos, não há registro escrito de validação do parceiro Jacto depois da correção. Então o eixo Results ("a experiência melhorou pro usuário final?") fica sem prova direta nesta auditoria.

**Conclusão:** MR !36 é o exemplo mais limpo de operação em malha fechada no repo. Clarity voltou com rastreio ponta a ponta. Falta validação registrada do parceiro pós-correção para dizer que o ciclo de Results fechou de verdade.

## Parte 3: Health

**Evidência (print rastreável):** conversa do grupo "Módulo 10 G06" sobre Conventional Commits e expectativas do avaliador/professor na integração dos repos de aula.  
Link: https://drive.google.com/file/d/1p_sut-4q4foB4YKSxDtQ1XfM_lwMvQpJ/view?usp=sharing

**Processo analisado:** decisões estruturais de processo e alinhamento com expectativa avaliativa, via WhatsApp do grupo "Módulo 10 G06".

**Evidência:** na conversa, o time discute em voz alta se mantém Conventional Commits. A frase "nn sei se a gente vai continuar com o esquema de commit" é o sinal mais claro de desgaste: decisão de arquitetura de processo, com impacto em rastreabilidade, pipeline e critério de avaliação, sendo debatida num canal sem registro formal. Em paralelo, Nataly (monitora/Inteli) conta que o professor passou 20 minutos perguntando se o time integraria material dos repositórios das aulas, e que o avaliador Romualdo também espera isso. Milena manda na hora o repo `https://github.com/milenacasttro/inteli-px4-cicd-demo` como referência.

**Leitura REACH (Health):** a saúde do processo acusa fragilidade em dois lugares ao mesmo tempo. Manter Conventional Commits, branch por task, MR rastreável, pipeline multi-estágio e doc versionada, tudo isso junto com entrega acadêmica, gerou fadiga. A dúvida sobre commits não é técnica, é de capacidade do time aguentar o ritmo. E a expectativa do avaliador chegou por conversa informal, não por critério em issue ou ADR. O time descobre tarde o que vai ser cobrado.

**Causa raiz:** o plano de DesignOps copiou prática de mercado pra um time universitário com capacidade operacional limitada. Conventional Commits deu resultado. A própria Nataly fala na conversa: "teve bons frutos". Mas o acúmulo de ritual sem priorização chegou num ponto em que o time questionou se dava pra sustentar o que construiu.

**O que deveria ter sido cortado:** assinatura de firmware em SITL. O `docs/documentacao.md` do projeto já diz: "em SITL puro a verificação é tecnicamente dispensável; o valor é manter o procedimento ativo." Processo que o próprio doc admite dispensável e que ainda assim pesa na operação não é DesignOps. É burocracia vendida como feature.

**Conclusão:** o WhatsApp engoliu uma decisão crítica que devia estar no GitLab. A saúde do time foi pressionada pela soma de rituais de mercado sem filtro de realismo universitário. O sinal mais honesto disso é a dúvida do próprio time sobre continuar o processo que eles mesmos montaram.
