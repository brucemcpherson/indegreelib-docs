# UI Workflow

This document explains the user interface workflow for the InDegree addon. The UI is built with Vue.js and Vuetify and is presented as a sidebar in the Google Workspace application.

## 1. Configuration Management

The first step is to either select an existing configuration or create a new one.

*   **Select a configuration**: A dropdown menu lists all available configurations. Selecting one will load its settings and data.
*   **Create a new configuration**: Click the "Create" button to open a dialog where you can enter a name for the new configuration.
*   **Delete a configuration**: Once a configuration is selected, a "Delete" button appears. Clicking it will permanently remove the configuration.

## 2. Main Workflow Tabs

The main workflow is organized into four tabs: Roster, Form, Graph, and Settings.

### Roster Tab

This tab is for managing the list of people who will be part of the analysis (the "roster").

*   **Roster Sheet**: Select the Google Sheet that contains the roster from a dropdown.
*   **Roster Count**: The UI displays the number of people found in the selected roster sheet.
*   **Next**: Once a roster with at least one person is selected, you can proceed to the "Form" tab by clicking the "Next" button.

### Form Tab

This tab is for creating and managing the Google Form that will be sent to the people in the roster.

*   **Form Template**: Select a Google Form to be used as a template for the Google Form.
*   **Processing Rules**: Select a rules file that defines how the form responses will be processed.
*   **Description**: Enter a description for the form.
*   **Phases**: The workflow is divided into "phases". You can select an existing phase or create a new one. Each phase can have its own form.
*   **Create Form**: Once the template, rules, and phase are set, you can create the form by clicking the "Create Form" button.
*   **Form Details**: After a form is created, the UI displays details about the form, such as the form's name, title, and description.
*   **Next**: Once at least one form has been created, you can proceed to the "Graph" tab by clicking the "Next" button.

### Graph Tab

This tab is for generating and exporting the network graph from the form responses.

*   **GraphML Export Name**: Specify a name for the exported GraphML file.
*   **Merge Configurations**: You can merge the data from the current configuration with other configurations.
*   **Responses**: The UI displays a table of the form responses, showing the number of responses for each phase.
*   **Filter by Attribute**: You can filter the nodes in the graph by specific attributes.
*   **Select Phases**: Choose which phases to include in the graph.
*   **Make Graph**: Click the "make" button to generate the GraphML file.
*   **Gephi**: Click the "gephi" button to open the generated graph in Gephi.

### Settings Tab

This tab is for configuring general settings for the addon.

*   **Template Folder ID**: The ID of the Google Drive folder that contains the form templates.
*   **Help Email**: The email address for users to contact for help.
*   **Output Folder ID**: The ID of the Google Drive folder where the output files will be saved. If left blank, the root of your Google Drive will be used.
*   **Organize into Subfolders**: If enabled, the output files will be organized into subfolders.
*   **Save/Cancel**: You can save or cancel your changes to the settings.
