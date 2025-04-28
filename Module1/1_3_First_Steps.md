## 1.3 First Steps in OSINT with Jupyter

Now that we've covered the fundamentals of both OSINT and Jupyter Notebook, it's time to combine these skills and take our first practical steps in conducting OSINT investigations using Jupyter. This section will introduce you to basic techniques for collecting, organizing, and documenting information programmatically, setting the foundation for more advanced methods in later modules.

### Programmatic Web Searching

While traditional web searches using browsers are valuable, programmatic approaches offer several advantages for OSINT: they can be automated, repeated consistently, documented precisely, and their results can be directly processed by other code. Let's explore how to perform basic web requests using Python's `requests` library.

The `requests` library is a powerful yet user-friendly HTTP library that allows you to send HTTP/1.1 requests without manually adding query strings to URLs, form-encoding POST data, or dealing with complex authentication mechanisms.

First, ensure the library is installed:

```python
# Run this if requests is not already installed
# !pip install requests
```

Now, let's perform a basic GET request to retrieve a web page:

```python
import requests

# Define the URL to request
url = "https://example.com"

try:
    # Send a GET request
    response = requests.get(url)
    
    # Check if the request was successful (status code 200)
    if response.status_code == 200:
        print(f"Request successful! Status code: {response.status_code}")
        
        # Print the first 500 characters of the response content
        print("Preview of content:")
        print(response.text[:500] + "...")
        
        # Get response headers
        print("\nResponse headers:")
        for header, value in response.headers.items():
            print(f"{header}: {value}")
    else:
        print(f"Request failed with status code: {response.status_code}")
        
except requests.exceptions.RequestException as e:
    print(f"An error occurred: {e}")
```

