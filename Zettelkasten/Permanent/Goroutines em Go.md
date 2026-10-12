---
id: 202610051200
type: permanent
title: Goroutines em Go
date: 2026-10-05
status: seedling
tags:
  - go
  - concorrencia
  - zettelkasten/permanent
aliases:
  - Goroutines
  - Go routines
source: conhecimento geral
links:
  - "[[MOC Go]]"
  - "[[Concorrência vs Paralelismo]]"
---

# Goroutines em Go

**Ideia central:** goroutine é uma função executada de forma concorrente, gerenciada pelo *runtime* do Go (não pelo SO). É leve (~2 KB de stack inicial, que cresce sob demanda), então é viável criar milhares delas.

```go
go func() {
    fmt.Println("rodando concorrentemente")
}()
```

## Como o scheduler funciona (G-M-P)

![[goroutines-gmp.svg]]

- **G**: goroutine
- **M**: thread do SO
- **P**: processador lógico (quantidade = `GOMAXPROCS`); cada P tem uma fila local
- Um P ocioso pode roubar trabalho de outro (*work stealing*).

```mermaid
flowchart LR
    G1[G] --> P1[P1]
    G2[G] --> P1
    G3[G] --> P2[P2]
    P1 --> M1[M / thread]
    P2 --> M2[M / thread]
    M1 --> CPU1[(CPU)]
    M2 --> CPU2[(CPU)]
```

## Comunicação com channels

![[goroutines-channel.svg]]

```go
ch := make(chan int)
go func() { ch <- 42 }()
v := <-ch
```

```mermaid
sequenceDiagram
    participant A as goroutine A
    participant C as channel
    participant B as goroutine B
    A->>C: ch <- 42
    C->>B: v := <-ch
```

## Pontos de atenção
- `main` terminando encerra todas as goroutines; use `sync.WaitGroup` para esperar.
- Evite *data races*: use channels ou `sync.Mutex` (teste com `go run -race`).
- Goroutine sem saída = *goroutine leak*; use `context` para cancelamento.

## Relacionadas
- [[MOC Go]]
- [[Concorrência vs Paralelismo]]
