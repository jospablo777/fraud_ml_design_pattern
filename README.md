# Designing fraud-prevention systems that keep analysts in the loop

Companion repository for the [Nerdearla Chile 2026](https://nerdearla.com/chile/) talk **[Fraud prevention, machine learning, and design patterns: keep your analysts in the loop](https://nerdearla.com/chile/schedule/prevencion-de-fraude-machine-learning-y-patrones-de-diseno-manten-a-tus-analistas-en-el-loop/)**, by [José P. Barrantes](https://www.linkedin.com/in/jose-barrantes/).

A conference slot is short. It fits the intuition but not the architecture background, the figures, the trade-offs, or the bibliography. So the material here is the longer version of the same argument:

| Format | Link |
| --- | --- |
| Technical article, web | [jospablo777.github.io/fraud_ml_design_pattern](https://jospablo777.github.io/fraud_ml_design_pattern/technical_article.html) |
| Technical article, PDF | [communication/LaTeX/technical_article.pdf](communication/LaTeX/technical_article.pdf) |
| Slides, PDF | [Nerdearla-fraud_prevention_ML_and_architecture.pdf](Nerdearla-fraud_prevention_ML_and_architecture.pdf) |
| Talk recording | [Nerdflix](https://nerdearla.com/nerdflix/tHTcBGFZgZ4/) |

## The argument in one paragraph

Fraud prevention gets treated as a classification problem, and it goes better when you treat it as a **software-architecture problem**. The model is one component among several that are all load-bearing: deterministic rules, a risk score, a policy layer that decides what to do with that score, expert analysts who take the cases nobody should automate, and the platform work that keeps all of it observable. Ship a strong classifier into a system with no policy layer, no queue design, and no way to capture what the analysts learned, and you have a good model inside a system that can't act on it.

![Rules, machine learning, and analysts drawn as three interlocking pillars of one fraud system](img/fraud_system_three_pillars.png)

**Fig. 1.** Three capabilities, one system. Rules encode what the organization already knows. Machine learning compresses weak signals into a ranking. Analysts supply judgment on the cases in between. The interesting design question is how they interact.

## Who this is for

Fraud, risk, and trust-and-safety teams shipping ML into production. Data scientists who have a working model and are now discovering that the hard part is everything around it. Software architects being handed an ML component. And anyone building high-stakes decision systems outside fraud, since most of this transfers: alert triage in a SOC, content moderation, credit adjudication, claims review.

## Why fraud resists a pure-model solution

Four properties of the domain push the problem out of the model and into the architecture.

**Attackers adapt.** Once a defense becomes known, the behavior changes. A fraud model faces a moving opponent, so yesterday's decision boundary decays on purpose, not by accident.

**Labels arrive late and dirty.** Ground truth often shows up weeks later, when a customer disputes a charge or an investigation closes. Some fraud is never reported at all, which means it sits in your training data labeled as legitimate. And "fraud" is not one phenomenon; collapsing account takeover, friendly fraud, and mule activity into a single positive class throws away structure you could have used.

**The two errors cost different things.** A miss is direct loss. A false positive is a blocked customer, a support ticket, and some churn risk. No single threshold is right for both, and the exchange rate between them is a business decision, not a modeling one.

**Timing is part of correctness.** A decision that arrives after the money moved is a correct answer that failed. Some cases have to be resolved in milliseconds, which decides what can be automated and what can afford to wait for a human.

Put together, these are the reasons that a fraud system is a layered control structure, part automatic and part human-supervised, rather than a scoring endpoint.

## Separate the score from the policy

This is the load-bearing idea of the whole pattern, and it's one line: **a model produces an estimate, a policy layer decides what to do about it.**

![Pipeline showing a risk score feeding a policy layer, which then selects an action](img/score_policy_action.png)

**Fig. 2.** Score, then policy, then action. Three steps, deliberately not two.

Collapsing those steps is tempting because it saves a service. It also buries your governance inside model weights. Real operating decisions have to account for analyst saturation on a Monday morning, a regional restriction, a promo campaign that spikes traffic, an SLA, and a guardrail that legal will ask about later. None of that belongs in the loss function. Keeping policy in its own layer gives you something you can read, version, audit, and change on Tuesday without retraining anything.

## The reference architecture

With score and policy separated, the runtime shape gets easy to describe.

![Reference architecture: rules filter, ML scores, a policy layer routes cases to approval, analyst review, or mitigation](img/hil_triage_reference_architecture.png)

**Fig. 3.** ML-enabled human-in-the-loop triage. Rules sit near the entrance and dispatch the explicit cases. Machine learning scores what's left. Policy maps score to action: approve the clearly safe traffic, mitigate the clearly hostile traffic, and send the ambiguous middle to people.

Each component does what it's actually good at. Rules are precise and auditable, and brittle the moment reality moves. Models generalize across weak signals, and can't be cross-examined about why. Analysts read context and catch novelty, and there are only so many of them. The architecture is the arrangement that lets each cover for the others' failure mode.

## Analysts are a runtime component

The most common way to get this wrong is to draw the analysts as a box labeled "manual review" hanging off the side of the diagram. They aren't an overflow bucket. They're the part of the system that handles novelty, and they're the only part that generates new knowledge.

![Closed-loop architecture where analyst outcomes feed back into rules and model improvement](img/hil_triage_feedback_architecture.png)

**Fig. 4.** The return paths are the point. Analyst outcomes should update rules, thresholds, labels, and routing logic. Note the yellow arrows from the automatic branches: analysts should also audit samples of what the system decided on its own, which is the only way anyone finds out about the false negatives.

The traffic runs both directions. The model earns its place by pointing scarce attention at the cases that repay it. Analysts earn theirs by adjudicating, escalating, and feeding structured outcomes back. Miss the second direction and you've built a one-way automation pipeline that gets slowly worse while reporting the same accuracy.

Which makes the review interface an architecture concern, not a UI afterthought.

![Analyst queue mockup with ranked cases, risk bands, reason codes, SLA timers, and structured feedback fields](img/analyst_queue_ui_mockup.png)

**Fig. 5.** An analyst workbench. Ranking policy, top contributing signals, SLA pressure, action controls, and structured outcome fields. The difference between a review queue and a learning system is whether the analyst's conclusion leaves the building as data or as a free-text note.

## Measure the system, not just the model

Accuracy is a fine number that answers a question nobody in the operation asked. The production objective is to reduce fraud loss while holding down customer friction and analyst overload, and that spans several metric families at once.

![Framework of metrics across data quality, model quality, queue operations, runtime health, and policy behavior](img/metrics_and_tradeoffs_framework.png)

**Fig. 6.** Four layers plus policy. Data and input quality, model and decision quality, queue and analyst operations, runtime and platform health. Precision at the top of the queue tends to tell you more than aggregate precision, because analyst capacity is finite and the bottom of the queue never gets read.

Approval, review, and block rates deserve special mention: those are choices the organization makes, not outputs the model happens to produce. Track them as policy metrics. And expect the frontiers, since speed trades against caution, catch rate against friction, coverage against capacity, and adaptability against auditability. Making those trades explicit is most of what architecture work is.

## Five ways this goes wrong

The failure modes are as reusable as the pattern.

**The score becomes the policy.** A raw model output triggers business action directly. Now your governance lives in a threshold constant that nobody can explain to an auditor. Give policy its own layer.

**Analysts as unstructured exception handling.** Reviewers leave prose notes, the system captures no structured outcome, and the feedback loop quietly dies. Structured fields for outcome, reason code, confidence, and suggested action are what turn review into learning.

**A great model in an unoperable system.** Months on the data science, and queue design, escalation logic, and the analyst interface get improvised in the last sprint. Alert volume climbs, analysts fatigue, the good model goes unused.

**Monitorability added after the first incident.** Fraud systems rarely fail loudly. They degrade as queue overload, rising analyst disagreement, or calibration drift, all of which are invisible unless you decided in advance to watch for them.

**Glue code and pipeline jungles.** Feature joins, a rules engine, a risk service, a case queue, an alerting layer, all accreted under delivery pressure. Fraud stacks are unusually prone to this because every incident adds a component.

## What it costs

Worth being honest about the bill. This architecture has more moving parts than a scoring service. Queue behavior needs its own monitoring. Decision logic needs versioning and an audit trail. Several teams now have to coordinate around shared system behavior, which is an organizational cost, not a technical one. The pattern framing earns its keep here: a pattern that documents only its benefits isn't a pattern, it's a pitch.

## Why call it a design pattern

Because it's a reusable answer to a recurring problem, and that's what the format is for. Naming the **context**, the **forces in tension**, the **solution**, and the **consequences** forces the trade-offs into the open where they can be argued about.

![Pattern card showing context, forces, solution, consequences, and related pattern lineage](img/pattern_context_forces_consequences.png)

**Fig. 7.** The pattern card. Context: review demand exceeds analyst capacity, some decisions are real-time, attackers adapt, errors cost different amounts. Forces: speed against caution, friction against missed fraud, autonomy against control, expertise against capacity.

No novelty is claimed for humans in the loop. The claim is narrower: *this particular* combination of rules, scoring, policy, analyst review, and closed-loop feedback recurs often enough in high-stakes decision systems to deserve writing down as a reusable architectural response.

The article develops all of this further, with five more figures (system anatomy, the co-design planning view, the operative learning loop, and the MLOps lifecycle among them), the open questions, and the full bibliography.

## What's in this repository

```
communication/
  Quarto/     Source of the web article (.qmd, references.bib, CSL styles)
  LaTeX/      Source and PDF of the article
docs/         Rendered site, published with GitHub Pages
img/          All figures as PNG, plus editable GIMP sources in img/GIMP/
Nerdearla-fraud_prevention_ML_and_architecture.pdf    Slide deck
```

The `.xcf` sources are in the repo on purpose, so the figures are editable rather than just viewable.

## Reading path

Start with this README for the thesis. Open the slides for the compressed talk version. Read the [technical article](https://jospablo777.github.io/fraud_ml_design_pattern/technical_article.html) for the full argument, the citations, and the parts that needed more room.

## A few references to start with

The article carries the full bibliography. These are the ones to read first.

- Sculley, D., et al. (2015). *Hidden technical debt in machine learning systems*. **NeurIPS 28**. The origin of "the model is the small part."
- Lewis, G. A., Ozkaya, I., & Xu, X. (2021). *Software architecture challenges for ML systems*. **ICSME**. https://doi.org/10.1109/ICSME52107.2021.00071 Where co-architecting, co-versioning, and monitorability come from.
- Kästner, C. (2025). *Machine learning in production: From models to products*. MIT Press. The best single book on treating the model as one component of a product.
- Andersen, J., & Maalej, W. (2024). *Design patterns for machine learning-based systems with humans in the loop*. **IEEE Software, 41**(4), 151-159. https://doi.org/10.1109/MS.2023.3340256 Treats human participation as a pattern concern in its own right.
- Alves, J. V., et al. (2025). *A benchmarking framework and dataset for learning to defer in human-AI decision-making*. **Scientific Data, 12**, 506. https://doi.org/10.1038/s41597-025-04664-y On deciding which cases to route to a person.
- Jalalvand, F., et al. (2024). *Alert prioritisation in security operations centres*. **ACM Computing Surveys, 57**(2). https://doi.org/10.1145/3695462 Queue design and alert fatigue, from the SOC side.
- Amershi, S., et al. (2019). *Guidelines for human-AI interaction*. **CHI 2019**. https://doi.org/10.1145/3290605.3300233 What the workbench owes the analyst.
- Chen, C., et al. (2022). *Reliable machine learning: Applying SRE principles to ML in production*. O'Reilly. For the monitoring and reliability half of the story.

## Using this material

Take the figures, the diagrams, the pattern, whatever is useful. Adapt it to your own decks, docs, and systems. Make it yours! 😃

No need to cite, but it would mean a lot if you did:

> Barrantes, J. P. (2026). *Designing fraud-prevention systems that keep analysts in the loop*. Nerdearla Chile 2026. https://github.com/jospablo777/fraud_ml_design_pattern

## Say hi

Questions, disagreements, or a war story about a fraud queue that went sideways are all welcome.

- LinkedIn: [José Pablo Barrantes](https://www.linkedin.com/in/jose-barrantes/)
- BlueSky: [doggofan77.bsky.social](https://bsky.app/profile/doggofan77.bsky.social)

Released under the [MIT License](LICENSE).
