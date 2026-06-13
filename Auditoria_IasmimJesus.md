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

**Evidência (print rastreável):** Merge Request `!36`, commit `5069ea7a` (`docs(assessment): alinha evidências e referências pós-feedback`), parent `d0f23345`.  
Link do commit: https://git.inteli.edu.br/graduacao/2026-1b/t13/g06/-/commit/5069ea7aee0c45e4f8b234f7396b43ad6be94190  
Link do print: https://drive.google.com/file/d/1HMdA3DxbjJO_V7R2SaqjUP_WNaUZJZka/view?usp=sharing

**Processo analisado:** delimitação de escopo na documentação do projeto após feedback avaliativo. A decisão central era separar o que era **observado do parceiro Jacto**, o que estava **implementado no protótipo acadêmico** e o que era **recomendação futura** (Jira, integrações externas, ambientes de maior fidelidade).

**Evidência:** o commit passou no Pipeline `#23960` com 9 checks verdes. Foram 6 arquivos alterados, 84 adições e 48 deleções. O que mudou de fato:

No `README.md`, links que apontavam para `/docs/...` (quebrados no GitLab) viraram `./docs/...`. Foi criado o `docs/index.md` como entrada única da documentação, deixando explícito que Blue Pen Pipe e Blueprint são nomes do projeto acadêmico, não tecnologias do parceiro.

No `docs/documentacao.md`, entrou a seção **"Base de Evidências e Limitações"**, separando três categorias: observado/fornecido pelo parceiro, implementado no protótipo, recomendado como evolução. O mesmo arquivo corrigiu ambiguidades que confundiam avaliador: trechos que tratavam Jira como se já existisse foram reescritos para GitLab Issues no protótipo e Jira como evolução futura; "GitLab Actions" virou "GitLab CI"; um parágrafo contraditório dizia que na Sprint 2 "não há uso de GitLab" quando o fluxo real é GitLab-first.

No `docs/dev-rules.md` e `docs/gestao-configuracao.md`, a API FastAPI deixou de ser descrita como stack do parceiro e passou a ser tratada como mock do protótipo. No `docs/gestao-projeto.md`, nomes inconsistentes ("BP Project", "BP Group") foram padronizados para Blue Pen Pipe e Blueprint.

**Leitura REACH (Clarity & Results):** em Clarity, o problema antes do MR !36 não era opinião visual. Era confusão de escopo: o avaliador recebia um documento que misturava proposta de arquitetura com o que já rodava no repo, alternava Jira e GitLab Issues como se fossem a mesma coisa, e tinha links que não abriam. Isso compromete a leitura de valor e prioridades. O DesignOps ajudou porque a correção não ficou no WhatsApp: entrou em branch, MR, pipeline e merge. A delimitação de escopo ficou rastreável, não subjetiva.

Em Results, o que dá pra medir é interno: pipeline verde, documentação navegável, referências corrigidas (IEEE Std 828-2012, Conventional Commits 1.0.0), ciclo feedback → branch `fix/corrige-artefato-pos-feedback` → MR → merge. Isso prova que o processo fechou a malha documental. Não prova, sozinho, que a Jacto validou a mudança depois.

**Causa raiz:** documentação escrita sprint a sprint, por vários autores, sem revisão final de consistência antes da entrega. Faltou um checklist simples: links funcionam no GitLab? protótipo está separado de recomendação? nomes e ferramentas batem com o que o repo realmente usa?

**Limite desta evidência:** o MR !36 comprova resposta estruturada ao feedback avaliativo. Nas evidências disponíveis, não há registro escrito de validação da parceira Jacto após a correção. Então o eixo Results ("a experiência melhorou para quem lê e decide?") fica comprovado para o avaliador do módulo, mas sem prova direta de aceite do parceiro.

**Conclusão:** a Clarity melhorou porque o commit `5069ea7a` transformou ambiguidade de escopo em texto delimitado e navegável. O DesignOps cumpriu o papel de dar evidência rastreável à correção. Para fechar Results de ponta a ponta, faltaria um artefato de validação da parceira registrado no mesmo fluxo (issue, ata ou comentário formal no MR).

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
