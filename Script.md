# Speaker Script — FINAL DECK (15 slides)
### Physics-Guided Optimization for Hemoglobin Estimation
**Target: 12–15 minutes.  Bold = emphasize.  [PAUSE] = stop 1–2 s.  (cue) = delivery note.**

---

## SLIDE 1 — TITLE  (~30 sec)
(Smile. Wait for the room to settle.)

"Good [morning/afternoon] everyone. Let me start with a question. **How many of you have had a finger-prick or an arm blood test?** (raise your own hand) Almost all of us.

Now imagine doing that **every single day** to track a chronic illness. What if instead, you could measure what's in your blood by just **shining a light on your fingertip**? That's what our work is about. I'm [name], from IIIT Naya Raipur, and I'll show how we used **physics — not a bigger neural network — to make that measurement trustworthy.**"

---

## SLIDE 2 — WHY NEEDLE-FREE HEMOGLOBIN?  (~1 min 15 sec)
"So why hemoglobin? **Hemoglobin is the protein in your blood that carries oxygen.** It's a key marker doctors use for **anemia, heart disease, and chronic illness.**

The problem is **how** we measure it today — an **invasive blood draw.** It's painful, it's slow, it needs a lab, and crucially, **you can't do it continuously** or for mass screening.

The alternative is **PPG** — the little light clip they put on your finger at the hospital. You shine light in, the blood absorbs some, you read what comes back. It's **cheap, painless, and already in every pulse-oximeter and smartwatch.**

(point to the box) So the core question of this whole talk is simple: **can we measure hemoglobin with light instead of a needle — reliably enough to actually trust it?**"

---

## SLIDE 3 — CHALLENGES  (~1 min 15 sec)
"If it were easy, it'd be solved already. **Three things make it hard.**

(point card 1) **One — the signal is noisy.** Light passing through skin is full of motion and sensor noise.

(point card 2) **Two — person-to-person variation.** PPG looks **enormously different** between individuals — different skin, different blood vessels, different fingers.

(point card 3) **And three — the tricky one — weak supervision.** We get only **ONE true hemoglobin label per person**, from a single blood draw, but the signal has **thousands of data points.** So the model has very little ground truth to anchor to.

[PAUSE]

Put those together, and a standard neural network gives you **unstable, untrustworthy answers.** That's the problem we set out to fix."

---

## SLIDE 4 — TRAIN IT SMARTER (THE KEY IDEA)  (~1 min 30 sec)
"Here's our key idea, and it's deliberately simple: **don't build a bigger network — train it smarter.**

(point left box) A **usual** model only does one thing: **minimize prediction error.** And that's the trap. It can hit good numbers on paper while giving **physically impossible answers.** It overfits the noise, and it doesn't generalize — so you can't trust it.

(point right box) Our approach — we call it **PCNN** — adds **physics rules directly into the loss function.** Two rules: **"more hemoglobin means more light absorption"** — that's the **Beer–Lambert law** — and **"one person has one hemoglobin value."** These keep the model's answers **physically valid.**

And the best part — (emphasize) it's **completely free**: no new parameters, no extra computation.

(point to bottom strip) This builds on **Karniadakis and colleagues, 2021**, who introduced **physics-informed machine learning** — the idea of putting known physics inside the learning process. We bring that idea, for the first time, to **PPG-based hemoglobin estimation.**"

---

## SLIDE 5 — THE DATASET  (~50 sec)
"Quickly, the data — and it's a **public dataset**, so it's reproducible.

**252 subjects.** Four wavelengths of light — **660, 730, 850, and 940 nanometers** — recorded from the **left index finger** at **200 hertz.** A good spread of ages, **21 to 90**, and a balanced gender mix.

Our **ground truth** — the labels — comes from a **real venous blood draw**, measured with a **HemoCue Hb 201+**, a validated clinical device. So our targets are solid.

And we also get **subject-level metadata** — age, gender, height, weight, blood pressure, glucose — which we feed in to help the model **adjust for person-to-person differences.**"

---

## SLIDE 6 — PIPELINE: RAW PPG SIGNAL  (~45 sec)
(point to the plot)
"Now let me actually **show you the signal.** This is the **raw, multi-wavelength PPG** — before any cleaning.

Each colored line is one wavelength. You can see the **heartbeat pulses** clearly, but notice two things: the four channels sit at **very different amplitude levels**, and the waveform **drifts and wobbles.** That baseline drift and that scale difference are exactly the noise we have to deal with before any model can learn from this. So — step one is cleaning it up."

