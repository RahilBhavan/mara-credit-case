# MARA credit case: sizing a hypothetical $5.0M secured revolver from public filings

[![CI](https://github.com/RahilBhavan/mara-credit-case/actions/workflows/ci.yml/badge.svg)](https://github.com/RahilBhavan/mara-credit-case/actions/workflows/ci.yml)
[![license: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

**Live:** https://mara.rahilbhavan.com/

[![Decision view: the $3.0 million conditional limit, set by the $3.05 million collateral cap, which is the lowest of four caps](docs/screenshot.png)](https://mara.rahilbhavan.com/)

This is a hypothetical credit committee case. It sizes a $5.0 million, 12-month secured revolver to MARA Holdings. The borrower facts come from MARA's public filings. The facility and the collateral are fictional. A tested Python decision engine runs 11 collateral and liquidity stress scenarios. The package includes a 16-sheet Excel model, committee memos, an HTML decision view and a validator that checks that all of these agree.

## Results

The case approves a conditional **$3.0 million** limit against the **$5.0 million** request (source: `artifacts/decision-record.json`). Funding stays blocked until nine pre-funding conditions clear. These include verified ownership, a first-priority lien, enforceable control of the collateral and a tested route from the collateral to repayment cash. If any of these stays unknown, the answer is decline.

The limit is the lowest of four caps, rounded down to the nearest $100,000:

| Cap | Amount | Basis |
|---|---:|---|
| Obligor | $7.50M | Illustrative standalone capacity for the borrower |
| **Collateral** | **$3.05M (binding)** | $3,870,600 stressed proceeds ÷ 1.25x coverage, less $50,000 accrued = $3,046,480 |
| Single-name | $6.00M | Illustrative policy limit for one name |
| Concentration | $4.00M | Shared custody route and sector capacity in the fictional portfolio |

- At the $3.0 million limit, pro forma exposure is $3.05 million and coverage is 1.27x.
- A fully drawn $5.0 million request would leave a $1,179,400 recovery shortfall.
- The illustrative obligor rating is `3 / Watchful`. It is an ordinal judgment, not a probability of default.
- The collateral is one simulated lot of 4,000,000 USDC on Base (`data/case/collateral.json`). Three of the 11 scenarios apply a 30% or 50% price shock, one of them across the correlated portfolio. The others are the base case, a stale price, zero collateral, two route delays, a route failure, missing ownership evidence and borrower cash stress (`data/case/scenarios.json`, results in `artifacts/scenario-results.csv`).

The borrower facts come from MARA's Form 10-Q for the period ended 30 June 2026 (`artifacts/issuer-fact-ledger.csv`, `artifacts/liquidity-analysis.csv`):

- $421.3 million of cash and cash equivalents. About 30% sits in a majority-owned subsidiary and is designated for its operations.
- 35,577 bitcoin with a $2.1 billion fair value. 4,742 were loaned and 4,528 pledged, which leaves 26,307 unrestricted.
- About $2.4 billion of debt. This includes a $150 million line of credit due within a year, $48.1 million of December 2026 notes and $291.6 million of notes that holders can put in June 2027.
- A static liquidity screen leaves $96.8 million after designated cash and near-term debt. If holders exercise the June 2027 put, the screen falls to negative $194.8 million.

The case does not treat MARA's reported bitcoin as collateral for this facility.

The package outputs are in `outputs/`:

- [Credit memo (PDF)](outputs/credit-memo.pdf): the two-page committee recommendation.
- [Opposing memo (PDF)](outputs/opposing-memo.pdf): the strongest case against the recommendation.
- [Committee packet (PDF)](outputs/committee-packet.pdf): the decision, thresholds, conditions, controls and model risks.
- [Credit model workbook (XLSX)](outputs/credit-model.xlsx): 16 sheets of live formulas, audited cell by cell against the engine.
- [Decision view (HTML)](https://mara.rahilbhavan.com/): the 11 scenarios, a collateral what-if lab, the decision surface and the condition register.
- [Review package (ZIP)](outputs/review-package.zip): every output, a start-here guide and a SHA-256 manifest.

## How it works

- **Engine** (`src/credit_risk/engine.py`): standard-library Python with `Decimal` arithmetic. It reads the case inputs in `data/case/`, runs the collateral waterfall and the four caps for each scenario, and writes `artifacts/decision-record.json` and `artifacts/scenario-results.*`.
- **Evidence**: five public sources are frozen with SHA-256 hashes under `02-research/snapshots/`. `scripts/extract_filing_facts.py` pulls the issuer facts from the filing XBRL and reconciles them to the memo values.
- **Builders**: the scripts in `scripts/` and `work/` generate the registers, PDFs, decision view and review bundle from the engine output.
- **Tests** (`tests/`): 65 unit tests. They include monotonicity checks (more stress never raises the limit) and hard-blocker checks.
- **Validator** (`scripts/validate_package.py`): 49 checks that reconcile the engine, workbook, memos, hashes and outputs. Each check reports PASS, FAIL or SKIP.

## Run it

You need Python 3.9 or later. CI uses Python 3.11.

```bash
pip install -r requirements.txt
PYTHONPATH=src python3 -m unittest discover -s tests
python3 scripts/validate_package.py
python3 scripts/build_package.py
```

On 21 Oct 2026 with Python 3.14.7, all 65 tests passed and the validator reported 49 PASS, 0 FAIL, 0 SKIP. `scripts/build_package.py` rewrites every artifact in dependency order, then runs the tests and the validator. That run also finished without errors.

The workbook builder is not in this repository, so `build_package.py` skips that step and keeps the committed `outputs/credit-model.xlsx`. The validator and `scripts/audit_workbook.py` check its formulas and values. The decision-view audit needs Node.js. On macOS with `ffmpeg`, add `--with-demo` to rebuild the narrated video. No step needs credentials or paid data.

To evaluate one scenario from the command line:

```bash
PYTHONPATH=src python3 -m credit_risk --case-dir data/case --scenario collateral_down_30
```

## Repository layout

- `01-brief/` to `06-execution/`: the project records, in order: brief, research and frozen sources, design, deliverable specifications, validation and the original build plan.
- `artifacts/`: generated registers, analyses, audits and reports.
- `data/case/`: case inputs for the facility, collateral, policy, scenarios and issuer facts.
- `docs/`: the README screenshot and the social card.
- `outputs/`: the memos, workbook, decision view, demo video and review package.
- `scripts/`: the artifact builders, the filing-fact extractor, the audits and the validator.
- `src/credit_risk/`: the decision engine and its command-line entry point.
- `tests/`: the unit tests.
- `work/`: the PDF, decision-view and demo builders.

## Documentation

- [Decision brief](01-brief/decision-brief.md) and [design options](03-design/options.md): the decision the case answers and the three designs compared.
- [Architecture](03-design/architecture.md) and [data model](03-design/data-model.md): the components and the calculation contract.
- [Data feasibility](02-research/data-feasibility.md) and [source register](02-research/source-register.csv): what public evidence exists and how it is frozen.
- [Validation plan](05-validation/validation-plan.md), [executed checks](05-validation/executed-checks.md) and [adversarial review](05-validation/adversarial-review.md): what is checked, how, and the scope cuts.
- [Validation report](artifacts/validation-report.md): the latest result of every validator check.
- [Reproducibility guide](artifacts/reproducibility.md) and [artifact manifest](artifacts/manifest.md): how to rebuild each artifact and its status.
- [Domain context](CONTEXT.md): the credit-decision terms the project uses.

## Sources

The source register is `02-research/source-register.csv`. All five sources were retrieved on 20 Sep 2026 and frozen as HTML snapshots under `02-research/snapshots/2026-09-20/`, each with a SHA-256 hash that the validator checks.

- MARA Holdings Form 10-Q for the period ended 30 June 2026 (SEC EDGAR).
- MARA Holdings Form 10-K for the year ended 31 December 2025 (SEC EDGAR).
- Coinbase Exchange API documentation for product candles and the product book.
- Base protocol documentation on derivation and finality.

The repository license covers this project's code and writing. It does not cover the SEC filings or the third-party documentation.

## Scope and limits

- This is an independent, hypothetical case. It is not affiliated with or endorsed by MARA Holdings or Coinbase, and it does not claim that MARA is a Coinbase customer.
- It is not a lending recommendation. The facility, collateral, Base route, covenants, policy limits, rating and portfolio are fictional.
- No private borrower data, facility documents, lien search, legal opinion or wallet evidence was reviewed.
- The caps and costs are illustrative, not calibrated credit policy.
- No independent reviewer has reproduced the numbers yet. `artifacts/external-review-kit.md` describes those checks.
- The validator does not recalculate the workbook in Excel or prove legal enforceability.

## License

MIT. See [LICENSE](LICENSE).

---

Part of [Rahil Bhavan's projects](https://github.com/RahilBhavan).
