# Google-Chrome-Extension-Data-Scraper-Tool
Creating a Chrome extension that mimics the functionality of "Instant Data Scraper" involves scraping data from web pages based on user-defined parameters (like CSS selectors) and then displaying or exporting the scraped data (e.g., as CSV or JSON). Below is a step-by-step guide on how to create such a Chrome extension.
Basic Overview:

    Manifest file: To define extension settings.
    Popup interface: Allows the user to interact with the extension.
    Content script: Scrapes data from the page based on the selectors.
    Background script: Handles events and manages data.
    Exporting the data: Provide a way for users to download the scraped data.

1. Manifest File (manifest.json)

This file defines the settings of the Chrome extension, including the permissions it needs.

{
  "manifest_version": 3,
  "name": "Instant Data Scraper",
  "description": "Scrape and export data from any webpage.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage",
    "downloads"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ],
  "host_permissions": [
    "http://*/*",
    "https://*/*"
  ],
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}

2. Popup Interface (popup.html)

The popup will display the interface for interacting with the extension (e.g., entering a selector, scraping data, etc.).

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Instant Data Scraper</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      width: 300px;
      padding: 10px;
    }
    input, button {
      width: 100%;
      margin-bottom: 10px;
      padding: 10px;
    }
  </style>
</head>
<body>
  <h2>Data Scraper</h2>
  <label for="selector">Enter CSS Selector:</label>
  <input type="text" id="selector" placeholder="e.g., .product-name">
  <button id="scrapeButton">Scrape Data</button>
  <button id="downloadButton" disabled>Download Data</button>

  <script src="popup.js"></script>
</body>
</html>

3. Content Script (content.js)

The content script will run on the current webpage and extract data based on the CSS selector provided by the user.

// Listens for the message from popup to start scraping
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'scrapeData') {
    let data = [];
    const selector = message.selector;
    
    // Try to capture the elements using the provided selector
    const elements = document.querySelectorAll(selector);
    elements.forEach(element => {
      data.push(element.innerText.trim());
    });

    // Send back the scraped data to popup
    sendResponse({ data: data });
  }
});

4. Background Script (background.js)

This script manages background tasks, such as enabling communication between different parts of the extension.

chrome.runtime.onInstalled.addListener(() => {
  console.log("Instant Data Scraper Extension Installed");
});

5. Popup Script (popup.js)

The popup script handles the interaction with the user, including capturing the CSS selector, sending the request to scrape the data, and handling the export of the data.

document.getElementById('scrapeButton').addEventListener('click', () => {
  const selector = document.getElementById('selector').value;

  if (selector) {
    // Send message to content script to start scraping
    chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
      chrome.tabs.sendMessage(tabs[0].id, { action: 'scrapeData', selector: selector }, (response) => {
        if (response && response.data) {
          // Enable download button and store the data
          chrome.storage.local.set({ scrapedData: response.data });
          document.getElementById('downloadButton').disabled = false;
        }
      });
    });
  } else {
    alert('Please enter a CSS selector');
  }
});

document.getElementById('downloadButton').addEventListener('click', () => {
  // Retrieve the scraped data
  chrome.storage.local.get('scrapedData', (data) => {
    if (data.scrapedData && data.scrapedData.length > 0) {
      // Create CSV file
      let csvContent = "data:text/csv;charset=utf-8," + data.scrapedData.join("\n");

      // Encode URI and initiate download
      const encodedUri = encodeURI(csvContent);
      const link = document.createElement("a");
      link.setAttribute("href", encodedUri);
      link.setAttribute("download", "scraped_data.csv");
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    } else {
      alert('No data to download');
    }
  });
});

6. Icons (optional)

You can include some basic icons for the extension in the icons folder, e.g., icon16.png, icon48.png, and icon128.png.
How it Works:

    Popup: Users enter a CSS selector (like .product-name or .price), which is then sent to the content script.
    Content Script: The script extracts data from the current webpage that matches the CSS selector.
    Background Script: Handles the installation and extension setup.
    Download: The scraped data is stored locally in chrome.storage, and the user can download it in CSV format.

7. Testing the Extension

    Go to chrome://extensions/ in your browser.
    Enable "Developer mode" in the top right corner.
    Click "Load unpacked" and select the folder containing your extension.
    Navigate to a page, click on the extension, enter a selector (e.g., .product-name), and click "Scrape Data".
    Once the data is scraped, click "Download Data" to export it as a CSV file.

Conclusion:

This is a basic Chrome extension that can scrape data from any webpage based on a user-defined CSS selector. It stores the scraped data locally and allows the user to export it as a CSV file. You can extend the functionality further by adding features like pagination handling, advanced data processing, or exporting data in different formats (e.g., JSON).
