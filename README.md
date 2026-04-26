# cc-notas

Vault do Obsidian com notas de estudo. O ecossistema principal está em [`SO/`](./SO) — notas sobre **Sistemas Operacionais**, baseadas em grande parte no livro *Operating Systems: Three Easy Pieces* (OSTEP).

## Como usar

Pré-requisito: [Obsidian](https://obsidian.md) instalado.

```bash
git clone https://github.com/victorfrancacg/cc-notas-de-estudo.git
```

No Obsidian:

1. `Open folder as vault`
2. Selecione a pasta `cc-notas` recém-clonada
3. Pronto — o grafo, wikilinks e configurações já vêm junto

## Estrutura

- `SO/` — notas de Sistemas Operacionais (processos, scheduling, virtualização, etc.)
- `.obsidian/` — configuração do vault (versionada para portabilidade)

## Observações

- As notas usam **wikilinks** (`[[Nota]]`), sintaxe nativa do Obsidian. No GitHub aparecem como texto literal — a experiência completa (grafo, backlinks, navegação) só funciona dentro do Obsidian.
- Notas vazias são **placeholders** de conceitos a escrever.
- Atualmente, as notas de Sistemas Operacionais cobrem o **primeiro "piece" do livro** (a parte I (caps 1-24)).