For more realistic OSINT scenarios, you might want to customize your requests to appear more like a regular browser (some websites block or limit requests that don't look like they come from a browser):

```python
import requests
import time
import random

# Define a user agent that mimics a browser
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"
}

# List of URLs to request
urls = [
    "https://example.com",
    "https://example.org",
    "https://example.net"
]

results = []

for url in urls:
    try:
        # Add a random delay between requests (good practice to avoid being blocked)
        time.sleep(random.uniform(1, 3))
        
        # Send a GET request with custom headers
        response = requests.get(url, headers=headers, timeout=10)
        
        # Store the results
        results.append({
            "url": url,
            "status_code": response.status_code,
            "content_length": len(response.text),
            "title": response.text.split("<title>")[1].split("</title>")[0] if "<title>" in response.text else "No title found"
        })
        
        print(f"Successfully retrieved {url}")
        
    except Exception as e:
        print(f"Error retrieving {url}: {e}")
        results.append({
            "url": url,
            "status_code": "Error",
            "content_length": 0,
            "title": str(e)
        })

# Display the results in a table format
print("\nResults Summary:")
print("-" * 80)
print(f"{'URL':<30} | {'Status':<10} | {'Length':<10} | {'Title':<30}")
print("-" * 80)
for result in results:
    print(f"{result['url']:<30} | {result['status_code']:<10} | {result['content_length']:<10} | {result['title'][:30]}")
```

### Basic HTML Structure Review

To effectively extract information from web pages, it's helpful to understand the basic structure of HTML (HyperText Markup Language), the standard markup language for documents designed to be displayed in a web browser.

HTML documents consist of elements, which are represented by tags. Tags are enclosed in angle brackets, like `<tagname>`. Most tags come in pairs with an opening tag and a closing tag (with a forward slash), such as `<p>` and `</p>`, with content in between.

Key HTML elements you'll encounter:

- `<!DOCTYPE html>`: Declares the document type
- `<html>`: The root element of an HTML page
- `<head>`: Contains meta-information about the document
- `<title>`: Specifies the title of the document (shown in browser tabs)
- `<body>`: Contains the visible page content
- `<h1>` to `<h6>`: Heading elements (h1 is the most important, h6 the least)
- `<p>`: Paragraph element
- `<a>`: Anchor element (creates hyperlinks)
- `<img>`: Image element
- `<div>`: Division element (a generic container)
- `<span>`: Inline container
- `<table>`, `<tr>`, `<th>`, `<td>`: Table elements
- `<ul>`, `<ol>`, `<li>`: List elements (unordered list, ordered list, list item)
- `<form>`, `<input>`, `<button>`: Form elements

Elements can have attributes that provide additional information. Attributes are specified in the opening tag, like `<a href="https://example.com">`.

Common attributes include:
- `id`: Unique identifier for an element
- `class`: Specifies one or more class names for styling
- `href`: Specifies the URL for links
- `src`: Specifies the source for images, scripts, etc.
- `alt`: Alternative text for images
- `style`: Inline CSS styling

Understanding this structure will help you navigate and extract data from HTML content in your OSINT investigations.

### Saving and Organizing Findings

Effective documentation is crucial in OSINT. Jupyter Notebooks excel at this by allowing you to combine code, outputs, and explanations. Here's how to organize your findings:

```python
# Example of documenting and saving OSINT findings in a structured way

import requests
import json
import csv
import os
from datetime import datetime

# Create a directory for our investigation if it doesn't exist
investigation_dir = "investigation_example_company"
if not os.path.exists(investigation_dir):
    os.makedirs(investigation_dir)
    print(f"Created directory: {investigation_dir}")

# Record investigation metadata
investigation_meta = {
    "subject": "Example Company",
    "investigator": "Your Name",
    "date_started": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
    "objective": "Gather basic public information about the company"
}

# Save metadata to a JSON file
with open(f"{investigation_dir}/metadata.json", "w") as f:
    json.dump(investigation_meta, f, indent=4)
    print(f"Saved investigation metadata to {investigation_dir}/metadata.json")

# Function to save a web page for later reference
def save_webpage(url, headers=None):
    try:
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            # Create a filename based on the URL
            filename = url.replace("https://", "").replace("http://", "").replace("/", "_")
            if not filename.endswith(".html"):
                filename += ".html"
            
            # Save the HTML content
            with open(f"{investigation_dir}/{filename}", "w", encoding="utf-8") as f:
                f.write(response.text)
            
            print(f"Saved {url} to {investigation_dir}/{filename}")
            return True
        else:
            print(f"Failed to retrieve {url}, status code: {response.status_code}")
            return False
    except Exception as e:
        print(f"Error saving {url}: {e}")
        return False

# Example usage
save_webpage("https://example.com")

# Create a CSV file to track findings
findings_file = f"{investigation_dir}/findings.csv"
with open(findings_file, "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerow(["Date", "Source", "Type", "Description"])
    writer.writerow([
        datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        "https://example.com",
        "Website",
        "Company's main website"
    ])
    print(f"Created findings log at {findings_file}")

# Function to add a new finding
def add_finding(source, type, description):
    with open(findings_file, "a", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow([
            datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            source,
            type,
            description
        ])
    print(f"Added new finding from {source}")

# Example usage
add_finding("Manual research", "Note", "Company appears to have offices in multiple countries")
```

### Interacting with Online OSINT Tools

Many specialized OSINT tools are available online. While we'll develop our own tools throughout this course, it's valuable to know how to interact with existing ones. Here's an example of how to programmatically use a public WHOIS service:

```python
import requests
import json

def whois_lookup(domain):
    """
    Perform a WHOIS lookup using a public API
    """
    url = f"https://api.whoisfreaks.com/v1.0/whois?apiKey=YOUR_API_KEY&whois=live&domainName={domain}"
    
    # Note: In a real scenario, you would need to sign up for an API key
    # For this example, we'll simulate a response
    
    # Simulated response for educational purposes
    simulated_response = {
        "domain_name": domain,
        "domain_registrar": "Example Registrar, LLC",
        "domain_registered_on": "2000-01-01T00:00:00Z",
        "domain_expires_on": "2030-01-01T00:00:00Z",
        "domain_status": "clientTransferProhibited",
        "nameservers": [
            "ns1.example-registrar.com",
            "ns2.example-registrar.com"
        ],
        "domain_dnssec": "unsigned"
    }
    
    print(f"WHOIS information for {domain}:")
    print(json.dumps(simulated_response, indent=4))
    return simulated_response

# Example usage
whois_result = whois_lookup("example.com")

# Save the result
with open(f"{investigation_dir}/whois_example.com.json", "w") as f:
    json.dump(whois_result, f, indent=4)
    print(f"Saved WHOIS data to {investigation_dir}/whois_example.com.json")
```

### Assignment 1: Profile a Public Company

**Objective:** Create a Jupyter Notebook that profiles a public company using search engines and basic web requests.

**Instructions:**
1. Choose a publicly traded company of your interest.
2. Create a new Jupyter Notebook with appropriate markdown sections for organization.
3. Use the `requests` library to retrieve the company's main website.
4. Extract and document key information such as:
   - Company name, logo, and slogan
   - Main products or services
   - Leadership team
   - Office locations
   - Recent news mentions
5. Document your methodology, including search terms used and websites visited.
6. Save relevant URLs and text snippets in structured formats (CSV, JSON).
7. Include a summary of your findings in markdown.

**Deliverables:**
- A Jupyter Notebook (.ipynb file) with your code, documentation, and findings
- A CSV file containing structured data about the company
- A folder with saved HTML pages or other relevant files

**Evaluation Criteria:**
- Completeness of information gathered
- Code functionality and organization
- Quality of documentation
- Proper use of Jupyter Notebook features (markdown, code cells)
- Adherence to ethical guidelines (only using publicly available information)

This assignment will help you apply the basic OSINT and Jupyter skills covered in this module while developing good habits for investigation documentation.