---

## SLIDE 7 — AFTER FILTERING & PREPROCESSING  (~50 sec)
(point to top plot)
"Here's the same signal **after a Butterworth band-pass filter.** We keep the **heartbeat frequencies** and strip out the slow drift and the high-frequency sensor noise. Notice the pulses are now **centered and clean** — the wobble is gone.

(point to bottom plot) And after **normalization**, here's the final preprocessed signal. Look — (emphasize) all **four wavelengths now line up beautifully**, on the same scale. This is what the model actually sees: clean, aligned, comparable pulses. We use **zero-phase filtering** so we never distort the **shape or timing** of the pulse — and the shape is exactly what carries the hemoglobin information."

---

## SLIDE 8 — AFTER SEGMENTATION  (~50 sec)
(point to the plot)
"The last preprocessing step — **segmentation.** We chop each recording into short windows and run **peak detection.**

The **blue dots** are the **systolic peaks** — the top of each pulse — and the **orange crosses** are the **diastolic valleys.** Detecting these lets us measure the **pulse shape precisely** — its height, its width, the timing between beats.

We cut the signal into **non-overlapping windows** so there's **no information leakage** between samples, and every window inherits that person's hemoglobin label. **Clean, segmented pulses** — now we're ready for the model."

---

## SLIDE 9 — DEEP TEMPORAL BACKBONE  (~1 min)
"So what's the model? A PPG pulse has both a **shape** and a **timing**, and you need **both.**

(point) The **1D-CNN** reads the **local shape** — the upstroke, the downstroke, the little notch in each beat.

(point) The **BiGRU or BiLSTM** reads the **timing** — how the beats evolve over time, looking both forwards and backwards.

(point) And **attention** learns **what to trust** — it turns up the clean pulses and turns down the noisy ones.

(point to bullets) Why all three? Because **CNN alone misses long-range timing**, and **recurrent alone misses the fine shape.** And in our experiments, (emphasize) **attention was the consistent winner** — focusing on the right pulses beats just stacking more layers."

---

## SLIDE 10 — THREE OPTIMIZATION STRATEGIES  (~1 min 30 sec)
"Now the **core** of the paper. We took that **same model** and trained it **three different ways** — so the comparison is fair.

(point row 1) **One — standard training.** Minimize prediction error only. Our baseline. Error: **1.04.**

(point row 2) **Two — Grey Wolf Optimization.** A clever, nature-inspired search — (slight smile) yes, it's actually modeled on **how wolf packs hunt**, with alpha and beta wolves leading. We used it to tune the model from **outside.** And the result is telling: (pause) it **barely helped** — error went **up** to **1.49.** Because it's blind and external, and our model was already good on its own.

(point row 3) That failure is exactly what motivated **strategy three — our PCNN** — putting the physics rules **inside** the loss. Error dropped to **0.98** — our best.

(emphasize) The lesson: **optimization shouldn't be bolted on the outside — it should live inside what the model is trying to achieve.**"

---

## SLIDE 11 — THE TWO PHYSICS CONSTRAINTS  (~2 min)
(Your most important slide. Slow down.)

"So what are these two rules? Here's the loss: the usual error, **plus** two physics penalties. Let me explain both in plain English.

(point ①) **Rule one — optical consistency.** From the **Beer–Lambert law**: more hemoglobin means **more light absorption** — never less. So we tell the model: your predictions must **move WITH the absorption signal.**

And here's the elegant part — it's a **one-sided** rule. If the model is **already** pointing the right way, which biology expects, it pays **zero penalty.** It only kicks in when the model predicts something **physically backwards.** Think of it as a **guardrail** — you only feel it if you **swerve off the road.**

[PAUSE]

(point ②) **Rule two — smoothness.** Pure common sense: **a person has ONE true hemoglobin value.** It doesn't jump around. So **all of that person's segments should predict the same thing.** This rule **penalizes disagreement** between a person's segments — which **kills the model's sensitivity to noise and motion artifacts.**

(step back) Rule one buys **accuracy**; rule two buys **stability.**"

---

## SLIDE 12 — RESULTS: ARCHITECTURE COMPARISON  (~50 sec)
(let the bars do the work)
"Let's look at the evidence. First, **which architecture wins.** Lower bar is better.

