# CHANGELOG v5.012 (planejamento multimídia + TTS)

## Resumo
- Estrutura do repositório padronizada (`docs/*`, `workflows/`) com versão `v5.xxx`.
- Workflow base `agt_suporte_gynprog_v5.012.json` criado para evoluções multimídia + TTS + RAG.

## Implementado na v5.012
- **Ingestão multimídia (stub inicial)**: novo nó `Enriquecer Midia` prepara `embedding_input`, media_type/id/url para áudio/imagem/vídeo/figurinhas.
- **Persistência**: nó `PostgreSQL: Salvar Mensagem Inbound` grava mensagens (client_id, content, derived_text, media info).
- **RAG ligado**: conexões ativas para `PostgreSQL: Mensagens Recentes`, `Pinecone: Historico`, `Pinecone: Knowledge Base` → `Merge Contextos` (inputs 1/2/3).
- **TTS**: pipeline `Empacotar Resposta GPT` → `TTS: Gerar Audio (OpenAI)` → `TTS: Preparar Saida` → `Merge GPT + Contexto`; `Preparar Resposta` agora envia áudio (quando `tts_audio_url` disponível) ou texto/template.
- **Embeddings**: uso de `embedding_input` prioriza transcrição/caption quando existir.
- **Infra**: n8n atualizado para `1.121.3` (compose ajustado e contêiner confirmando versão nos logs).
- **Bloco multimídia**: cadeia `If: Tem Midia` → `Meta: Obter Media URL` → `Baixar Midia (Binary)` → `Marcar Tipo Midia` → `If: Tipo Audio` → `Transcrever Audio (Whisper)` → `Aplicar Transcricao` → `Rate Limit Check` (ramo “sem mídia” vai direto ao rate limit). Transcrição alimenta `embedding_input`.

## Desenho dos blocos (v5.012)
- Ingest (azul): Webhook Meta, validação HMAC, roteamento GET/POST.
- Normalização (ciano): `Normalizar Mensagem` → `Enriquecer Midia` → checagem de mídia.
- Multimídia (roxo): obtenção/ download de mídia, marcação de tipo, transcrição Whisper para áudio.
- Contexto/RAG (verde): Postgres memória + mensagens recentes, Pinecone history/KB → `Merge Contextos`.
- LLM/TTS (laranja): `GPT-4: Gerar Resposta`, TTS OpenAI, merge com contexto.
- Saída Meta (amarelo): `Preparar Resposta` → `Meta: Enviar Mensagem` → métricas.
- Observabilidade (cinza): rate limit, métricas, error handler.

## Próximos passos
- Integrar download real de mídia + transcrição (Whisper) e caption (vision) alimentando `embedding_input`.
- Implementar upload de áudio TTS para CDN/Spaces e popular `tts_audio_url` com link real.
- Ajustar DDL de mensagens/mídia e validar consultas/índices para histórico e KB.
