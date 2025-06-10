📌[Declaration]
This specification defines the instruction set for Codex to generate a fully hardcoded implementation of step7, which must be executed directly after step6 without modifying any existing data structure, variable, or session key used in step1_to_6.py.

The goal is to convert structured data and interpreted condition logic into deterministic, fully hardcoded code blocks without introducing any new variables or logic outside the provided scope.

The following reference files must be strictly used:

- `step7_data_refac.xlsx`:
  Contains human-readable column values which describe conditions and expected outputs.  
  These values (e.g., subitem_met, requirements_met, requirements_unmet) are not executable logic themselves but must be **interpreted into if-condition logic by Codex**, based on actual user responses from step6.

- `step1_to_6.py`:
  Defines the full implementation of steps 1 through 6 in Streamlit.
  In particular, the structure of `step6_items` and the state keys used in `st.session_state.step6_selections` must be preserved and reused exactly. Step 7 and Step 8 code must be appended to this file rather than placed in a new file.

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
2. The interpretable condition values provided in `step7_data_refac.xlsx`
3. The exact key mapping declared in `step6_used_key_info.csv`

Step7 must evaluate the user’s previous input (step6) and apply a matching logic to determine which predefined output (defined in the Excel) should be shown.

Codex must treat this logic as fully deterministic:  
every conditional rule is derived from structured inputs, and the output must follow only when all specified values in the row are satisfied.

This specification serves as the complete reference for Codex to implement step7 without additional assumptions, data, or dependencies.

──────────────────────────────────────── Step 7 Specification (Codex-ready) ────────────────────────────────────────
NOTE – All identifiers, column names, and UI strings in Korean **must stay exactly as-is.**  
This document contains **no executable code** – only the deterministic rules Codex must follow when it appends
Step 7 and Step 8 logic after the existing `step1_to_6.py` file. Do not place this code in a new file.

───────────────────────────── 1. Fixed Data Sources ─────────────────────────────
EXCEL_FILE  :  `step7_data_refac.xlsx`

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
Examples:

* If a condition states that step6\_selections.get("s2\_2\_req\_3") == "충족", and that condition is met, the corresponding output\_1\_text and output\_2\_text must be rendered in full.
* Results are collected into step7\_results\[title\_key] = \[(tag, output1, output2), ...] for use in step8.

The fallback message shown when no conditions match:
해당 변경사항에 대한 충족조건을 고려하였을 때,
「의약품 허가 후 제조방법 변경관리 가이드라인」에서 제시하고 있는
범위에 해당하지 않는 것으로 확인됩니다

⚠ All formatting and structure for matched outputs must follow exactly what is defined in the Excel rows. Do not paraphrase or simplify any string.

Hardcoding must be preserved at all times.

In STEP7, the 75 independently evaluated cases must be regrouped by title\_key and the results generated per title\_key grouping using fully hardcoded logic. Focus only on this task. Any additional enhancements will be considered later.

Even if one or more of the grouped cases under a title\_key produces results, when none match, the fallback guidance message must be shown. Steps 1 to 6 are functioning correctly and must not be modified.

Each individual case sharing the same title\_key must be evaluated independently, and their respective outputs collected under a common title\_key grouping.

Rendering logic for each of the 75 condition cases according to paging rules:
Each of the 75 condition cases must store their evaluation results independently.
However, based on paging rules, conditions with the same title\_key are grouped into a single screen. If multiple conditions produce results, all outputs are displayed in parallel on the same page.
If no condition under a given title\_key produces a result, then a fallback message must be shown.

IMPORTANT: A detailed example is included at the end of this specification.

This logic must be applied to all 24 defined title\_key groupings.

──────────────────────────────────────── Step 8 Specification (Codex-ready) ────────────────────────────────────────

Step 8 uses the results from step7 to auto-populate an official PDF application form following the exact template defined in step8\_양식.docx and step8프롬프트\_출력서식.docx.

Page generation:

* Each title\_key with any valid result from step7 must generate one full page.
* Pages must repeat based on the number of title\_keys in step7\_results.

Form population rules:

* Section 1 (신청인): Leave blank.
* Section 2 (변경유형): Use title\_text from the matched title\_key.
* Section 3 (신청유형): Use the first line of output\_1\_text (e.g., “IR”, “Cmaj”, etc.).
* Section 4 (충족조건): For each requirement from step6\_selections linked to that title\_key:

  * List all keys used (e.g., s2\_2\_req\_1 \~ req\_10) with their literal prompt text.
  * Next to each, show “○” if value is “충족”, “×” if “미충족”.
* Section 5 (필요서류):

  * From output\_2\_text, split by line breaks.
  * For each line, enter in order and leave 구비여부 and page fields blank for manual input.

Output format:

* PDF must be generated with fixed layout matching the original Word template.
* Fonts, spacing, cell widths, numbering, and line breaks must be identical.
* File name: 신청양식\_{title\_key}\_{YYYYMMDD}.pdf
* PDF must be read-only, non-editable.

UI integration:

