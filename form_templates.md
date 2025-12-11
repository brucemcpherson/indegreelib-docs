# How to Set Up a Form Template with `{{blocks}}` Syntax

This document explains how to dynamically generate Google Forms using the `{{blocks}}` syntax within a template form and a `rules` JSON object. This system allows for flexible form creation by inserting predefined question blocks into a template form.

## Core Concept: Dynamic Form Generation

The `{{blocks}}` syntax leverages a template Google Form that contains special "placeholder" items. These placeholders are identified by JSON objects embedded in their title or help text. When the form generation process runs, these placeholders are replaced or expanded by predefined "blocks" of questions and sections found in a `rules` JSON object.

## The `rules` JSON Object

The core of dynamic form generation is a `rules` JSON object (typically loaded from a file, e.g., `template-rules.json`). This object defines various aspects of form processing and, crucially, contains the `blocks` definition.

A simplified structure of the `rules` object relevant to blocks:

```json
{
  "blocks": {
    "your_block_name_1": {
      // Definition for block 1
    },
    "your_block_name_2": {
      // Definition for block 2
    }
  },
  "processing": {
    // Other processing rules (scoring, aggregation, etc.)
  },
  "network": {
    // Network output definitions
  }
}
```

## Block Definition

The `blocks` object contains multiple named definitions, where each key is a unique identifier for a block (e.g., `"flourishing"`, `"happiness"`, `"demographics"`).

Each block can have the following properties:

*   `reference` (string, optional): An internal reference or metadata for the block.
*   `type` (string, optional): A categorizing type (e.g., `"measure_set"`).
*   `concept` (string, optional): A brief description of the block's purpose or concept.
*   `description` (string, optional): A more detailed description of the block.
*   `sections` (array, **required**): An array of section objects that define the structure and content of the questions within this block.
*   `next_block` (string, optional): The name of another block to navigate to if routing conditions are met.
*   `skip_next_block` (string, optional): The name of another block to skip to if routing conditions are met.

**Example Block (`flourishing`):**

```json
"flourishing": {
  "reference": "Flourishing (Diener-8) [Validated Scale]",
  "type": "measure_set",
  "concept": "Psychological flourishing (meaning, relationships, engagement, optimism, purpose).",
  "description": "Measures meaning, relationships, engagement, optimism, and purpose.",
  "sections": [
    {
      "title": "Flourishing",
      "description": "Below are 8 statements with which you may agree or disagree...",
      "items": [
        {
          "questionType": "multiple_choice_grid",
          "required": false,
          "labels": {
            "Strongly disagree": 1,
            "Disagree": 2,
            "Slightly disagree": 3
            // ... more labels
          },
          "questions": [
            { "id": "fl_competent", "text": "I am competent and capable..." },
            { "id": "fl_good", "text": "I am a good person..." }
            // ... more questions
          ]
        }
      ]
    }
  ]
}
```

## Sections within a Block

Each block consists of one or more `sections`. Each `section` object defines a logical grouping of questions within the form.

*   `title` (string): The title of the section.
*   `description` (string): The description or introductory text for the section.
*   `notes` (string, optional): Internal notes not displayed in the form.
*   `items` (array, **required**): An array of item objects, each representing a single question or form element.

## Items (Questions)

Each `item` object within a section defines a specific question in the Google Form.

**Common Properties for All Item Types:**

*   `questionType` (string, **required**): Specifies the type of Google Form question. Valid types include:
    *   `multiple_choice_grid`
    *   `checkbox_grid`
    *   `linear_scale`
    *   `multiple_choice`
    *   `dropdown`
    *   `short_answer`
*   `required` (boolean, optional): Set to `true` if the question must be answered.
*   `id` (string): A unique identifier for the question. This `id` is crucial for linking responses to `processing` rules (e.g., scoring).
*   `text` (string): The actual question text (used within the `questions` array for grid/scale/dropdown items).
*   `title` (string): The main title for the item (used for some question types like grids).

**Specific Properties by `questionType`:**

### `multiple_choice_grid` & `checkbox_grid`

These types create grid-style questions where rows are defined by `questions` and columns by `labels`.

*   `labels` (object): A mapping of choice text (column headers) to their associated values.
    ```json
    "labels": {
      "Not at all": 0,
      "Several days": 1,
      "More than half the days": 2,
      "Nearly every day": 3
    }
    ```
*   `questions` (array): An array of objects, where each object defines a row in the grid.
    ```json
    "questions": [
      { "id": "de_interest", "text": "Little interest or pleasure in doing things." },
      { "id": "de_down", "text": "Feeling down, depressed, or hopeless." }
    ]
    ```
*   `rosterField` (string, optional): If present, the rows will be dynamically populated from a roster, using the value of this field from each roster member as a row label.
*   `additionalRows` (array, optional): An array of strings for additional static rows to be appended to dynamically generated rows (e.g., `"Name not listed"`).

### `linear_scale`

Creates a scale question (e.g., 1 to 7).

*   `labels` (object): Defines the start and end labels and their values for the scale.
    ```json
    "labels": {
      "not a very happy person": 1,
      "a very happy person": 7
    }
    ```
*   `questions` (array): An array of objects, each representing a single linear scale question.
    ```json
    "questions": [
      { "id": "hs_consider", "text": "In general, I consider myself:" }
    ]
    ```

### `multiple_choice` & `dropdown`

These create single-choice questions.

*   `labels` (object): A mapping of choice text to their associated values.
    ```json
    "labels": {
      "Yes, I am currently a leader": 1,
      "No, but I have been a leader previously": 2
    }
    ```
*   `questions` (array): An array of objects, typically containing a single question definition.
    ```json
    "questions": [
      { "id": "de_leader", "text": "Are you currently a leader in your sorority?" }
    ]
    ```
*   `rosterField` (string, optional): If present, the dropdown/multiple-choice options will be dynamically populated from a roster, using the value of this field from each roster member as an option.
*   `routing` (object, optional): Defines conditional navigation based on the selected option for `dropdown` items.
    *   `name` (string): The option text that triggers the routing.
    *   `gotoMatch` (string): Action to take if `name` is selected (`"next_section"`, `"skip_next_section"`, `"submit"`).
    *   `gotoElse` (string): Action to take if `name` is not selected (`"next_section"`, `"skip_next_section"`, `"submit"`).

### `short_answer`

Creates a short text input field.

*   `questions` (array): An array of objects, typically containing a single question definition.
    ```json
    "questions": [
      { "id": "my_name", "text": "My name" }
    ]
    ```

## Placeholder Item in the Google Form Template

To utilize these blocks, your Google Form template must contain placeholder items. These are standard form items (e.g., a Section Header, or any question type) whose **title** or **help text** contains a JSON object enclosed in double curly braces, referencing a block name from your `rules` object.

**Example Placeholder Text:**

```
{{ "block": "flourishing" }}
```

When the form is generated:

1.  The system finds this placeholder item.
2.  It looks up the "flourishing" block in your `rules` object.
3.  The title and description of the placeholder item itself are updated using the `title` and `description` of the *first section* within the "flourishing" block.
4.  All `items` defined in *all sections* of the "flourishing" block are then inserted into the form immediately after this placeholder item.

By using this `{{blocks}}` syntax, you can create modular and dynamic forms that are easily configurable through a JSON `rules` object.
