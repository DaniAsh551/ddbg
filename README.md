# DDBG Client - Stack Trace Viewer

A web-based client application built with [NiceGUI](https://nicegui.io/) designed to receive and display stack traces and associated trace information pushed from a remote source. It's intended to be part of a larger distributed debugging or monitoring system.

The client listens for data on a specific endpoint, registers itself with a companion server component, and displays the received information in real-time via a web UI.

## Features

* **Web UI:** Simple and clean interface for viewing the latest received stack and trace data.
* **Real-time Updates:** Automatically polls for new data and refreshes the display without requiring manual page reloads.
* **HTTP Endpoint:** Receives data via a simple `POST` request to the `/push` endpoint.
* **Flexible Configuration:** Can be configured using environment variables (for automated setups) or through a dedicated `/bind` page in the web UI.
* **Server Binding:** Registers its own accessible URL with a companion server component via the server's `/attach` endpoint.

## Requirements

* Python 3.8+
* pip (Python package installer)
* A companion "DDBG Server" component (this client needs a server URL to bind to). This server component is *not* included in this codebase but is required for the binding process.

## Installation

1.  **Clone the repository or download the script:**
    ```bash
    # If using Git
    git clone <your-repo-url>
    cd <your-repo-directory>

    # Or just save the Python script (e.g., as ddbg_client.py)
    ```

2.  **Create and activate a virtual environment (Recommended):**
    ```bash
    python -m venv venv
    # On Windows
    .\venv\Scripts\activate
    # On macOS/Linux
    source venv/bin/activate
    ```

3.  **Create a `requirements.txt` file** with the following content:
    ```txt
    nicegui
    pydantic
    requests
    validators
    ```

4.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

## Configuration

The client needs two crucial URLs to function:

1.  **`LOCAL` URL:** The base URL where *this* client application is accessible from the network (specifically, accessible by the server it binds to). Example: `http://192.168.1.10:8080`
2.  **`SERVER` URL:** The base URL of the companion "DDBG Server" component, including any specific path required for binding. Example: `http://192.168.1.20:8000/ddbg`

You can configure these in two ways:

* **Method 1: Environment Variables (Recommended)**
    Set the `LOCAL` and `SERVER` environment variables before running the script. The application will automatically read these, validate them, and attempt to bind with the server on startup.
    ```bash
    # Example on Linux/macOS
    export LOCAL="http://<your_client_ip_or_hostname>:8080"
    export SERVER="http://<your_server_ip_or_hostname>:<port>/<path>"
    python ddbg

    # Example on Windows (Command Prompt)
    set LOCAL="http://<your_client_ip_or_hostname>:8080"
    set SERVER="http://<your_server_ip_or_hostname>:<port>/<path>"
    python ddbg

    # Example on Windows (PowerShell)
    $env:LOCAL="http://<your_client_ip_or_hostname>:8080"
    $env:SERVER="http://<your_server_ip_or_hostname>:<port>/<path>"
    python ddbg
    ```
    *(Note: Replace `<...>` placeholders with actual values. The client runs on port 8080 by default if not specified otherwise in code/environment for NiceGUI)*

* **Method 2: Web UI (`/bind` page)**
    If you run the script without setting the environment variables, accessing the application in your browser will automatically redirect you to the `/bind` page.
    1.  Enter "This App's Base URL" (your `LOCAL` URL).
    2.  Enter "Remote Server Base URL" (your `SERVER` URL).
    3.  Click "Save and Bind".
    4.  If the binding is successful (the client receives a `"true"` response from the server's `/attach` endpoint), you will be navigated to the main display page (`/`). A dialog will show the binding attempt result.

## Running the Application

1.  Ensure configuration is done via environment variables OR be prepared to use the `/bind` page.
2.  Navigate to the directory containing the script and your activated virtual environment.
3.  Run the script:
    ```bash
    python ddbg
    ```
4.  The application will start, and NiceGUI will print the URL where the UI is accessible (usually `http://<your-ip>:8080` or similar). Check the console output.

## Usage

1.  Open the URL provided in the console output in your web browser.
2.  If not configured via environment variables, you'll be directed to `/bind` first. Complete the configuration.
3.  Once configured and bound, the main page (`/`) will initially show "Waiting...".
4.  When an external process sends data to this client's `/push` endpoint, the display will update automatically to show the latest "Stack" and "Trace" information.

## API Endpoint - `/push`

This is the endpoint the external monitored process should send data to.

* **Method:** `POST`
* **Path:** `/push`
* **Request Body:** JSON object matching the following structure:
    ```json
    {
      "timestamp": 1678886400,        // Integer: Unix timestamp of when the data was generated
      "data": "Main stack trace...",  // String: The primary data (e.g., stack trace)
      "trace": "Additional trace..."  // String: Any supplementary trace info
    }
    ```
* **Success Response:** The `timestamp` value from the request payload (as an integer).
* **Error Response:** An error message string (e.g., `"Err: Malformed payload"`).

## Architecture Overview

1.  **Startup:** The client starts, checks for `LOCAL` and `SERVER` URLs (env vars first, then requires UI input via `/bind`).
2.  **Binding:** The client sends its `LOCAL` URL to the `SERVER`'s `/attach` endpoint to register itself.
3.  **Listening:** The client's `/push` endpoint listens for incoming `POST` requests containing stack/trace data.
4.  **Data Reception:** When data is received at `/push`, the client updates its internal state (global variables `stack`, `trace`, `stack_time`).
5.  **UI Polling:** The web UI periodically checks (via `poll_stack` function) if the `stack_time` has changed.
6.  **UI Refresh:** If `stack_time` indicates new data, the UI component (`show_stack`) is refreshed to display the latest `stack` and `trace`.