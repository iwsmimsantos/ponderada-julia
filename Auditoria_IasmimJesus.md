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

**Evidência (print rastreável):** conversa do grupo "Módulo 10 G06" sobre decisões de processo e alinhamento do projeto.  
Link: https://drive.google.com/file/d/1p_sut-4q4foB4YKSxDtQ1XfM_lwMvQpJ/view?usp=sharing

**Processo analisado:** discussão do grupo durante o planejamento da **Sprint 3**, focada na **configuração da esteira de CD**, principalmente sobre manter ou adaptar o fluxo de commits que vinha sendo usado até então.

**Evidência:** o print é de uma conversa no WhatsApp enquanto a Sprint 3 ainda estava sendo organizada. Às 10:08 aparece a mensagem: *"nn sei se a gente vai continuar com o esquema de commit"*. O timing dessa fala é o que chama atenção. Isso aconteceu justamente quando a sprint estava entrando numa parte mais técnica, de configuração da esteira de CD, onde versionamento e rastreabilidade começam a sair do discurso e passam a impactar a implementação de verdade.

No grupo, o "esquema de commit" tinha uma lógica mais organizada. A ideia era uma pessoa fazer sua parte, subir as alterações, depois a próxima continuar em cima daquilo, seguindo uma ordem para diminuir conflito e facilitar rastreamento. No começo parecia fazer bastante sentido, principalmente porque ajudava a entender quem mexeu em quê e evitava gente alterando as mesmas coisas ao mesmo tempo.

Só que o print mostra um detalhe que a documentação não mostra: no meio do caminho o grupo começou a renegociar esse combinado.

A resposta da Nataly deixa isso claro. Ela fala *"eu acho top"* e logo depois *"teve bons frutos"*, mas completa dizendo que apoiava o grupo decidir junto o que continuaria sendo feito. Não parece alguém defendendo um processo rígido. Parece mais um "funcionou, mas talvez precise ajustar".

Depois entra outra camada na conversa. Às 10:32, ela comenta que o Hermano passou um bom tempo perguntando se o grupo ia usar algo dos repositórios preparados nas aulas. Ela escreve: *"o hermano disse q seria mt bom usar algo dos repositórios q ele preparou"* e depois *"acho q o Romualdo tb tá esperando isso"*. Minutos depois, a Milena manda no grupo o repositório `https://github.com/milenacasttro/inteli-px4-cicd-demo`.

Isso muda um pouco o cenário da sprint. A discussão já não era só configurar a esteira de CD. Surgiu também uma preocupação de mostrar alinhamento com o que tinha sido ensinado no módulo. Então, ao mesmo tempo que o grupo precisava implementar uma parte técnica do projeto, começou também uma conversa sobre o quanto valia manter certos rituais e o quanto fazia sentido incorporar exemplos das aulas.

**Leitura REACH (Health):** pelo eixo de **Health** do REACH, acho que o problema aparece exatamente aí. Não parece falta de conhecimento técnico, porque ninguém está dizendo "não sabemos fazer". Também não parece desinteresse. O que aparece é um grupo tentando equilibrar coordenação e velocidade.

O fluxo de commits é um exemplo bom disso. Fazer tudo em sequência ajudava na organização, mas também criava dependência. Às vezes alguém não conseguia entrar no mesmo horário, outra pessoa já tinha mexido numa parte relacionada, ou a sprint simplesmente precisava andar mais rápido. Como cada integrante tinha rotina e matérias diferentes, esse tipo de coordenação começou a ficar mais difícil de sustentar do que parecia quando o plano foi escrito.

**Causa raiz:** olhando hoje, acho que nosso erro foi ter planejado um processo muito bonito no papel. Quando escrevemos o plano de DesignOps, várias práticas pareciam encaixar bem juntas: commit organizado, rastreabilidade, reaproveitamento dos repositórios das aulas, pipeline, documentação. Só que a Sprint 3 foi um momento em que isso começou a bater na realidade do tempo do grupo.

**O que deveria ter sido cortado:** se eu tivesse que cortar alguma coisa, provavelmente deixaria o fluxo de commit mais flexível. Não abandonaria totalmente, porque realmente ajudou e a própria Nataly fala que *"teve bons frutos"*. Mas talvez uma ordem tão rígida entre pessoas não fizesse sentido numa sprint já pesada por si só. Em alguns momentos, organizar demais começou a competir com o tempo de fazer.

**Conclusão:** o print mostra um processo que funcionou no começo, mas que começou a pesar justamente quando a Sprint 3 exigiu implementação técnica e alinhamento com o módulo ao mesmo tempo. Health, aqui, não falhou por falta de esforço. Falhou porque o plano não foi desenhado pra caber no tempo real do grupo.