The pattern is crystal clear. **Recurrent models alone** — BiLSTM, BiGRU — sit at the top, **weak.** Add a **CNN** in front, and the error drops nicely. And (point to the green bar) **add attention**, and you get the **best architecture at 1.09.**

So before we even touch the physics, the message is: you need **shape and timing together, with attention.** **1.09 is the number to beat.**"

---

## SLIDE 13 — RESULTS: OPTIMIZATION METHODS  (~1 min 30 sec)
(Slow down. This is your proof.)

"And **this** is the result the whole talk builds to — what the **physics rules** actually do.

(point) Start at the top: the **baseline, no constraints** — error **1.04.**

(point) Add **just the optical rule** — (pause) — error drops to **0.975.** So the physics didn't just make the model prettier — (emphasize) it made it **genuinely more accurate.**

(point) Add the **smoothness rule** too, and the error ticks **slightly up**, to 1.02. (pre-empt the question) And I want to be **honest** about that — it's **not a bug.** It's a **deliberate trade-off.** We give up a hair of accuracy to buy **stability** — so every reading for the same person **agrees.** In a clinic, that can matter more.

(point to side box) So we offer **both**: **optical-only is the most accurate**, the **full model is the most stable.** You choose based on what you need."

---

## SLIDE 14 — LIMITATIONS, BIGGER PICTURE, CONCLUSION  (~1 min 30 sec)
(Limitations build credibility — say this confidently, not apologetically.)

"No honest talk skips the **limitations.**

(point left) The big one is **skin tone.** **Melanin** — skin pigment — absorbs light in **almost the same band** we use for hemoglobin. So our model could be **biased for darker-skinned subjects**, and our dataset didn't have skin-type labels to correct for it. I name that openly — it's not just technical, it's a **fairness issue** the field needs to tackle. We also don't yet correct for **LED current drift.**

(point right) But here's the **bigger picture** — and why this matters beyond hemoglobin. **Nothing about our method is specific to blood.** Any problem that's **noisy, weakly-supervised, and governed by known physics** can use the **same recipe.** That ties directly to this session's theme — **data-driven optimization.**

(point to conclusion) So if you take **one thing** away: (emphasize, slow) **constrain the optimization, not the network.** Two free physics rules made our predictions **more accurate, more stable, and more trustworthy.**"

---

## SLIDE 15 — THANK YOU  (~15 sec)
"That's our work — **adding physics to the loss, not the architecture**, for reliable, needle-free monitoring. **Thank you** — I'd be glad to take your questions."

(Smile. Step back. Wait confidently. Don't fill the silence.)

---

## TIMING TABLE
| Slide | Topic | Time |
|---|---|---|
| 1 | Title + hook | 0:30 |
| 2 | Why needle-free | 1:15 |
| 3 | Challenges | 1:15 |
| 4 | Train it smarter | 1:30 |
| 5 | Dataset | 0:50 |
| 6 | Raw signal | 0:45 |
| 7 | After filtering | 0:50 |
| 8 | After segmentation | 0:50 |
| 9 | Backbone | 1:00 |
| 10 | Three strategies | 1:30 |
| 11 | Two constraints | 2:00 |
| 12 | Results: architecture | 0:50 |
| 13 | Results: optimization | 1:30 |
| 14 | Limitations + conclusion | 1:30 |
| 15 | Thank you | 0:15 |
| | **TOTAL** | **~14:30** |

## IF RUNNING LONG — cut these first
1. Combine slides 6–7–8 into one fast sweep (~1:30 total instead of 2:25): "Here's the raw signal → after filtering it's clean and aligned → after segmentation we detect every peak."
2. Trim slide 5 (dataset) to 30 sec — just 252 subjects, 4 wavelengths, HemoCue ground truth.
**Never rush slides 11 and 13** — your contribution and your proof.

## THREE ENERGY TIPS
1. **Open with the question on Slide 1** — "who's had a blood test?" — before the title sinks in.
2. **Use the wolf-pack line on Slide 10** — a small laugh resets attention at the midpoint.
3. **Slow down and PAUSE after "0.975" on Slide 13.** Speakers rush their best result — don't.

## ⚠️ ONE FIX TO MAKE ON THE DECK
Slide 5 says "56.7% female, 46.3% male" — that adds to 103%. It should read **43.3% male** (so it sums to 100%). Quick correction before the talk.
