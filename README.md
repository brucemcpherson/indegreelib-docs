# InDegreeLib Backend

This project is a Google Apps Script application for conducting social network analysis surveys. It can be deployed as a web app and as a Google Workspace Add-on.

## Core Functionality

The application automates the process of generating, distributing, and analyzing network-style surveys. The workflow is as follows:

1.  **Setup**: An administrator defines a 'roster' of participants in a Google Sheet, 'rules' for question generation in a JSON file, and a 'template' Google Form.
2.  **Form Generation**: The application programmatically generates a Google Form by merging the roster with the rules. This creates customized questions for each participant (e.g., 'Rate your interaction with Person X'). This is orchestrated by `Googly.js` calling `BlockHandler.js`.
3.  **Data Collection**: Participants fill out the generated forms.
4.  **Data Ingestion**: The application collects the form responses and processes them into a graph data structure, consisting of 'vertices' (the participants) and 'edges' (the relationships and their ratings between participants). This is handled by `Googly.js` and `ResponseHandler.js`.
5.  **Data Export**: The resulting graph is saved in two formats: dumped into a Google Sheet for easy viewing and rendered as a GML (Graph Modelling Language) file for use in external graph analysis and visualization software. The GML generation is handled by `Gml.js`.
6.  **Secure Download**: The application provides a secure, temporary, one-time-use download link for the generated GML file, using a clever system of dual caching (`UserCache` and `ScriptCache`) implemented in `ContentServe.js`.

## How to Use It (as a developer)

1.  **Local Development**: Write modern JavaScript (ESM) in the `src` directory.
2.  **Testing**: Use the `@mcpher/gas-fakes` library to test Google Apps Script services locally.
3.  **Bundling**: The `fromgas:build` script (or similar) in `package.json` uses `esbuild` to bundle the entire `src` directory into a single file compatible with the Google Apps Script V8 runtime.
4.  **Deployment**: The `togas.sh` script pushes the bundled code to a linked Google Apps Script project. The project is deployed as a web app.

## Key Architectural Points

*   **Facade Pattern**: `Googly.js` acts as a central facade, providing a clean API for the client-side UI and orchestrating the various backend services (`StoreHandler`, `DriveHandler`, `BlockHandler`, etc.).
*   **Client-Server Communication**: The frontend (in the `src/client` directory) communicates with the backend via `google.script.run`, which calls the `runWhitelist` function in `Googly.js`. This provides a secure way to expose specific server-side functions to the client.
*   **Secure Content Delivery**: `ContentServe.js` provides a robust mechanism to serve sensitive, dynamically generated files to users without exposing long-lived credentials or URLs.
*   **Configuration Management**: The application's state and different survey configurations are managed via `StoreHandler.js`, which likely uses `PropertiesService` as a key-value store.
