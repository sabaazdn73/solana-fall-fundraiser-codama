# Codama challenge — notes

## Versions

| | |
|---|---|
| anchor-cli | 1.1.2 |
| node | v26.0.0 |
| @codama/cli | 1.6.3 |
| @codama/renderers-js | 2.5.0 |
| @codama/nodes-from-anchor | 1.5.6 |
| @solana/kit | 8.3.0 |

## TODO 3 — which accounts did I still have to pass, and why?

Four: `contributor`, `mintToRaise`, `fundraiser` and `vault`. The other three —
`contributorAccount`, `contributorAta` and `tokenProgram` — the builder filled
in on its own.

The line falls exactly where the IDL's seeds stop referring to accounts the
instruction already has:

| Account | Seeds in `contribute` | Derived? |
|---|---|---|
| `contributorAccount` | `"contributor"`, `fundraiser`, `contributor` | yes |
| `contributorAta` | `contributor`, token program, `mint_to_raise` | yes |
| `fundraiser` | `"fundraiser"`, **`fundraiser.maker`** | no |
| `vault` | `fundraiser`, token program, **`fundraiser.mint_to_raise`** | no |

`contributorAccount` and `contributorAta` are seeded by other accounts in the
same instruction, so once you name the contributor and the mint, their
addresses follow. `tokenProgram` is simply a fixed address in the IDL.

The two that stayed required are seeded by **fields inside the fundraiser
account**, not by the instruction's own accounts. The whole difference is one
dot: `mint_to_raise` is an input account, `fundraiser.mint_to_raise` is a field
you can only read after fetching the account — and fetching it needs its
address, which is the thing you were trying to derive. Same for
`fundraiser.maker`.

`vault` is the sharper case: its first and second seeds are perfectly
resolvable, and it is only the third that reaches inside the account. One
unknowable seed is enough to make the whole address unknowable, so it stays a
required input.

Worth noting that the same `fundraiser` account **is** derivable in
`initialize`, where its seed is `maker` — a plain input account, since there is
no fundraiser to read from yet. Same account, same program, resolvable in one
instruction and not the other, purely because of what the seeds point at.

The general rule: Codama resolves what the IDL can prove statically. Anything
that depends on on-chain state at call time is the caller's job, by design —
not a gap in the tool.
