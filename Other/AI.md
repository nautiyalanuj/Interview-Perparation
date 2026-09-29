- Create custom agent using Github Copilot
  - Agents are mostly present in [/.github/agents](https://github.com/nautiyalanuj/Rate-Limiting/tree/main/.github/agents)
   <img width="468" height="214" alt="image" src="https://github.com/user-attachments/assets/b8bd1f1c-4393-4cad-b728-dfa8111aaef3" />
   
   - Its mostly a English language instruction, with tools(MCP) required.



|Term| Meaning| Analogy|
|-|-|-|
|Agent|AI worker that reasons and executes tasks| Employee|
|Instructions|Rules that guide the agent| Company Policies|
|Skills|Capabilities the agent can perform| Reusable task, like creating excel|
|Tools|Functions/APIs the agent can call|
|MCP|Standard protocol for accessing tools|
|Plugin (Awesome-Copilot)|Packaged expertise, instructions, prompts, and skills for a domain| Training Course|

- Agent vs Plugin

|Question|	Agent|	Plugin|
|-|-|-|
|Can reason?|	✅	|❌|
|Can plan?|	✅	|❌|
|Can decide next step?|	✅	|❌|
|Contains instructions?|	✅|	✅|
|Contains skills?|	✅|	✅|
|Executes tasks?|	✅|	❌|
|Active component?|	✅|	❌|
  - Generic Agent + C# Plugin == C# Expert Agent
