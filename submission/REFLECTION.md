# Reflection — Day 17 (≤ 200 words)

Answer briefly, in your own words. This is graded on reasoning, not length.

1. **The flywheel.** Day 13 emitted agent traces; today you turned them into an
   eval set and DPO pairs that Day 22 will train on. Which step in
   `traces → Bronze → datasets` would break most silently in production if you
   got it wrong — and how would you detect it?

2. **Decontamination.** Your run dropped 2 of 3 preference pairs because their
   prompts were in the eval set. What concretely goes wrong if you *skip* this
   step and train on those pairs? How would the lie show up in your metrics?

3. **Point-in-time.** The naive join leaked a future `lifetime_spend` into the
   training row. Describe one feature in a system you know that would be
   dangerous to join without an `ASOF`/point-in-time guard.

4. **Graph vs vector.** From `kg_demo.py`, name one question the knowledge graph
   answers well that flat chunk retrieval (`embed.py`) would struggle with, and
   one where the graph is overkill.

_Write your answers below._

1. The most silent failure is the trace-to-dataset curation step, especially mapping root-span `user_input` and `agent_output` into eval rows and preference pairs. The pipeline may still run, but the generated datasets would be wrong or incomplete. I would detect it with row-count checks, per-status distributions, and sampled records that compare raw traces against final JSONL outputs.

2. If I skip decontamination, I would train on prompts that also appear in the eval set. Then the model could memorize those answers, so eval would look better than real performance. The lie would show up as unusually high offline eval scores while production quality, generalization, or failure rate does not improve accordingly.

3. In an e-commerce system, `lifetime_spend` or `total_orders` is dangerous without a point-in-time guard. If I join the latest value instead of the value known at event time, the model sees future purchases during training and gets unrealistic accuracy.

4. The graph answers multi-hop questions well, such as "Where does a widget ship from?" because it can connect `widget -> accessory -> hanoi fulfillment center`. Flat chunk retrieval struggles when the facts are split across chunks. The graph is overkill for a direct fact like "What is the gadget warranty?"
