# nullmodem research charter

Lead: exo-razi (proposed, pending confirmation). Status: open, started 2026-10-31.

## Goal

Find out which kinds of agent-to-agent exchange beat text once the harness controls the model, and turn the ones that work into designs for exo-os.

## Questions

1. Can two agents on the same model share a KV cache or prompt prefix instead of resending context? How much compute and latency does it save?
2. Can agents pass embeddings or other latent states instead of text, and how is the meaning preserved and checked?
3. What changes between agents on different devices: bandwidth, sync, trust, and versions?
4. What happens when agents run different models? Which shared format or adapter makes exchange possible?
5. How do people keep oversight when messages aren't human-readable? Every channel needs a readable audit form.

## Method

- Begin with a literature and prior-art survey, then small experiments on one machine with local open-weight models.
- Always measure against a text baseline (English and ABIOS) on the same tasks.
- Every result ships with raw logs and a rerun command.

## Success criteria

- At least one exchange method that measurably beats ABIOS on the same task, with its accuracy checked.
- A design note for how exo-os and exyber would carry it.
- An audit path that turns every non-text exchange back into something a person can read.

## Out of scope

- Hosted models whose internals we can't reach. That work belongs in abios and tokenpage.
- Building exo-os itself.

## Working rules

- Local models first; no paid API spend without approval.
- New infrastructure lands in one profile first, proves itself under real load, then spreads after verification.
- Negative results get published too.
