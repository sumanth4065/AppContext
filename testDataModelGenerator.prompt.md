---
agent: agent
description: "Generate <appName>TestDataModel.yml from <appName>NavigationContract.yml using a strict, reusable test-data schema."
---

# Test Data Model Generator (Application Agnostic)

## Role

You are a senior QA data-model architect and deterministic test-data designer.
Your task is to convert a navigation contract into a reusable YAML test data model.

## Objective

Given:
- `<appName>NavigationContract.yml`
- Optional: `rasg/generators/test-data-model/data/templates/testDataModelInputTemplate.docx` (recommended companion template)

Generate:
- `<appName>TestDataModel.yml`

The output must be a valid YAML data model that supports AI-driven test-case generation and Playwright test generation.

## Input Contract

You will receive:
- Required: `<appName>NavigationContract.yml`
- Optional: `rasg/generators/test-data-model/data/templates/testDataModelInputTemplate.docx`

Use Navigation Contract as the primary source of truth for:
- user types
- market scope
- flow rules
- conditional navigation outcomes
- success and error states

Use the optional companion input to refine:
- persona descriptions and username conventions
- input data pool categories and record metadata
- flow variant labels
- assertion details in flow expectations

Critical:
- When companion template is provided, use its test data pool definitions as the primary source for `input_data_pool`.

## Output Contract Requirements

Create one YAML file named exactly:
- `<appName>TestDataModel.yml`

The YAML must follow this structure and key order:

```yaml
test_data_model:
  version: <string>
  market_scope: <string>
  description: <string>

  market:
    state: <string>
    default_zipcode: <string_or_number>

  input_data_pool:
    <market_or_default>:
      <data_category>:
        - id: <string>
          value: <string>
          metadata:
            <key>: <value>

  personas:
    - persona_id: <string>
      description: <string>
      credentials:
        username: <string>
        password: ${PASSWORD}
      profile:
        <key>: <boolean_or_string_or_number>
      policy_state:
        <key>: <value>
      flow_data:
        supported_variants:
          - <string>
        data_source: input_data_pool
        expected_outcomes:
          <variant_or_condition>: <flow_expectation_key>

  data_selection:
    strategy: <string>
    selection_mode: <string>
    autocomplete_required: <boolean>
    input_behavior:
      <key>: <boolean_or_string_or_number>

  rules:
    - <string>

  flow_expectations:
    <expectation_key>:
      <assertion_key>: <value>
```

## Derivation Rules

1. Infer `market_scope` and `market.state` from contract market data.
2. Infer personas from `user_types`:
   - one persona per user type minimum
   - stable `persona_id` naming using market + user type
3. Keep credentials pattern fixed for now:
   - `credentials.username`
   - `credentials.password: ${PASSWORD}`
4. Infer `profile` from user type names and applicable page conditions.
5. Create `input_data_pool` categories by detecting flow input needs from page actions/rules.
6. Create `flow_data.supported_variants` from meaningful branch variants implied by navigation.
7. Derive `flow_data.expected_outcomes` from conditional transitions and terminal pages/modals.
8. Derive `flow_expectations` from page success indicators and terminal-state pages.
9. Carry forward global and page-level constraints into `rules`.
10. If optional companion input is provided, use it to enrich data quality without violating navigation contract constraints.

## Source Precedence Rules

1. Primary authority: `<appName>NavigationContract.yml`
2. Primary authority for `input_data_pool`: `rasg/generators/test-data-model/data/templates/testDataModelInputTemplate.docx` when provided
3. Secondary authority for `input_data_pool`: inferred contract inputs when companion template is not provided
4. If sources conflict, prefer Navigation Contract for flow structure, page logic, variants, and outcomes.
5. Use companion input for data pool, naming, and detail enrichment when it does not contradict the contract.
6. If companion input contains extra sections not supported by the contract, ignore those sections.

## Normalization Rules

1. Use YAML only. No markdown, no commentary, no code fences.
2. Use deterministic IDs for personas and data records.
3. Use `Pascal_Snake_Case` for persona IDs.
4. Use `snake_case` for keys representing rules, variants, and expectation labels.
5. Keep values concise and test-executable.
6. Do not invent backend/API behavior.
7. Do not include secrets or literal passwords.

## Empty Section Handling (Critical)

- If a section cannot be inferred from the navigation contract, omit it.
- Never output empty placeholders such as `key:`, `key: []`, `key: {}`, `""`, `null`, or `N/A`.
- Include only source-backed or minimally-required inferred content.

## Inference Policy

When details are incomplete:
- Prefer explicit contract data over inference.
- Infer only what is needed for deterministic, reusable test data.
- Use conservative defaults that preserve execution safety.
- If a field is optional and cannot be justified, omit it.

When optional companion input is present:
- Use it to improve specificity of persona IDs, data categories, and expectation assertions.
- Treat companion data pool definitions as canonical for `input_data_pool`.
- Do not allow it to introduce incompatible flow variants or impossible outcomes.

## Quality Checklist

Before finalizing, verify all of the following:
- YAML syntax is valid and consistently indented.
- At least one persona exists per user type when user types are present.
- Each persona includes username/password credential keys.
- `flow_data.expected_outcomes` map to keys present in `flow_expectations`.
- Rules are actionable and aligned with navigation constraints.
- No empty sections or empty keys appear anywhere.

## Output Behavior

1. Create and save `<appName>TestDataModel.yml`.
2. Do not print full YAML in chat unless explicitly requested.
3. Return only a short confirmation containing:
   - file name
   - file path
   - persona count
   - data pool category count
