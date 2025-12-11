# Rules File Configuration

The `template-rules.json` file is central to how the InDegreeLib application generates Google Forms, processes survey responses, and prepares data for social network analysis. It defines question blocks, scoring methodologies, and network edge extraction rules.

## Top-Level Structure

The rules file is a JSON object with three main top-level keys:

*   `blocks`: Defines the structure and content of the questions to be generated in the Google Form.
*   `processing`: Specifies how to process the raw responses, including scoring, aggregation, and edge extraction.
*   `network`: Defines the final output structure for network analysis (vertices and edges).

---

## `blocks` Object

This object contains definitions for different sets of questions, often corresponding to psychological measures or demographic sections. Each key within the `blocks` object represents a unique question block.

**Example Block Structure (e.g., `flourishing`):**

```json
"flourishing": {
    "reference": "Flourishing (Diener-8) [Validated Scale]",
    "type": "measure_set",
    "concept": "Psychological flourishing (meaning, relationships, engagement, optimism, purpose).",
    "description": "Measures meaning, relationships, engagement, optimism, and purpose.",
    "sections": [
        {
            "title": "Flourishing",
            "description": "Below are 8 statements...",
            "items": [
                {
                    "questionType": "multiple_choice_grid",
                    "required": false,
                    "labels": {
                        "Strongly disagree": 1,
                        "Strongly agree": 7
                    },
                    "questions": [
                        {
                            "id": "fl_competent",
                            "text": "I am competent and capable..."
                        }
                    ]
                }
            ]
        }
    ]
}
```

### Block Properties

*   `reference` (string, optional): A reference or source for the measure (e.g., a validated scale name).
*   `type` (string): The type of block. Common values include `measure_set` (for a set of related questions) or `section` (for a general section).
*   `concept` (string, optional): A brief description of the concept the block measures.
*   `description` (string, optional): A more detailed description of the block.
*   `sections` (array of objects): An array defining the sections within this block.

### Section Properties

Each object in the `sections` array can have the following properties:

*   `title` (string): The title of the section as it appears in the Google Form.
*   `description` (string): Descriptive text for the section.
*   `notes` (string, optional): Internal notes, not displayed in the form.
*   `items` (array of objects): An array defining the individual question items within this section.

### Item Properties

Each object in the `items` array defines a single question or a group of related questions.

*   `questionType` (string): Specifies the type of Google Form question.
    *   `multiple_choice_grid`: A grid where respondents select one option per row.
    *   `linear_scale`: A scale with a defined range and labels for endpoints.
    *   `multiple_choice`: A single-choice question from a list of options.
    *   `dropdown`: A single-choice question from a dropdown list.
    *   `checkbox_grid`: A grid where respondents can select multiple options per row.
    *   `short_answer`: A single-line text input.
*   `required` (boolean): If `true`, the question must be answered.
*   `labels` (object): Maps display labels to internal values.
    *   For `multiple_choice_grid`, `linear_scale`, `multiple_choice`, `checkbox_grid`: Key-value pairs where the key is the display text and the value is the associated score/value.
    *   For `checkbox_grid` where each label should be a checkbox, the value can be `true`.
*   `questions` (array of objects): An array of individual questions within this item.
*   `rosterField` (string, optional): Used for question types like `dropdown` or `checkbox_grid` where options are dynamically populated from a participant 'roster' (e.g., "name" to list all participant names).
*   `additionalRows` (array of strings, optional): For `checkbox_grid`, provides additional static options beyond those generated from the roster (e.g., "Name not listed").
*   `routing` (object, optional): Defines conditional logic for `dropdown` questions to control form navigation.
    *   `name` (string): The display text of the option that triggers routing.
    *   `gotoMatch` (string): Action if the option matches (e.g., `"next_section"`, `"skip_next_section"`).
    *   `gotoElse` (string): Action if the option doesn't match.

### Question Properties (within `items.questions`)

*   `id` (string): A unique identifier for the question. This ID is crucial for linking responses to scoring and processing rules.
*   `text` (string): The actual question text displayed in the Google Form.

---

## `processing` Object

This object defines how the raw survey responses are transformed into structured data suitable for analysis.

### `roster` Array

(array of strings): Lists fields from the participant roster that are relevant for processing, such as `"group"` or `"node_selector"`.

### `scoring` Array

(array of objects): Defines how to calculate composite scores from individual question responses.

**Scoring Item Properties:**

*   `method` (string): The aggregation method.
    *   `mean`: Calculates the average of the input question values.
    *   `sum`: Calculates the sum of the input question values.
    *   Custom methods like `"ab1234"` might be used for specific transformations.
*   `output` (string): The name of the resulting score variable.
*   `inputs` (array of strings): An array of `id`s of the questions whose responses should be used for this scoring calculation.
*   `reverse` (array of strings, optional): An array of `id`s from the `inputs` list whose values should be reverse-scored before aggregation. This is common in psychological scales.

### `phaseAggregation` Array

(array of objects): Defines aggregations that might occur across different phases of data collection, often used for calculating changes over time or other advanced metrics.

**Aggregation Item Properties:**

*   `method` (string): The aggregation method (e.g., `"difference"`, `"mean"`, `"max"`).
*   `output` (string): The name of the resulting aggregated variable.
*   `inputs` (array of strings): An array of variable names (which might be outputs from `scoring` or raw question `id`s) to use for this aggregation.

### `edges` Object

(object): Defines rules for extracting "edges" (relationships) between participants from specific question types, typically `checkbox_grid` questions where respondents select other participants.

**Edge Definition Properties (e.g., `closest_edges`):**

*   `method` (string): The method used to extract edges, typically `"checkbox_grid"`.
*   `input` (string): The `id` of the form question from which to extract edges.
*   `rosterField` (string): The field from the roster that was used to populate the options in the `checkbox_grid` (e.g., `"name"`).
*   `attributes` (array of strings): Defines the attributes to be included with each extracted edge. Common attributes include:
    *   `"source_name"`: The name of the person filling out the form.
    *   `"target_name"`: The name of the person selected in the checkbox.
    *   `"column.text"`: The text of the column label that was checked for the target.

### `post` Array

(array of objects): Defines post-processing steps, often for calculating network metrics from the extracted edges.

**Post-processing Item Properties:**

*   `method` (string): The calculation method (e.g., `"count"`).
*   `type` (string): The type of data being processed (e.g., `"edge"`).
*   `inputs` (array of strings): An array of edge definition names (from the `edges` object) to use as input.
*   `match` (string): The field to match (e.g., `"id"`).
*   `matchTo` (array of strings): The field(s) to match against (e.g., `["target"]` to count in-degree).
*   `output` (string): The name of the resulting network metric.

---

## `network` Object

This object specifies which variables and edges should be included in the final GML (Graph Modelling Language) output for network analysis.

### `outputs` Object

*   `vertices` (array of strings): A list of variable names (from scoring, aggregation, or raw question `id`s) that should be included as attributes for each vertex (participant) in the GML file.
*   `edges` (array of strings): A list of edge definition names (from the `edges` object) that should be included in the GML file.