* On Step 8 page, show preview of filled forms with two buttons: \[파일 다운로드하기], \[인쇄하기].
* If no title\_keys yield a result, show only:
  신청양식 자동생성 불가: 도출 결과 없음

⚠ Codex must strictly follow the filled example layout and cannot infer field usage. Every label, cell value, and layout structure must match what is defined in the reference Excel and Word forms.

Step 8 implements the final output document: a fully formatted, printable application form based on the official 의약품 허가 후 제조방법 변경관리 가이드라인 template.

Purpose:

* Convert all final outputs from step6/step7 into the fixed layout of the government application form.
* Generate a read-only PDF using the official template, with data auto-filled from earlier steps.

Key Features:

* Each title\_key with valid results generates one full page of output (step8 pagination must follow title\_key).

* Form is populated with:

  * Section 2: Upper/mid/lower item from title\_text.
  * Section 3: 보고유형 from output\_1\_text.
  * Section 4: All condition keys from step6 shown with ○ (충족) or × (미충족).
  * Section 5: Required documents (output\_2\_text), listed in full by line with their checkmark status left blank.

* Section 1 (신청인), and any blank placeholders in the template, must remain empty for manual user input.

UI Rules:

* Dedicated page for form preview.
* Two fixed buttons: \[파일 다운로드하기] and \[인쇄하기].
* PDF must exactly match the font, spacing, layout, and phrasing of the attached Word/Template image.

Repeat Generation:

* Multiple title\_key results → repeat template on new pages.
* No valid results → print default page with message: 신청양식 자동생성 불가: 도출 결과 없음.

PDF Output:

* File name format: 신청양식\_{title\_key}\_{YYYYMMDD}.pdf.
* Must be read-only and uneditable.

⚠ All formatting, order, spacing, and phrasing must match the official template precisely.

Codex must render step8 only after all step7 results are finalized, using step7\_results as the sole data source.

Output fields must follow the structure defined in the filled version of the template Excel provided. Codex must extract the field mappings directly from the filled template structure and not infer or omit any logic.

[Project Purpose]
- This project is for the automatic generation of regulatory application forms (according to the “Guidelines for Managing Post-Approval Changes in Manufacturing Methods”)—specifically, the hardcoded implementation of STEP7 and STEP8.
- Users proceed through STEP6, making selections (Fulfilled/Not fulfilled, etc.), which are saved in st.session_state.step6_selections as a dict.
- All logic, structure, text, and templates for STEP7/STEP8 must be **strictly hardcoded**; no code inference, shortening, or text modification is allowed.

[Data and Code Structure]
- In step1_to_7 (4).py, the 75 evaluation cases are hardcoded in a dict list (e.g., STEP7_ROWS).
    - Each dict contains: title_key, output_condition_all_met (as string), output_1_text, output_2_text, and metadata.
    - output_condition_all_met is always a string (e.g., "step6_selections.get('s2_2_sub_2a') == 'Changed' and ..."), to be evaluated at runtime using eval().
- Each title_key (e.g., 's2_2', 'p7_14') defines one logical “page” (group); multiple cases may share a title_key.

[STEP7 Details: Independent Evaluation and Grouping]
1. **Each of the 75 evaluation cases in STEP7 must be implemented as a fully hardcoded if-block or eval block.**
2. **Cases with the same title_key are grouped as one “page”; results are displayed in parallel on that page.**
3. For each case:
    - Independently evaluate against the user’s STEP6 selections.
    - If one or more cases under a title_key match, output *all* of their results in parallel (sub-heading + output text).
    - If none match, display the fallback guidance message only (“No matching case…”).
4. **No for-loops, functions, or dynamic logic—everything must be hardcoded (no matter how verbose).**
5. Output text, formatting, numbering, markdown, etc. must be printed exactly as in STEP7_ROWS (no change allowed).

[Example Workflow]
- If there are 7 cases with title_key 's2_2', and user input matches 2, display both results as parallel blocks.
- If none match for 's2_2', display only the guidance message.

[STEP8 Details: Official Form PDF Generation]
1. For each title_key with results, generate a PDF page using the official template (provided Word/Excel/image).
2. Page and table structure, field names, cell positions, line breaks, etc. must be **100% identical** to the template.
3. Insert STEP7/6 output (application type, documents, conditions, etc.) exactly as output (no changes, translations, or formatting differences).
    - Applicant and manual fields remain blank.
    - Table borders, widths, fonts, colors, and line breaks must match template 1:1.
4. Implement PDF download/print buttons; PDF must be read-only.

[Error/Exception Handling]
- If none of the cases under a title_key match, display only the default guidance message.

[Critical Notes]
- No refactoring, dynamic code, or extra features outside this scope.
- Absolutely NO change (add, omit, alter) to output text, table structure, or logic.
- If unclear, clarify before proceeding.

[References]
- step1_to_7 (4).py: full code and logic
- step7_data_refac.xlsx: all original conditions and output text
- STEP8 Word/Excel/image: PDF layout and fields for strict matching
