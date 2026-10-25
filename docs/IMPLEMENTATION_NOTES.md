# Artha AI implementation notes

This checkout was reviewed against the attached India monetary-policy simulator blueprint and the live beta deployment. It remains a research prototype; it is not an official RBI system or an investment/policy recommendation engine.

## Fixes applied

- Fixed the public Next.js build failure by declaring the hook-using `AgentTimeline` as a client component.
- Fixed the CPI trend card so it reports the number of plotted historical observations and the latest observed date. Previously it showed `0 macro observations` despite rendering the series and described the forecast endpoint as the latest observation.
- Fixed the research critic's false positives: a boolean planner flag is not a numeric economic claim, and CPI is legitimate monetary-policy evidence. Numeric CPI claims without a source remain flagged.
- Fixed report generation to handle Pydantic evidence records, preserve deterministic source order, include uploaded-document citations and page metadata in the report, and collect policy uncertainties safely.

## Model and data handling

- Re-ran `app/models/india_macro_models_enriched/scripts/train_enriched_models.py` against its bundled observed-source panel. It regenerated the ARIMA, Ridge and XGBoost prototypes, direct 1/3/6-month horizon models, feature variants and rolling-origin evaluation. The package validator passed: 152 observed CPI YoY labels (2014-01 through 2026-08), 51 metric rows, 1,224 rolling-origin predictions, 42 Ridge/XGBoost artifacts, no synthetic training labels, and baseline metrics reproduced within floating-point tolerance.
- Re-ran `scripts/retrain_p2_compat_models.py --force` against its separately identified MIT-licensed comparison panel. ARIMAX converged at every one of the 52 walk-forward origins; XGBoost was saved in native JSON. This is an external benchmark with a distinct target definition, not the application's MoSPI 2024-base target.
- The imported checksum-verified `Final_model_sets/India_CPI_Model_Package` remains a frozen artifact package. This repository does not contain its model-training source code, so its serialized estimators and saved forecast exports were not retrained or silently replaced. The API serves those saved, checksum-verified outputs. Use the included retraining scripts for the documented enriched and P2 suites, and review the model-specific limitations before interpreting any result.
- Several important channels remain incomplete or on different vintages (for example food inflation is a 2012-base series, the latest repo-rate observation is older than the latest CPI observation, and annual GDP is context—not a monthly training target). The application appropriately withholds an evidence-weighted policy recommendation and CPI decomposition when matched data are absent. Scenario outputs are explicitly illustrative sensitivities, not estimated causal effects.

## Validation performed

- `python -m pytest -q`: **74 passed**.
- `python app/models/india_macro_models_enriched/scripts/validate_package.py`: **passed**.
- Python smoke checks covered local sourced series, the seasonal-naive baseline, the packaged Ridge forecast, scenario sensitivity and all 51 packaged model-metric entries.
- Frontend `npm run typecheck` and `npm run build`: **passed** after the fixes. `npm run lint` also completed with zero errors; existing unused-variable and React-hook-dependency warnings remain.
- The live API was probed for `/health`, `/series`, `/sources`, baseline/model forecasts and model registries. The beta frontend needs a new Vercel deployment before these source changes appear on the hosted domain.

## Vercel deployment

This repository defines separate API, public frontend, and admin frontend Vercel projects. Follow [`VERCEL_DEPLOYMENT.md`](VERCEL_DEPLOYMENT.md), set `NEXT_PUBLIC_API_BASE_URL` on each frontend to the deployed API origin, configure the API's allowed origins to include the exact public frontend/admin domains, and redeploy all affected projects. Never commit `.env` secrets. Vercel functions use ephemeral storage; durable uploads, evidence, and research runs require external persistent storage.
