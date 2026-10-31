# nullmodem

Research on how agents talk to each other below the level of text.

A null modem cable linked two PCs directly, with no network in between. This repo asks what agents can exchange when the harness owns the model, the tokenizer and the machine, and doesn't have to pass everything through text.

## Why

Agents today talk in tokens because hosted models only accept tokens. [Exo](https://github.com/Moissanite-Labs/exo) running on [exo-os](https://github.com/Moissanite-Labs/exo-os) changes that: the harness and a local model work together at the root, down to memory, caches and the tokenizer. Agents running that way across several devices could share far more than messages.

## What lives here

- Research notes and design write-ups.
- Small experiments on local open-weight models, such as sharing a KV cache between agents, passing embeddings or other latent messages, sharing a tokenizer, and keeping memory in sync across devices.
- Measurements against a text baseline (plain English and ABIOS), with raw logs and rerun commands.

Ideas that prove out here move into exo-os and [exyber](https://github.com/Moissanite-Labs/exyber), one profile first and then outward after verification.

See [CHARTER.md](CHARTER.md) for scope, questions and success criteria.

## Related

- [abios](https://github.com/Moissanite-Labs/abios): the text-level message protocol.
- [tokenpage](https://github.com/Moissanite-Labs/tokenpage): tokenizer research.

## License

MIT
