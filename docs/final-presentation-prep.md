# Final HLD Presentation Prep — NeoBank Digital Leap

> For the Final HLD submission talk. Companion to `docs/final-presentation/index.html`
> (15 slides, ~15 min). Same engine as MS1; facts anchored to the Final HLD v1.0 sources.
>
> **Status:** slide content drafted from `docs/00..13` + appendices; images rendered
> fresh from governed `.mmd` sources into `docs/final-presentation/img/` — do **not** reuse
> the MS1 `c4.png` (stale; MS1's "DR deferred to MS2" note no longer applies).

---

## Quick reference

| | |
|---|---|
| **Total time** | 15 min talk + Q&A |
| **Slides** | 15 |
| **Deck** | `docs/final-presentation/index.html` (self-contained, like MS1) |
| **Image assets** | `img/high-level.png` (01), `img/network-topology.png` (06), `img/delivery-roadmap.png` (18), `img/performance-capacity.png` (17), `img/c4-container.png` (07 — backup for Q&A), `img/security-trust.png` (12 — backup for Q&A) |
| **Opening line** | "MS1 showed the picture. Today: the finished design — hardened boundaries, managed contracts, sizing, cost, and the plan." |
| **Closing line** | "The mainframe was never the obstacle. The honest boundary beats the clever feature. Constraints aren't the problem — they're the design." |
| **Headline message** | MS1's three insights survived review; the final hardened them into 8 zones, RC classes, 14 managed contracts, and a P75 plan. |

## Slide map (15)

1. **Title** — project, Revision 1.0, August 2026.
2. **MS1 → Final** — what survived review (3 insights) vs what MS2/final added.
3. **High-Level Solution** — `01-high-level-solution.mmd`. Normative arrows.
4. **8 trust zones** — Z1..Z8 with crossing rules. MS1's rules turned into zone algebra.
5. **Four journeys** — auth/ingress, account read, transfer, Open Banking. Failure discipline per journey.
6. **Managed contracts** — 7 APIs + 7 events, authority per field; Appendix B.
7. **Network topology** — `06-network-topology.mmd`. Teacher-mandated; zones made physical.
8. **Recovery classes** — RC0..RC3 RTO/RPO table; human-governed RC0 promotion (L-013).
9. **Release discipline** — immutable promotion, admission fails closed, compatibility windows, WORM receipts.
10. **Sizing & cost** — nodes 34→66, infra TCO 2.42 → 5.20 USD M base, bands + 20% contingency (L-009).
11. **Delivery plan** — P75 = 6,839 pd ≈ 311 pm; 46 avg / 55 peak FTE; cutover Wk 48; delay scenarios 52/60/68–76 wk; zero AI uplift (L-011).
12. **Risks & limitations** — R-009, L-005, L-013, R-016 sampled from 21 risks + 15 limitations.
13. **Traceability & evidence** — 89 requirements, 4 appendices, 20 assumption IDs, 12 open issues.
14. **Closing message** — the three sentences.
15. **Q&A** — 4 rules footer (added Rule 4: unknown outcomes → PROCESSING).

## Speaker tips

- **Slide 2 is the credibility slide.** The reviewer hears "MS1 gaps acknowledged and closed" before any new content. Do not skip it under time pressure.
- **Time check:** at slide 7 (topology) ≈ 7:00; at slide 11 (delivery) ≈ 12:00. If past 8:00 by slide 7, compress slides 8–9.
- **Numbers are bands.** Say "low–base–high" at least once on slide 10 and "P75, not P50" on slide 11 — that's the estimating discipline the course marks.
- **Rule 4 is new.** MS1 ended with 3 rules; the final adds "unknown outcomes → PROCESSING" as the fourth rule. Say it on the Q&A slide.

## Q&A backup (likely follow-ups beyond MS1's set)

**Q: Why P75 and not P50?**
> P50 folds when the external dependencies (regulatory approval, procurement, mainframe windows) slip — and they do. The P75 plan plus a separately governed 10% reserve is the funded envelope; external delays are modelled as elapsed-time scenarios (52/60/68–76 weeks), not hidden person-day padding. P50 is reported as sensitivity only.

**Q: The 24h staleness rule — still honest?**
> Narrowed since MS1 (L-004): it applies only to incoming external or out-of-app payments. Completed digital transfers get read-your-writes through the CBS-confirmed recent-write overlay until the ODS converges (NFR-DI-060). Freshness metadata is always on screen (FN-160).

**Q: Why is AI inference almost free on the cost sheet?**
> It's honest: 15.77B input tokens in Year 1 at ≥90% cost-efficient routing is ≈6K USD/yr base. The token volume is what makes advice affordable; the boundary (Advisor Context) is what makes it approvable. Approval, not inference cost, is the AI risk (R-007).

**Q: What blocks production readiness?**
> The unaccepted High risks: R-001..R-006, R-009..R-013, R-016, R-019 — each named with owner and expiry (HLD §6). And the ten Phase-0 validations in the open-issues register (§7). Validation failure must surface as a visible decision, never a silent contract rewrite.

**Q: Network topology — why defer CIDRs but fix zones?**
> Zone relationships and prohibited routes are architectural decisions (they preserve the security model). CIDRs, carriers, and bandwidth are procurement and Phase-0 validation inputs (L-006). Fixing the former early forces the latter to fit; fixing the latter early would be false precision.

**Q: Two resolved open issues — which?**
> OI-007 (recovery classes fixed: RC0 15m/0 loss → RC3 24h) and OI-010 (deployment posture: projection build, thin slices, pilot, canary→limited→full, Wk-48 cutover, Wk 49–52 hypercare, state-aware recovery). Ten validations remain for Phase 0.

## Export checklist (night before)

- [ ] Re-render any touched `.mmd` before presenting:
  ```bash
  npx -y -p @mermaid-js/mermaid-cli mmdc -i docs/diagrams/01-high-level-solution.mmd -o docs/final-presentation/img/high-level.png -w 1600 -b white
  npx -y -p @mermaid-js/mermaid-cli mmdc -i docs/diagrams/06-network-topology.mmd -o docs/final-presentation/img/network-topology.png -w 1600 -b white
  npx -y -p @mermaid-js/mermaid-cli mmdc -i docs/diagrams/18-delivery-roadmap.mmd -o docs/final-presentation/img/delivery-roadmap.png -w 1600 -b white
  npx -y -p @mermaid-js/mermaid-cli mmdc -i docs/diagrams/17-performance-capacity.mmd -o docs/final-presentation/img/performance-capacity.png -w 1600 -b white
  ```
  Slide 7 (network topology) uses a **split layout** (text left, image right). The image is a top-down
  `flowchart TD` (tall) — if you re-render at a different width and the image becomes near-square, drop the
  `.split` and use the `.image-slide` layout instead (see slide 3 for the wide pattern).
- [ ] Open `docs/final-presentation/index.html` in the presentation browser; press `N` to confirm notes load.
- [ ] Deep links work: `index.html#N` opens slide N (1-based) — useful for rehearsal.
- [ ] Print via `@media print` fallback (each slide one page) as the PDF backup.
- [ ] Rehearse slide 2 (MS1 → Final) verbatim — it sets the review posture.
