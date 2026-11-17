# Soak Before Sync

**How long should a shared agent skill update soak before it reaches every recipient?** This is a decision prototype that uses a composition Monte Carlo (mixtures, convolutions, Box-Muller) to pick the window.

- **Live demo:** https://shriyashekatkar25-byte.github.io/soak-before-sync/
- **Gallery:** [APM Lab](https://shriyashekatkar25-byte.github.io/apm-lab/), a series of product-decision prototypes built with Monte Carlo methods
- **Author:** Shriya Shekatkar · UCLA · Stats 102C Monte Carlo Methods

## The launch that inspired it

On **November 16, 2026**, Databricks released a Beta that lets providers include **Unity Catalog agent skills in OpenSharing shares and Databricks Marketplace listings**. Databricks recipients get read-only access, and updates reach them asynchronously ([release notes, November 2026](https://docs.databricks.com/aws/en/release-notes/product/2026/november)). Within a workspace, sharing a skill "grants access to the live skill", so recipients' agents read the latest published version ([Create and share skills docs](https://docs.databricks.com/aws/en/agents/uc-skills/create-share-uc-skills)). Background on OpenSharing: [Databricks blog](https://www.databricks.com/blog/announcing-new-opensharing-and-marketplace-capabilities-ai-era).

## The PM decision

Recipients can't edit a shared skill, so a provider's bad version quietly changes how another company's agents behave. I play the PM who owns skill distribution and has to choose a default:

- **Live sync:** every new version reaches every recipient immediately.
- **Soak window (proposed):** a new version goes live for an opt-in canary ring and for the provider's own evals first. It reaches everyone else after *S* hours unless a break is detected. Security fixes can skip the soak.

| | Target |
|---|---|
| Success metric | Broken agent-hours per recipient per quarter cut by at least 50% vs live sync |
| Guardrail | Broken agent-hours / recipient / quarter ≤ 1.0 h |
| Guardrail | Recipients hit by ≥1 bad version per quarter ≤ 8% |
| Guardrail | Improvement delay (soak length) ≤ 72 h |
| Guardrail | Security-fix delay, P90 ≤ 24 h |

Stakeholders: skill providers, recipient platform teams, Marketplace and sharing engineering, security, and field/support.

With the default inputs, the sweep finds that **about 36 hours** is the shortest soak that clears every guardrail. It cuts broken agent-hours by roughly half compared with live sync. The memo on the page rewrites itself whenever you move a slider.

## What this shows about product judgment

- **Customer problem framing:** the cost is framed from the recipient's side ("my agent changed and I didn't touch anything"), not only from the provider's.
- **Prototyping:** a working, seeded simulator with a live sweep chart. Any stakeholder can test their own assumptions.
- **SQL / Python:** illustrative queries fit the model inputs from version and rollback logs, and a NumPy sweep reruns the model at scale.
- **Data-driven decisions:** the recommendation is the shortest window that clears every guardrail, not a gut-feel number. Common random numbers keep the comparison between windows fair.
- **Clear writing:** a director memo covering decision, evidence, risks, rollout and next step, plus a one-sentence version for the VP.

## Methods (Stats 102C, Chapter 5: Composition)

| Method | Where it's used |
|---|---|
| Mixture sampling | Each provider is drawn as "careful" or "move-fast" first, then gets that component's ship and break rates. Whether the provider's own eval catches a break is a second mixture. |
| Convolution | 13 weekly Bernoulli ship decisions sum to a Binomial update count. Exponential report waits plus the fix time sum to a Gamma-shaped time to rollback. |
| Box-Muller | Normal quality-lift draws for good updates, used to measure the value a soak holds back. |
| Inverse CDF (supporting) | Exponential waits drawn as −m·ln U. |
| Seeded PRNG | mulberry32, so every run is reproducible from its seed. |

## Design baseline

I picked four pins at random from a Pinterest home feed and redrew each one's look in CSS, SVG and canvas. No pin images are used anywhere.

- **Hero:** ["gifts from God" by Angel](https://www.pinterest.com/pin/1005991635536644887/). I took the full-bleed hot-magenta field and the heavy black grotesque type, fully justified with wide gaps and a soft blurred glow.
- **Cards and buttons:** [red marker quote card (Case Kenny), pinned by April](https://www.pinterest.com/pin/1085930528938961340/). I took the plain white card, red felt-tip handwriting for headings, and the small red signature line.
- **KPI badges:** ["Bug me anytime / Little ladybug" by Elly 4](https://www.pinterest.com/pin/594404850862077075/). I took the brown serif and dusty-pink serif type pairing and the ladybug shell with heart-shaped spots, redrawn as SVG badges.
- **Simulation room:** ["🪷" lotus pond by уσℓι٭](https://www.pinterest.com/pin/811492426651242659/). I took the dark rain-dimpled green water, the pink-and-white lotus blooms, and rain ripples as the visual for each sample.

---

*Unofficial concept prototype by Shriya Shekatkar. All data is simulated. Not affiliated with any company.*
