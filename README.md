# Ryan Prasad | AI Engineer

I build tool-using AI applications, from the agent's behavior and evaluations to the infrastructure that runs them. My main project is **[TollChat](https://tollchat.ai/)**, a deployed application that answers Northern Virginia toll-price questions and estimates annual costs for the tolled portion of a commute. I built its Python agent, deterministic pricing tools, PostgreSQL/PostGIS route model, evaluation harness, and AWS delivery pipeline.

I'm looking for AI engineering work where I can own agent behavior, tools, and evaluation. The Kubernetes and orchestration projects below show the supporting skills I bring to that work.

**[Try TollChat](https://tollchat.ai/) · [Read the code](https://github.com/rhprasad0/nova-toll-budget-agent) · [LinkedIn](https://www.linkedin.com/in/ryan-prasad-ai/) · [GitHub](https://github.com/rhprasad0)**

## Engineering evidence index

This section maps capabilities to public implementation, tests, and recorded decisions. TollChat is the primary project; the other repositories supply supporting evidence. Status reviewed **October 7, 2026**. Links to `main` follow current source; immutable review snapshots are listed below.

### TollChat: end-to-end tool-using AI application

- **Ownership:** Independent personal project. I built the data ingestion, directed route model, pricing tools, agent, web interface, evaluations, infrastructure, and release controls.
- **User problem:** Resolve supported toll-road entrances and exits, retrieve current prices, and estimate annual toll-commute costs from recorded prices and explicit assumptions. Untolled segments are excluded; vehicle-cost and tax assumptions are fixed and disclosed.
- **Stack:** Python, Strands Agents SDK, OpenAI model integration, Amazon Bedrock AgentCore Runtime, Bedrock Guardrails, PostgreSQL/PostGIS, Terraform, GitHub Actions, Lambda, S3, CloudFront, WAF, and CloudWatch.
- **Deployment:** Separate AWS development and production environments. This is a deployed personal reference implementation with a public demo.

| Capability | Implementation evidence | Tests / inspection |
| --- | --- | --- |
| Bounded tool use and structured contracts | [Strands agent](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/agent/toll_agent.py) exposes two pricing tools; the model resolves intent while code owns route validation and money arithmetic | [Tool contract tests](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/tests/test_tool_contract.py) |
| Deterministic current and annual pricing | [Current-price domain](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/agent_tools/current_price_domain.py) and [annual-ballpark tool](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/agent_tools/get_annual_toll_ballpark.py) validate inputs, apply pricing rules, and return structured results | [Routing and pricing contract](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/db/oracle/CONTRACT.md); [build and database verification](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/README.md#verify-the-build) |
| Source-backed data and least-privilege access | [Oracle data builder](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/oracle/build_oracle_data.py) maps source data into directed routes; [database roles](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/db/roles.sql) bound runtime access | [Source mappings](https://github.com/rhprasad0/nova-toll-budget-agent/tree/main/v2/oracle/sources) and [IAM contract tests](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/tests/test_infrastructure_iam.py) |
| Input/output safety and privacy-aware observability | [Runtime guardrail checks](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/agent/agentcore_entrypoint.py) and [telemetry exporter](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/agent/telemetry.py) redact detected PII before trace export and omit affected content on redaction failure | [Runtime tests](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/tests/test_agentcore_entrypoint.py) and [redaction tests](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/tests/test_telemetry_redaction.py) |
| Agent evaluation and failure analysis | [Simulated conversations](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/eval/simulated.py), deterministic tool-call checks, model-based judges, and candidate-bound evaluation contracts | [Evaluation guide](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/eval/README.md), [simulation tests](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/tests/test_simulated_evaluation.py), and [experiment journal](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/eval/EXPERIMENT_JOURNAL.md) |
| Reproducible delivery and environment separation | [Terraform account contract](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/infra/account-contract.json), [development workflow](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/.github/workflows/v2-development-delivery.yml), and [production workflow](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/.github/workflows/v2-production-release.yml) | [Foundation tests](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/tests/test_infrastructure_foundation.py) and [production release checks](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/RUNBOOK.md#production-release-checks) |
| Blue-green releases and conversation isolation | [Release controller](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/scripts/release_blue_green.py) validates candidates before cutover; [chat proxy](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/lambdas/chat_proxy/handler.mjs) rejects sessions from a different release | [Blue-green tests](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/tests/test_blue_green.py), [session tests](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/lambdas/chat_proxy/handler.test.mjs), and [deployment runbook](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/runbooks/blue-green-deployments.md) |

### Engineering decisions worth inspecting

1. **Keep pricing outside the model.** The agent has two tools, not arbitrary SQL or a model-authored pricing plan. The directed route contract separates reachability from geographic proximity; coordinates alone cannot create a road connection.
2. **Make failures visible.** Evaluation records distinguish successful, failed, and inconclusive measurements. The [experiment journal](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/eval/EXPERIMENT_JOURNAL.md) preserves hypotheses, comparisons, limitations, and decisions, including attempts that missed their targets.
3. **Treat a release as a conversation boundary.** A session from an older release gets an explicit restart instead of silently continuing under changed agent behavior. Production cutover requires separate human approval and retains the prior release for recovery.
4. **Separate measured behavior from safety controls.** Guardrails and redaction reduce risk but can miss information. Tests and workflow definitions establish particular contracts; live deployment and evaluation results need their own run-specific evidence.

### Evaluation status and limits

As of the review date, the former 100-case development corpus is archived. Its exposed development results are historical measurements, not production qualification or whole-agent accuracy.

The current harness prepares a new **50-case training / 10-case shadow / 25-case external holdout** design. No fresh set is active yet. Shadow CI is wired for reviewed inputs but remains in readiness validation until the required cases, approval, and infrastructure are present. A readiness check is not a measured shadow result. See the [current evaluation guide](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/eval/README.md) and [October preparation record](https://github.com/rhprasad0/nova-toll-budget-agent/blob/main/v2/eval/journal/2026-10-05.md).

The six-scenario scheduled simulation suite is a separate operational check. Model-based judge scores require calibration and interpretation; repeated holdout feedback can influence tuning. Delivery gates verify release provenance and operational checks, not independently measured agent accuracy.

## Supporting projects

### Kubernetes and GitOps: [aws-devops-lab](https://github.com/rhprasad0/aws-devops-lab)

An ephemeral AWS EKS lab provisioned with Terraform. Argo CD reconciles application manifests from Git; Argo Rollouts stages canary releases; Karpenter provisions Graviton capacity with Spot-interruption handling. This supplies Kubernetes/platform evidence alongside TollChat's agent-runtime deployment.

- **Implementation:** [EKS/VPC and Argo CD bootstrap](https://github.com/rhprasad0/aws-devops-lab/blob/main/infra/main.tf), [Argo CD application](https://github.com/rhprasad0/aws-devops-lab/blob/main/k8s/argocd/applications/guestbook.yaml), [canary rollout and health probes](https://github.com/rhprasad0/aws-devops-lab/blob/main/k8s/guestbook/rollout.yaml), [Karpenter](https://github.com/rhprasad0/aws-devops-lab/blob/main/infra/karpenter.tf), [CloudWatch/Fluent Bit/X-Ray](https://github.com/rhprasad0/aws-devops-lab/blob/main/infra/cloudwatch-observability.tf).
- **Scope:** Completed learning lab; the repository does not establish that a cluster is currently running. Canary steps are replica-based; automated metrics-driven rollback and precise ALB traffic splitting are not implemented.

### Agent orchestration: [closed-loop-ai-podcast](https://github.com/rhprasad0/closed-loop-ai-podcast)

A seven-stage Python workflow on AWS Lambda and Step Functions using Amazon Bedrock. Typed state carries discovery, research, scripts, and production artifacts between stages. A Producer evaluator returns structured feedback to the Script agent, with **three total script attempts** before terminal failure; successful runs proceed to artwork, speech generation, and media assembly.

- **Implementation:** [Step Functions state machine](https://github.com/rhprasad0/closed-loop-ai-podcast/blob/main/terraform/step-functions.tf), [typed handoff state](https://github.com/rhprasad0/closed-loop-ai-podcast/blob/main/lambdas/shared/python/shared/types.py), [Producer evaluator](https://github.com/rhprasad0/closed-loop-ai-podcast/blob/main/lambdas/producer/handler.py), [script contract and feedback tests](https://github.com/rhprasad0/closed-loop-ai-podcast/blob/main/tests/unit/test_script.py).
- **Scope:** Personal orchestration project. Historical episode/run counts are not current adoption metrics, and audience-performance feedback is manually ingested.

## Machine-readable profile

```yaml
schema_version: 1
name: Ryan Prasad
github: https://github.com/rhprasad0
linkedin: https://www.linkedin.com/in/ryan-prasad-ai/
focus: AI engineering for tool-using applications
preferred_work:
  - Agent behavior and tool contracts
  - Evaluation harnesses and failure analysis
  - End-to-end application delivery
primary_project:
  name: TollChat
  repository: https://github.com/rhprasad0/nova-toll-budget-agent
  demo: https://tollchat.ai/
  ownership: independent personal project
  status: deployed reference implementation; active development
  language: Python
  agent_framework: Strands Agents SDK
  runtime: Amazon Bedrock AgentCore
  data_layer: PostgreSQL and PostGIS
  infrastructure: Terraform and GitHub Actions
  evaluation_status: fresh training and shadow inputs pending
supporting_projects:
  - repository: https://github.com/rhprasad0/aws-devops-lab
    evidence: Kubernetes; EKS; Terraform; Argo CD; Argo Rollouts; Karpenter
    scope: ephemeral learning lab
  - repository: https://github.com/rhprasad0/closed-loop-ai-podcast
    evidence: Python; Step Functions; Lambda; Bedrock; typed handoffs; bounded review loop
    scope: personal orchestration project
claims_reviewed_on: "2026-10-07"
```

## Verification snapshots

The review used these public source revisions. File links above follow `main`, so implementations and status can change after this snapshot.

- **TollChat:** [7670ff82](https://github.com/rhprasad0/nova-toll-budget-agent/tree/7670ff82a4a36e57080d87047c86b76bc039ab63)
- **Kubernetes lab:** [f993d09](https://github.com/rhprasad0/aws-devops-lab/tree/f993d09b9a39e9e627a7ad1c4cc0160b314382a1)
- **Podcast pipeline:** [b19e670](https://github.com/rhprasad0/closed-loop-ai-podcast/tree/b19e670b629502e4f338dac7e0040ef90dc8a511)

For a short technical review: start with TollChat's agent and two tools, follow a contract into its tests, then inspect one experiment and the release/session boundary. Public repositories show implementation and engineering judgment. Customer scale and production reliability require evidence beyond these project artifacts.
