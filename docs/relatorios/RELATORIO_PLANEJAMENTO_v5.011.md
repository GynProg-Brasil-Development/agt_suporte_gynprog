# Relatório de Planejamento – agt_suporte_gynprog_v5.011

## Escopo
- Evoluir o agente para multimídia (áudio, imagem, vídeo, figurinhas/emoji) e resposta em voz sintetizada (persona Maurylio Honorio).
- Estruturar workflow em blocos modulares (cores) para escala de 3000+ usuários.
- Manter desenvolvimento em VPS atual (DigitalOcean) e preparar migração para VPS no Brasil para produção.

## Estado atual (v5.011)
- Meta/WhatsApp: webhook com handshake GET/POST, validação HMAC, roteamento, envio com retries.
- Normalização: sanitiza texto inbound; limita a 4k chars; marca meta/client_id.
- Contexto: busca memória em Postgres (`gynprog_support.client_memory`); gera embedding com `text-embedding-3-large`; consulta Pinecone (history/KB) mas **não** está conectado ao merge; lê “mensagens recentes” mas não persiste mensagens.
- LLM: prompt persona “Maurylio Honorio – programação automotiva (imobilizadores/EEPROM)”; responde texto.
- Memória: GPT “Atualizar Memoria” gera summary/resolved/pending/key_topics/sentiment e salva em Postgres; merge de contexto só usa memória (sem mensagens/Pinecone).
- Observabilidade: métricas no console; rate limiter é stub in-memory.

## Lacunas principais
- Sem ingestão multimídia (áudio/imagem/vídeo/figurinhas) nem TTS na saída.
- Mensagens não são persistidas em `messages`; histórico e KB não chegam ao LLM (RAG desligado na prática).
- Rate limit/cache sem Redis; sem armazenamento de mídia; sem política clara de fallback de respostas.

## Planejamento multimodal + TTS
- Entrada (bloco “Normalização/Multimídia”): baixar mídia via Graph; transcrever áudio (Whisper API ou local); gerar caption/extração para imagem/vídeo (GPT-4o mini-vision); strip de áudio de vídeo para transcrição curta; mapear emoji/figurinhas para tags básicas.
- Contexto (bloco “Contexto/RAG”): persistir mensagens em Postgres (tipo, texto derivado, mídia_id/url, duração, direction, channel); opcional tabela de mídia; indexar textos relevantes em Pinecone (`client_history` com filtro client_id; `gynprog_knowledge` com metadados de sistema/marca/modelo).
- Resposta (bloco “LLM/TTS”): LLM com persona (ajuste posterior) + recuperação de histórico/KB; TTS (OpenAI TTS `tts-1-hd` ou ElevenLabs) gerando áudio; hospedar MP3/OGG em bucket (DO Spaces/S3) e enviar como áudio no WhatsApp, com texto opcional.
- Saída (bloco “Envio Meta”): retries já existem; checar suporte a envio de mídia/voz com URL público e tipo correto.
- Observabilidade (bloco “Monitoramento”): Redis para rate limit/cache; logs estruturados; métricas de latência/erros por rota/mídia.
- Infra: habilitar queue mode do n8n; limitar concorrência em nós pesados (transcrição/visão/TTS); planejar migração para VPS Brasil (AWS/GCP/Azure/Hostinger local) após homolog.

## Decisões pendentes
- Provedor de TTS (OpenAI vs ElevenLabs) e se haverá voz custom “Maurylio”.
- Serviço de visão/caption (GPT-4o mini-vision) e limites/tamanho de mídia.
- Storage de mídia (DO Spaces/S3) e política de retenção.
- Esquema de tabelas Postgres (messages, media) e metadados obrigatórios em Pinecone (client_id, segmento, marca/modelo/ano/sistema).
- Política de resposta: quando enviar áudio, quando só texto, quando ambos; tamanho máximo de transcrição.
- Template final da persona (manutenção automotiva ampla).

## Próximos passos sugeridos
1) Escolher stack: TTS, transcrição, visão, bucket de mídia; confirmar provedores e região (para produzir URLs públicos).  
2) Definir DDL de mensagens/mídias e metadados Pinecone; habilitar inserção de mensagens (in/out).  
3) Conectar nós de contexto: mensagens recentes + Pinecone (history/KB) → Merge Contextos → LLM.  
4) Implementar blocos multimídia: download, transcrição/caption, enriquecimento de contexto.  
5) Ajustar saída: envio de áudio TTS + texto; testes E2E em homolog antes de migrar para VPS no Brasil.
