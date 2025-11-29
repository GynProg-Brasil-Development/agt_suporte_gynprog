# Relatório v5.012 – Estrutura Multimídia + TTS

## Panorama
- Versão do n8n: 1.121.3 (compose ajustado e contêiner confirmado).
- Workflow base: `workflows/agt_suporte_gynprog_v5.012.json`.
- Objetivo: agente multimídia (áudio/imagem/vídeo/figurinhas), RAG ligado, saída em áudio (voz TTS).

## Desenho em blocos (cores sugeridas no editor)
- Azul (Ingest): Webhook Meta, validação HMAC, roteamento GET/POST.
- Ciano (Normalização): `Normalizar Mensagem` → `Enriquecer Midia`.
- Roxo (Multimídia): `If: Tem Midia` → `Meta: Obter Media URL` → `Baixar Midia (Binary)` → `Marcar Tipo Midia` → `If: Tipo Audio` → `Transcrever Audio (Whisper)` → `Aplicar Transcricao` → `Rate Limit Check` (ramo sem mídia vai direto ao rate limit). A transcrição preenche `embedding_input`.
- Verde (Contexto/RAG): `PostgreSQL: Buscar Memoria`, `PostgreSQL: Mensagens Recentes`, `Pinecone: Historico`, `Pinecone: Knowledge Base` → `Merge Contextos`.
- Laranja (LLM/TTS): `GPT-4: Gerar Resposta` → `Empacotar Resposta GPT` → `TTS: Gerar Audio (OpenAI)` → `TTS: Preparar Saida` → `Merge GPT + Contexto`.
- Amarelo (Saída): `Preparar Resposta` → `Meta: Enviar Mensagem` → `Calcular Latencia` → `Registrar Metricas`.
- Cinza (Observabilidade): `Rate Limit Check` (stub), `Error Handler`/logs.

## Ajustes implementados
- Ingestão multimídia inicial: download de mídia via Graph, marcação de tipo, transcrição Whisper para áudio; fallback sem mídia passa direto.
- Persistência: `PostgreSQL: Salvar Mensagem Inbound` grava client_id/conteúdo/derivação e metadados de mídia.
- RAG: mensagens recentes + Pinecone (history/KB) agora entram no `Merge Contextos`.
- TTS: pipeline OpenAI TTS integrado; `Preparar Resposta` envia áudio quando há `tts_audio_url`, ou texto/template.
- Embeddings: usam `embedding_input` (prioriza transcrição/caption quando disponível).

## Próximos passos sugeridos
- Completar caption/visão para imagem/vídeo (GPT-4o-mini-vision) e alimentar `embedding_input`.
- Upload real do áudio TTS para bucket/CDN (DO Spaces/S3) e setar `tts_audio_url`.
- Validar DDL de `gynprog_support.messages` e ajustar se necessário (colunas: client_id, content, derived_text, direction, channel, message_type, message_id, media_type, media_id, timestamp).
- Refinar rate limit (Redis) e remover chave `version:` obsoleta do docker-compose.
