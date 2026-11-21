# F03-STORY-001 · Repetição segura da criação

O changeset `f03-story-001-event-creation` acrescenta `creation_key_hash` e `creation_payload_hash` à tabela `scrapbook.events`. As duas colunas são nulas nos registros anteriores. A restrição exige ambos os hashes de 32 bytes juntos. O índice parcial único combina conta criadora e hash da chave, de modo que tentativas simultâneas da mesma conta convergem no banco.

O avanço registra a assinatura da geração pela função de checkpoint da fundação. A reversão obtém lock exclusivo na tabela e recusa o rollback se houver chaves gravadas. Se puder reverter, restaura a assinatura anterior. O ensaio usa PostgreSQL 18.6 descartável, inclusive para upgrade com registros, reaplicação, reversão e preservação de tentativas já registradas. [Spec](../spec.md#evolução-para-repetição-segura)
