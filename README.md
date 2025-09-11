## OCA Composer Iframe Embed

### Overview

This project demonstrates how to embed the OCA Composer into your own website using an iframe in a front-end application. The app loads a local JSON schema, sends it to the embedded OCA Composer via `postMessage`, and receives validated data back (as CSV) from the iframe for display and further use.

### Features

- **Schema selection**: Choose a JSON schema from `src/assets/config/schemas.json`.
- **Embedded Composer**: Opens OCA Composer inside an iframe.
- **One-click send**: Sends the selected schema to the Composer using `postMessage`.
- **Validated output**: Receives validated CSV data from the Composer and renders it in a table.

### Project Structure (key files)

- `src/App.vue`: Top-level UI; selects a schema and displays results.
- `src/components/ValidatorCard.vue`: Hosts the iframe and handles `postMessage` send/receive.
- `src/components/DataTable.vue`: Renders the returned CSV data as a table.
- `src/assets/config/schemas.json`: Lists available schema files to load and send.

---

## Getting Started

### Prerequisites

- Node.js and npm
- An accessible OCA Composer instance (default expected at `http://localhost:3000/oca-data-verifier`).
  - If your Composer runs elsewhere, update the `validatorUrl` prop in `ValidatorCard.vue` or pass a different value when using the component.

### Install & Run

1. Clone the repository and enter the project folder:

```bash
git clone <your-repo-url>
cd vue-iframe-demo
```

2. Install dependencies:

```bash
npm install
```

3. Start the development server:

```bash
npm run dev
```

4. Ensure OCA Composer is running and accessible at the URL configured in `ValidatorCard.vue` (default: `http://localhost:3000/oca-data-verifier`).

---

## Quick Start: Run OCA Composer Locally (required)

You need a local (or hosted) OCA Composer instance for the iframe to connect to. To run it locally:
`https://github.com/agrifooddatacanada/OCA_Composer/tree/feat/iframe-file-listener`

1. Clone or download the OCA Composer project from GitHub and open it in your terminal.
2. Create a `.env` file in the Composer project root and add:

```
REACT_APP_GA_ID=0
```

3. Install dependencies:

```
npm install
```

4. In the Composer project directory, start the development server:

```bash
npm start
```

This runs the Composer in development mode at `http://localhost:3000`. The page reloads on edits, and you will see any errors/warnings in the console.

Once Composer is running at `http://localhost:3000`, the front-end app’s iframe will load the Composer route defined by `validatorUrl` (default: `http://localhost:3000/oca-data-verifier`). If your Composer runs on a different host/port or path, either change `validatorUrl` in `src/components/ValidatorCard.vue` or pass it as a prop when using the component.

---

## Configuration

Schemas are configured in `src/assets/config/schemas.json`:

```json
{
  "schemas": [
    { "name": "Carcass_Schema.json", "path": "src/assets/files/CSV_OCA_package.json" },
    { "name": "Soil_Schema.json", "path": "src/assets/files/soil_package_1.json" }
  ]
}
```

Each `path` should point to a local JSON file that will be fetched and sent to the Composer.

---

## How the Iframe Integration Works

The integration is implemented in `src/components/ValidatorCard.vue`.

### Sending the schema to OCA Composer

When the iframe loads, the component posts the selected schema to the Composer:

```javascript
iframeRef.value.contentWindow.postMessage(
  {
    type: 'JSON_SCHEMA',
    data: JSON.parse(JSON.stringify(fileContent.value)),
  },
  '*', // For production, prefer the exact origin of your Composer instance
)
```

You can improve security by replacing `'*'` with the Composer origin (for example, `'http://localhost:3000'`).

### Receiving validated data from OCA Composer

The component listens for messages from the iframe and checks the origin:

```javascript
function receiveData(event) {
  if (event.origin !== 'http://localhost:3000') {
    return
  }
  if (event.data.type === 'VERIFIED_DATA') {
    const csvData = event.data.data
    emit('send-file', csvData)
    emit('close')
  }
}
```

The received CSV string is then rendered by `DataTable.vue`.

---

## Usage Flow

1. Select a schema from the dropdown in the app.
2. Click "Submit" to open the OCA Composer inside the iframe.
3. Complete the validation inside the Composer.
4. The Composer sends back a CSV string, which is displayed as a table in the app.

---

## Security Notes

- **Origin checks**: Always validate `event.origin` to ensure messages are from your trusted Composer host.
- **Target origin**: Prefer passing the exact Composer origin as the second argument to `postMessage` instead of `'*'`.

---

## Branding and White Labeling

OCA Composer supports white labeling so you can apply your own branding (logos, colors, titles, favicons, and other UI elements). For setup steps and the full list of options, please refer to the Composer project's README in the OCA Composer repository. Once configured and served, your customized Composer will load in the iframe without additional changes to this front-end app.

---

## Troubleshooting

- **Blank iframe or network error**: Verify `validatorUrl` points to a running OCA Composer and that the URL is reachable from your browser.
- **No data received**: Confirm the Composer posts messages with `type: 'VERIFIED_DATA'` and that `event.origin` matches your Composer origin.
- **Different ports/origins**: If running on different hosts, update both the `validatorUrl` and the origin checks in `ValidatorCard.vue` accordingly.

---

## License

This project is licensed under the MIT License.
