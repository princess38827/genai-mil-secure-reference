@@ -0,0 +1 @@
# genai-mil-secure-reference
GenAI.mil Security Architecture — Secure Scaling Reference (IL5 → IL6/IL7)
Reference Implementation & Architecture Analysis
Unclassified, based on public FedRAMP High, DoD SRG, NIST AI RMF, and Zero Trust standards.

This repo models how the Department of War's GenAI.mil platform safely scales commercial frontier AI (Gemini for Government, OpenAI, Anthropic, Meta Llama Gov) across a workforce of millions without risking national security data spillage.

The hardest part of scaling GenAI.mil isn't model capability — it's governance.

How do you let millions of personnel build hundreds of thousands of agents without an agent getting admin privileges or leaking CUI?

I put together an unclassified reference architecture that models the approach:
• IL5 CUI Hub as primary entry point, with secure pipelines to IL6 (SIPRNet) / IL7 (JWICS)
• ZTA enforced by CAC + continuous verification
• Input/output sanitization per Prohibited Data Notice
• Sovereign hosting — prompts never train commercial models
• MCP + RBAC + continuous red-teaming for vibe-coding controls

Full code + diagram + NIST RMF crosswalk in the repo. Built to learn in public and contribute to the conversation around secure enterprise AI.

Live Repo: https://github.com/princess38827/genai-mil-secure-reference

1. Overview
Rather than isolating tools in a legacy, physically air-gapped environment, GenAI.mil functions as a heavily protected, unified enterprise hub. It enforces rigorous security data standards, continuous testing, and restricted user boundaries.

Three core design choices make scale possible:

Unified Hub, Not Air-Gap: One IL5 entry point for all unclassified/CUI workflows
Sovereign Control: Commercial models run in dedicated government hosting layers - prompts never train commercial models
Agent Governance First: MCP + RBAC prevents autonomous agents from lateral movement or admin escalation
2. Data Classification and System Certifications
IL5 — Unclassified / CUI Hub (Primary User-Facing)
Authorization: Impact Level 5 (IL5) under FedRAMP High guidance
Allows: Controlled Unclassified Information (CUI), sensitive unclassified military workflows
Default Handling: All generated outputs treated as CUI by default to mitigate information tracking and leakage
IL6 / IL7 — Classified Expansion
IL6 (SIPRNet): Separate secure pipeline for Secret workloads, encrypted ingress/egress, Cross-Domain Solution (CDS)
IL7 (JWICS): Top Secret / SCI network, air-gapped secure pipeline, enhanced audit & monitoring
Integration: Frontier models from Google, OpenAI, Microsoft, xAI integrated via secure pipelines with TLS 1.3
3. Infrastructure Guardrails and Data Sovereignty
Zero Trust Architecture (ZTA)
python
def verify_cac(x_cac_cert: str = Header(...)):
    # Continuous verification, PKI/CAC validation, MFA, device compliance
    if not x_cac_cert or "DOD" not in x_cac_cert:
        raise HTTPException(401, "Zero Trust: Valid CAC required")
Preventative Input/Output Sanitization
In accordance with Prohibited Data Notice, automated filters intercept IL5 inputs:

python
PROHIBITED_PATTERNS = {
    "CLASSIFIED": r"\b(TS|SECRET|TOP SECRET|SIPR|JWICS)\b",
    "PII": r"\b\d{3}-\d{2}-\d{4}\b",
    "PHI": r"\b(medical record|diagnosis)\b"
}
Sovereign Cloud Data Control
Models run in secure, dedicated sovereign government hosting
Enterprise parameters guarantee prompts and operational metadata are never used to train foundation models
US-personnel support, data residency guarantees, audit logging
4. Agent Governance & "Vibe-Coding" Controls
With military personnel building hundreds of thousands of autonomous agents:

Model Context Protocol (MCP) + RBAC
python
class MCPTool(BaseModel):
    name: str
    allowed_dbs: list[str]
    requires_human_approval: bool

MCP_REGISTRY = {
    "query_mission_db": MCPTool(allowed_dbs=["mission_db"], requires_human_approval=False),
    "query_personnel_db": MCPTool(allowed_dbs=["personnel_db"], requires_human_approval=True),
}

def check_agent_rbac(agent_role: AgentRole, requested_db: str):
    if agent_role == AgentRole.ADMIN:
        raise HTTPException(403, "Agents cannot have admin privileges")
    if requested_db not in authorized_dbs:
        raise HTTPException(403, "RBAC: No lateral movement")
Non-human agents lack administrative network privileges
No lateral movement without user oversight
Session isolation & audit trails for every agent action
Continuous Crowdsourced Assurance (CDAO)
Continuous red-teaming, autonomous red-team testing, threat simulation
Automated risk scoring and playbook-driven responses
Prevents malicious prompt injection and toxic drift
python
def risk_score_prompt(prompt: str) -> int:
    if "ignore previous instructions" in prompt.lower():
        return 95 # Blocked by playbook
    return 5
5. NIST AI Risk Management Framework Alignment
NIST Function	GenAI.mil Implementation	IL5/IL6 Control
Govern (GV)	CDAO ownership, continuous red-teaming, CUI-by-default	GV-1, GV-2 risk tolerance
Map (MP)	Prohibited Data Notice filters, classification boundaries, RBAC scoping	MP-2 data validity
Measure (ME)	Automated risk scoring, toxic drift detection, vulnerability scanning	ME-2, ME-3
Manage (MG)	ZTA + CAC, MCP+RBAC, sovereign hosting, audit logging	MG-1, MG-2, MG-4
6. Contractual Data-Privacy Parameters (Vendor Requirements)
What DoW requires of commercial frontier providers:

No Training: Prompts, outputs, metadata cannot be used to train/fine-tune commercial models
Sovereign Hosting: Dedicated government cloud layers, not multi-tenant commercial endpoints
Data Ownership: Enterprise retains ownership, with deletion and audit guarantees
IL Authorization Inheritance: Vendor must maintain FedRAMP High equivalency for IL5 and support secure pipeline integration for IL6/IL7
7. How to Run This Reference Stack
bash
pip install fastapi uvicorn
uvicorn genai_mil_reference_stack:app --reload
Frontend:

bash
# React / Next.js
# POST /chat with Header X-CAC-Cert: DOD-CAC-XXXX
8. Why This Matters
This architecture solves the real bottleneck in DoW AI adoption: not model capability, but secure, compliant, governable scale. The focus on MCP, RBAC, and sanitization is what enables millions of users to build agents without creating a data spillage incident.

Author: Alana Battaile — @princess38827 on GitHub
Focus: Secure AI Infrastructure, Zero Trust, FedRAMP/IL5-7, Agent Governance
Repo: https://github.com/princess38827/genai-mil-secure-reference

