**Note**: This application is currently under active development. Features and functionality may change as development progresses.

# React Verifier

A React application for verifying data against predefined schemas using an embedded React-based verifier.

# Iframe Communication Demo

## Overview

This project demonstrates how to use the `postMessage` API to communicate between a front-end application and an iframe. It allows you to send and receive data securely across different origins, making it suitable for various front-end frameworks and libraries.

## Features

- **Dynamic File Selection**: Select JSON files from a dropdown to load them into the application.
- **Modal Display**: Open a modal to display the selected file content in an iframe.
- **Data Reception**: Receive data from the iframe using the `postMessage` API.

## Getting Started

### Prerequisites

- Node.js and npm installed on your machine.
- A basic understanding of HTML, CSS, and JavaScript.

### Installation

1. Clone the repository:

   ```bash
   git clone https://github.com/agrifooddatacanada/react_verifier.git
   cd react_verifier
   ```

2. Install dependencies:

   ```bash
   npm install
   ```

3. Start the development server:
   ```bash
   npm run dev
   ```

## Code Explanation

### Sending Data to the Iframe

To send data to the iframe, use the `postMessage` method on the iframe's `contentWindow`. Here's an example:

```javascript
function sendDataToIframe(data) {
  const iframe = document.getElementById('myIframe') // Replace with your iframe's ID
  iframe.contentWindow.postMessage(
    {
      type: 'LOAD_FILE',
      data: data,
    },
    '*', // Specify the target origin for security
  )
}
```

### Receiving Data from the Iframe

To receive data from the iframe, set up an event listener in your JavaScript code. This is typically done in the `window` object:

```javascript
window.addEventListener('message', receiveData)

function receiveData(event) {
  // Check the origin of the message for security
  if (event.origin !== 'https://www.semanticengine.org/') {
    return // Ignore messages from unknown origins
  }

  // Check the type of the message
  if (event.data.type === 'VERIFIED_DATA') {
    const csvData = event.data.data
    // Handle the CSV data as needed
    console.log('Received CSV data:', csvData)
  }
}
```

### Security Considerations

- **Origin Check**: Always check the `event.origin` in the `receiveData` function to ensure that the message is coming from a trusted source. This prevents potential security vulnerabilities.
- **Target Origin**: When sending messages to the iframe, specify the target origin as the second argument in `postMessage` to restrict the message to a specific domain.

## Example

In the `Modal.js` component, the iframe is set up to receive data from the parent window. When a file is selected, the content is sent to the iframe using `postMessage`. The iframe listens for messages and processes the received data accordingly.

## Example Usage

### Handling CSV Data Received from the Iframe

When you receive a CSV string from the iframe, you can handle it in various ways. Below is an example of how to download the received CSV data as a CSV file:

```javascript
function receiveData(event) {
  // Check the origin of the message for security
  if (event.origin !== 'http://your-iframe-origin.com') {
    return // Ignore messages from unknown origins
  }

  // Check the type of the message
  if (event.data.type === 'CSV_STRING') {
    const csvData = event.data.data // Assuming this is the CSV string received

    // Create a Blob from the CSV string
    const blob = new Blob([csvData], { type: 'text/csv' })

    // Create a URL for the Blob
    const url = URL.createObjectURL(blob)

    // Create a temporary anchor element to trigger the download
    const a = document.createElement('a')
    a.href = url
    a.download = 'validatedData.csv' // Specify the filename for the download
    document.body.appendChild(a) // Append the anchor to the body
    a.click() // Trigger the download

    // Clean up: remove the anchor and revoke the URL
    document.body.removeChild(a)
    URL.revokeObjectURL(url)
  }
}
```

### Explanation

- **Blob Creation**: A `Blob` is created from the CSV string, specifying the MIME type as `text/csv`.
- **URL Creation**: A URL is generated for the Blob using `URL.createObjectURL(blob)`.
- **Download Trigger**: A temporary anchor (`<a>`) element is created, its `href` is set to the generated URL, and its `download` attribute is set to the desired filename. The anchor is then appended to the document body and clicked to trigger the download.
- **Cleanup**: After triggering the download, the anchor is removed from the document, and the URL is revoked to free up resources.

This example demonstrates how to effectively handle CSV data received from an iframe and provide a user-friendly way to download it. Adjust the filename and any additional processing as needed for your application.

## License

This project is licensed under the MIT License.
