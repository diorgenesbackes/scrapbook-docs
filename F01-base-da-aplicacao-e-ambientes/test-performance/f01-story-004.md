# F01-STORY-004 · Medição do tempo de recuperação

A medição do tempo de restauração em homologação está bloqueada. Ela depende de uma publicação bem-sucedida identificável, artefatos recuperáveis e uma janela de ensaio autorizada. O tempo da suíte Compose mede uma regressão local e não representa o tempo de recuperação do ECS, ALB e Amplify.

No ensaio de homologação, registrar o início da indisponibilidade e o primeiro instante em que a versão esperada esteja ativa com prontidão no ALB e a UI sirva o manifesto e o shell esperados. Conferir a integridade dos dados sintéticos no cenário aplicável. A meta de até dez minutos vale para os cenários cobertos no PRD. [Critérios do produto](../../PRD.md#9-critérios-de-aceitação) · [Ordem de recuperação](../delivery/f01-story-004-recuperacao.md)
