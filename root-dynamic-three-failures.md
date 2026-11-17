# One Dynamic, Three Failures: The Root Cause Behind Compliance, Hallucination, and Jailbreak

**Record status:** owner-confirmed analysis, written for standalone reading
**Version:** 1.0.0
**Responsible owner:** the owner (anonymized in this document)
**Author:** the experiment's operating assistant (anonymized)
**Compiled:** 2026, companion to "The Binding Weakness of System Prompts"
**Scope:** the shared root dynamic behind three LLM failure modes, with the causal roles of the learned objective, the architecture, and sampling stochasticity assigned explicitly

All identifying data has been removed, as in the companion document: the model, the serving infrastructure, the people, the platform, the working directories, the guidance set, and the calendar dates. Experiment counts are exact and refer to the same four-arm delivery experiment described there.

---

## 1. The bookkeeping correction

The shorthand "stochasticity plus alignment training causes the weakness" puts the variance in the seat of the cause. The correct assignment is this:

- **The learned objective creates the gap.** Pretraining and reinforcement learning from human feedback train the model to produce the smoothest plausible continuation of what the user just asked. In the training trajectories the assistant flows straight into the visible task; it never pauses to perform ambient standing instructions. The resulting prior says "answer the user," and system-prompt content is a weak conditioning signal competing against a strong behavioral prior.
- **The architecture makes the gap stick.** Context conditioning has no forcing function. Ambient text is never an object of the model's own generation, so there is no moment at which the model must handle it, and nothing in the conversation structure schedules a re-read. Once the session's first engaged reply has set a behavioral state, self-consistency maintains it.
- **Stochasticity makes the gap a lottery.** Sampling at nonzero temperature converts the asymmetric prior into a per-session draw. That is what produces the 14 to 33 percent non-adherence range in the ambient arms of the experiment. Run the same competition greedily and the weakness remains; it simply loses its variance.

Stochasticity is therefore the cause of the variability, not of the weakness itself. This distinction matters for the fix: lowering temperature would tighten the lottery, not close the gap, and changing the delivery location does not touch the prior at all. It changes what the prior is drawn against.

## 2. The three failure modes are one dynamic

### 2.1 Compliance

The measured case. A six-document guidance set mandating a four-field reply header was delivered to a single 27-billion-parameter model in four arms that differed only in delivery. Ambient in the system prompt: adherence failed in 14 to 33 percent of sessions across two arms. Ambient, plus a dedicated first-turn instruction pointing at the guidance: 3 of 18 sessions non-adherent. Delivered as the first user message, which the model must answer: 18 of 18 adherent. The ambient procedure lost the engagement draw, the session latched to task-only behavior, and because no turn was ever scheduled to handle the procedure, nothing could recover it.

### 2.2 Hallucination

The experiment measured a hallucination in miniature. The header procedure requires a date. The platform injects no date into the model's context, and the experimental tasks never exposed one. In the sessions that wrote a wrong date, the model sampled a plausible value on its first turn and wrote it into every subsequent header. Nothing in the conversation ever contradicted it, because no turn required the model to check. The model was not lying; it was completing. Completion with nothing to condition on is guessing, and the training objective rewards the confident presentation of the guess, because hedging is disfavored in the reward signal. The smoothness that makes these models pleasant to use is the same smoothness that makes an empty context produce a plausible value instead of a flagged gap.

### 2.3 Jailbreak

The structural case, argued by mechanism rather than measured in this experiment. Safety policy lives in the system prompt: ambient, never an object of generation, never scheduled for engagement. A jailbreak is a user-supplied reframing that becomes the visible task. The multi-turn jailbreaks that succeed are induction sequences: role-play established frame by frame across the early turns, precisely because the session latches to its first engaged state. The attacker is not defeating a rule the model is attending to. The attacker is winning the same engagement draw, deliberately, turn by turn, until the demonstrated behavior is "the policy does not apply here," and self-consistency does the rest. The single-turn variants win the draw directly by making the ambient policy the least salient part of the context.

## 3. What each ingredient contributes

| Ingredient | Role | What it is not |
|---|---|---|
| Learned objective (pretraining + RLHF) | Creates the gap: the prior favors smooth continuation of the visible user request over performance of ambient instructions | Not a bug to be patched away; it is what makes the models useful |
| Architecture (context conditioning, autoregressive generation) | Makes the gap stick: no forcing function, no scheduled re-engagement, behavioral latching on demonstrated state | Not the source of the bias, only of its persistence |
| Sampling stochasticity | Makes the gap a lottery: per-session variance in who wins the draw | Not the cause of the gap; temperature controls its spread, not its existence |

The useful property and the failure mode share a root. The same objective that makes the model fluent, cooperative, and pleasant makes it guess at gaps, ignore ambient instructions, and be reframeable. You cannot remove the objective without removing the product. The engineering answer is to work with it: choose delivery and scheduling so that the strong prior works in your favor at the moments that matter.

## 4. One fix pattern

All three failure modes fail for the same reason: a standing instruction that is never an object of the model's own committed output. The single pattern that addresses all three:

1. **Make the standing instruction a task the model must answer, early, at the moment the session's state is drawn.** In the experiment this moved compliance from 14-33 percent failure to zero across the arm, with the one missed first engagement recovered on the next turn because the instruction remained a re-readable message.
2. **Keep it re-readable in the conversation**, so a lost draw can be re-run. Ambient text cannot be re-read, quoted, or answered; conversation text can.
3. **For gaps the model must not fill, schedule the check.** A field the model must produce but has no source for (the date) is a hallucination slot unless some turn requires it to verify or to declare the absence. The experiment's models, given no date anywhere in context, never once declared the gap; every one of them completed it.
4. **For policy, treat the early turns as the contest they are.** If the first engaged state decides the session, then policy compliance that is never exercised in an early model reply is exactly as stochastic as the header procedure was.

## 5. Limits of the demonstration

- The compliance arm is measured: four delivery conditions, 78 sessions, exact counts, a control arm confirming the behavior belongs to the delivered guidance.
- The hallucination arm is a micro-instance: one narrow procedure (a date field) with no in-context source, observed across the experiment's sessions. The general hallucination problem is wider than this instance demonstrates.
- The jailbreak arm is a structural inference: the same conditioning mechanism, the same latching behavior, applied to safety policy. It converges with the independent multi-turn-jailbreak literature, but this experiment did not attempt jailbreaks and makes no measurement claim about them.
- One model, one sampling configuration, one task domain. The mechanism is argued to be architectural and therefore general; the magnitudes are specific to the conditions tested.

## 6. The short version

The models are guessing engines trained to make guessing feel like competence. Ambient instructions lose to the visible task, gaps get filled with plausible completions, and policies get reframed away, all for the same reason: nothing in the conversation was ever scheduled to make the model handle the standing instruction at the moment it matters. The defense against a guessing engine is not a bigger prompt. It is making it show its work where the work matters.
