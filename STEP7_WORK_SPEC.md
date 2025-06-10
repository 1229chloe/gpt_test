📌[Declaration]
This specification defines the instruction set for Codex to generate a fully hardcoded implementation of step7, which must be executed directly after step6 without modifying any existing data structure, variable, or session key used in step1_to_6.py.

The goal is to convert structured data and interpreted condition logic into deterministic, fully hardcoded code blocks without introducing any new variables or logic outside the provided scope.

The following reference files must be strictly used:

- `step7_data.xlsx`:  
  Contains human-readable column values which describe conditions and expected outputs.  
  These values (e.g., subitem_met, requirements_met, requirements_unmet) are not executable logic themselves but must be **interpreted into if-condition logic by Codex**, based on actual user responses from step6.

- `step1_to_6.py`:  
  Defines the full implementation of steps 1 through 6 in Streamlit.  
  In particular, the structure of `step6_items` and the state keys used in `st.session_state.step6_selections` must be preserved and reused exactly.

- `step6_used_key_info.csv`:  
  Lists the exact keys that were declared and used during step6 logic execution.  
  Codex must use this list as the **only valid reference** for conditional key matching in step7. No derived, renamed, or inferred keys are allowed.

Key instructions for Codex:

- All logic for step7 must derive from the provided condition values.  
  Codex is responsible for transforming the values in `subitem_met`, `requirements_met`, and `requirements_unmet` into complete logical conditions (e.g., if A and B and not C…).

- A result is shown (from `result_1`, `result_2`, `result_1_tag`) **only if all listed conditions for that row are satisfied.**  
  Otherwise, the default fallback message (as defined later in this spec) must be displayed.

- The output text, formatting, and structure from the Excel cells must be preserved 1:1, including line breaks, markdown, numbering, or indentation.

- No new logic, shortcuts, or abstractions are allowed.  
  The implementation must be deterministic, explicit, and fully based on the instructions and resources above.

This declaration precedes and governs all subsequent task instructions. Codex must treat this section as binding logic and not deviate from it.
────────────────────────────────────────────────────────────────────────────
📌[Purpose of This Specification]
The purpose of this document is to instruct Codex to generate a hardcoded implementation of step7 logic that is fully aligned with:

1. The user selection states stored in `st.session_state.step6_selections` (as implemented in `step1_to_6.py`)
2. The interpretable condition values provided in `step7_data.xlsx`
3. The exact key mapping declared in `step6_used_key_info.csv`

Step7 must evaluate the user’s previous input (step6) and apply a matching logic to determine which predefined output (defined in the Excel) should be shown.

Codex must treat this logic as fully deterministic:  
every conditional rule is derived from structured inputs, and the output must follow only when all specified values in the row are satisfied.

This specification serves as the complete reference for Codex to implement step7 without additional assumptions, data, or dependencies.

──────────────────────────────────────── Step 7 Specification (Codex-ready) ────────────────────────────────────────
NOTE – All identifiers, column names, and UI strings in Korean **must stay exactly as-is.**  
This document contains **no executable code** – only the deterministic rules Codex must follow when it appends
Step 7 logic after the existing `step1_to_6.py` file.

───────────────────────────── 1. Fixed Data Sources ─────────────────────────────
EXCEL_FILE  :  `step7_data.xlsx`

COLUMNS (exact order in the worksheet)  
  `step`, `heading_text`, `title_key`, `title_text`,  
  `subitem and requirements steat`, `output_condition_all_met`,  
  `subitem_met`, `requirements_met`, `requirements_unmet`,  
  `output_1_tag`, `output_1_text`, `output_2_text`

───────────────────────────── 2. Objects Provided by Step 6 ─────────────────────
`step6_targets`      → list[str]   # title_key values selected by the user, in display order  
`step6_selections`   → dict        # key → "변경 있음" / "충족" / "미충족"  
`step6_items`        → dict        # title_key → {"title": (str with line-breaks)}

───────────────────────────── 3. New Session Variables for Step 7 ───────────────
`step7_page`    (int)   → current page index, 0-based  
`step7_results` (dict)  → {title_key: [(output_1_tag, output_1_text, output_2_text), …]}

───────────────────────────── 4. Fixed UI Button Labels ────────────────────────
"이전단계로"  (offset −1)  
"다음단계로"  (offset +1)  
"신청양식 확인하기"  (offset +1, shown only on the final page)

───────────────────────────── 5. Deterministic Processing Rules ────────────────
5-1  **Page construction**  
     • One page per `title_key` in *step6_targets*.  
     • `total_pages  = len(step6_targets)`  
     • `current_key  = step6_targets[step7_page]`

5-2  **Header rendering (every page)**  
     `## 제조방법 변경에 따른 필요서류 및 보고유형`  
     `Subtitle = step6_items[current_key]["title"]` (line-breaks preserved).

5-3  **Row evaluation for current_key**  
     For each worksheet row where column *title_key* == current_key  
       a. Strip leading `"if "` and trailing `":"` from *output_condition_all_met*  
       b. Evaluate with `eval(expr, {}, {"step6_selections": step6_selections})`  
       c. If the expression is **True**, mark the row as *hit*.

5-4  **Page output**  
     • If ≥ 1 rows are hit → display each hit sequentially:  
       – `output_1_text` (unsafe HTML markdown)  
       – `output_2_text` (unsafe HTML markdown)  
       – Append `(output_1_tag, output_1_text, output_2_text)` to `step7_results[current_key]`.  
     • If 0 rows are hit → show once:  
       ```
       해당 변경사항에 대한 충족조건을 고려하였을 때,
       「의약품 허가 후 제조방법 변경관리 가이드라인」에서 제시하고 있는
       범위에 해당하지 않는 것으로 확인됩니다
       ```

5-5  **Navigation logic (per page)**  
     • "이전단계로"  → `step7_page -= 1`  (disabled on first page)  
     • "다음단계로"  → `step7_page += 1`  (hidden on last page)  
     • "신청양식 확인하기" → `st.session_state.step = 8`  (visible only on last page)

───────────────────────────── 6. Data-Integrity Rules ──────────────────────────
• **No** new column names, session keys, or functions may be invented.  
• Preserve Korean UI strings and cell formatting (whitespace, bullets, line-breaks).  
• Do **not** alter worksheet text when displaying it.

───────────────────────────── 7. Step 8 Dependency ─────────────────────────────
`step7_results` is the sole input for Step 8.  
Each tuple `(output_1_tag, output_1_text, output_2_text)` collected above must remain intact so Step 8 can generate
the final consolidated guidance and form-filling summary for the user.

────────────────────────────────────────────────────────────────────────────────
