---
id: 202610161609
type: fleeting
title: "jq — processador de JSON na linha de comando"
date: 2026-10-16
status: inbox
tags: [ferramentas, cli, json, shell]
aliases: ["jq"]
source: ""
links: []
---

# jq — processador de JSON na linha de comando

## Ideia central
Ferramenta de linha de comando para **filtrar, transformar e formatar JSON**, do mesmo jeito que `sed`/`awk` tratam texto puro. Substitui o `grep`/`cut` improvisado em cima de JSON — e, diferente dele, não quebra quando o JSON tem chaves aninhadas.

Alternativa que já existe sem instalar nada: `python3 -m json.tool arquivo.json` (formata bonito, mas não filtra).

## Correção de nota mental
**O jq não é uma ferramenta GNU.** O GNU tem `grep`, `sed`, `awk`, `find`; o jq é um projeto independente — criado por **Stephen Dolan** (2012), licença **MIT**. Isso importa na hora de procurar: "a versão GNU do jq" não existe. O que existem são reimplementações: **gojq** (Go) e **jaq** (Rust, mais rápida).

## Exemplo
```bash
# pegar só os títulos dos posts de um feed (formato Blogger/Atom)
curl -s "https://exemplo.blogspot.com/feeds/posts/default?alt=json&max-results=50" \
  | jq -r '.feed.entry[].title.$t'

# montar uma listinha a partir de um JSON grande
jq -r '.feed.entry[] | "\(.published.$t[:10]) | \(.title.$t)"' feed.json
```

Filtros que cobrem quase tudo: `.campo` (acessa), `.[]` (itera), `.[] | select(.x > 10)` (filtra), `map(...)` (transforma lista), `{a: .a, b: .b}` (remonta objeto), `sort_by`, `group_by`, `length`.
Flags do dia a dia: `-r` (string crua, sem aspas), `-c` (compacto, uma linha), `-s` (slurp: junta tudo num array), `-e` (usa o resultado como **exit code**, ótimo para condicional em shell), `--arg nome valor` / `--argjson` (passar variável do shell com segurança).

## Armadilhas
- **Número grande perde precisão**: o jq trata número como float de 64 bits, então um ID de 19 dígitos pode sair arredondado. Nesses casos, ler como texto.
- **Aspas no shell**: o filtro vai sempre entre **aspas simples**. Com aspas duplas, o shell tenta expandir `$t` antes de o jq ver o filtro.
- **Valor do shell nunca concatenado no filtro**: usar `--arg`/`--argjson`.
- **`-r` é obrigatório** quando a saída vai para outro comando; sem ele, vem com aspas.
- Chave com hífen: `.["chave-com-hifen"]`.
- Não confundir com **`yq`** — é o mesmo conceito para YAML, mas outro projeto.

## Ganchos para depois
- Candidata a permanente: "tratar dados como texto estruturado no shell" — jq + yq + `python3 -m json.tool`.
- Uso real a resolver: o monitor de promoções do Hermes lê o feed de um blog e hoje faz isso com `grep -o`; com jq o mesmo trecho fica legível (ver `scripts/brindes_check.py`).
