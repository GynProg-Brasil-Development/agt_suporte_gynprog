# Estrutura de pastas e nomenclatura

- `docs/`
  - `relatorios/` – relatórios de planejamento por versão (ex.: `RELATORIO_PLANEJAMENTO_v5.011.md`).
  - `changelogs/` – mudanças por versão (ex.: `CHANGELOG_v5.012.md`).
  - `analises/` – análises detalhadas, resumos e artefatos técnicos.
  - `prompts/` – instruções de persona e guias de interação (ex.: `prompt_arquiteto_codex_v1.0.md`, `CLAUDE.md`).
- `workflows/`
  - Arquivos n8n com versão padronizada em `v5.xxx` (três dígitos, ponto), ex.: `agt_suporte_gynprog_v5.011.json`.

Padrão de versão
- Workflows e documentos acompanham a mesma versão (v5.xxx).
- Evitar underscore em versões; usar ponto e três dígitos: `v5.004`, `v5.005`, …, `v5.011`.

Próximos passos recomendados
- Manter novos relatórios/changelogs na versão correspondente à próxima entrega (ex.: `v5.012`).
- Se precisar de diffs entre versões, gerar `CHANGELOG_v5.012.md` em `docs/changelogs/`.
