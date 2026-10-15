---
name: webapp-testing
description: Playwright-based execution and verification toolkit for local web applications, DOM inspection, screenshot capture, and browser log extraction.
license: Complete terms in LICENSE.txt
---

# Web Application Testing

## Execution Routing

- Parse each target HTML file directly via the `file://` protocol to extract DOM selectors when testing static markup.
- Transition to the dynamic web application workflow when static markup parsing yields incomplete selector definitions.
- Invoke `scripts/with_server.py` to manage process lifecycles when targeting an uninitialized dynamic web server.
- Execute the reconnaissance-and-action sequence directly against the active endpoint when the target web server is running.

## Server Lifecycle Management

- Execute `python scripts/with_server.py --help` prior to script invocation to inspect command-line interface parameters, isolated from reading the helper script source code.
- Invoke `scripts/with_server.py` as an external process wrapper for each single-server or multi-server topology:
  - Single-server execution:
    ```bash
    python scripts/with_server.py --server "npm run dev" --port 5173 -- python your_automation.py
    ```
  - Multi-server execution:
    ```bash
    python scripts/with_server.py \
      --server "cd backend && python server.py" --port 3000 \
      --server "cd frontend && npm run dev" --port 5173 \
      -- python your_automation.py
    ```

## Browser Automation & Reconnaissance

- Initialize each synchronous automation session via `sync_playwright()` and launch Chromium with `headless=True`, isolated from server startup logic.
- Navigate to each target endpoint and synchronize execution with `page.wait_for_load_state('networkidle')` prior to DOM inspection:
  ```python
  from playwright.sync_api import sync_playwright

  with sync_playwright() as p:
      browser = p.chromium.launch(headless=True)
      page = browser.new_page()
      page.goto('http://localhost:5173')
      page.wait_for_load_state('networkidle')
      page.screenshot(path='/tmp/inspect.png', full_page=True)
      content = page.content()
      buttons = page.locator('button').all()
      browser.close()
