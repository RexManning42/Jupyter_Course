# Module 2: Intermediate - Data Collection, Analysis, and Visualization

Welcome to Module 2! Having established the foundations of OSINT and Jupyter Notebook in the beginner module, we now move into more powerful techniques. This intermediate module focuses on automating data collection, performing sophisticated data manipulation and analysis, and creating insightful visualizations – core skills for any proficient OSINT practitioner using Jupyter.

## 2.1 Automated Data Collection

Manual data collection is often tedious, time-consuming, and prone to errors. Automation allows us to gather larger volumes of data more efficiently and consistently. This section explores key Python libraries and techniques for automating the collection of information from websites and APIs.

### Web Scraping Techniques

Web scraping is the process of automatically extracting data from websites. While some sites offer APIs, many valuable sources only present information within their HTML structure. Scraping allows us to pull this data programmatically.

**Understanding the Building Blocks: HTML, CSS Selectors, XPath**

As reviewed briefly in Module 1, HTML provides the structure of a web page. To target specific data within that structure, we use selectors:

*   **CSS Selectors:** Patterns used to select HTML elements based on their tag name, ID, class, attributes, etc. They are widely used in web development for styling and are intuitive for selecting elements.
    *   Examples:
        *   `p`: Selects all `<p>` elements.
        *   `#main-content`: Selects the element with `id="main-content"`.
        *   `.article-title`: Selects all elements with `class="article-title"`.
        *   `a[href*="example.com"]`: Selects `<a>` elements whose `href` attribute contains "example.com".
        *   `div > p`: Selects `<p>` elements that are direct children of a `<div>`.
        *   `h2 + p`: Selects the first `<p>` element immediately following an `<h2>`.
*   **XPath (XML Path Language):** A query language for selecting nodes from an XML or HTML document. XPath is often more powerful and flexible than CSS selectors, especially for navigating complex document structures.
    *   Examples:
        *   `//p`: Selects all `<p>` elements anywhere in the document.
        *   `//div[@id=\'main-content\']`: Selects the `<div>` element with `id="main-content"`.
        *   `//h2[@class=\'title\']/text()`: Selects the text content of `<h2>` elements with `class="title"`.
        *   `//a[contains(@href, \'example.com\')]`: Selects `<a>` elements whose `href` attribute contains "example.com".
        *   `//table/tbody/tr[3]/td[2]`: Selects the second `<td>` in the third `<tr>` of a `<tbody>` within a `<table>`.

Understanding how to use these selectors is crucial for telling your scraping tools exactly what data to extract.

**Robust HTTP Requests with `requests`**

We introduced `requests` in Module 1. For scraping, we often need more control:

```python
import requests

# Using sessions for persistent parameters (like cookies)
session = requests.Session()

# Custom headers to mimic a browser
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
    "Accept-Language": "en-US,en;q=0.9",
    "Referer": "https://www.google.com/"
}

# Update session headers
session.headers.update(headers)

url = "https://httpbin.org/headers" # A site to test headers

try:
    # Send request using the session
    response = session.get(url, timeout=10)
    response.raise_for_status() # Raise an exception for bad status codes (4xx or 5xx)
    
    print("Request successful!")
    print("Response:")
    print(response.json()) # httpbin.org/headers returns JSON
    
except requests.exceptions.RequestException as e:
    print(f"An error occurred: {e}")
```

**Parsing HTML/XML with `BeautifulSoup`**

`BeautifulSoup` is the go-to library for parsing HTML and XML documents. It creates a parse tree from the page source code that can be used to extract data in a Pythonic way.

First, install it and a parser (like `lxml` or `html.parser`):

```python
# !pip install beautifulsoup4 lxml
```

Example usage:

```python
import requests
from bs4 import BeautifulSoup

url = "https://quotes.toscrape.com/" # A website designed for scraping practice

try:
    response = requests.get(url)
    response.raise_for_status()
    
    # Create a BeautifulSoup object
    # Use 'lxml' for speed, or 'html.parser' (built-in)
    soup = BeautifulSoup(response.text, "lxml")
    
    # Find all quote elements (they have class="quote")
    quotes = soup.find_all("div", class_="quote")
    
    print(f"Found {len(quotes)} quotes on the page.\n")
    
    extracted_data = []
    for quote_div in quotes:
        # Extract text using .text or .get_text()
        text = quote_div.find("span", class_="text").get_text(strip=True)
        
        # Extract author
        author = quote_div.find("small", class_="author").get_text(strip=True)
        
        # Extract tags (nested elements)
        tags = [tag.get_text(strip=True) for tag in quote_div.find_all("a", class_="tag")]
        
        extracted_data.append({
            "text": text,
            "author": author,
            "tags": tags
        })
        
        print(f"Quote: {text}")
        print(f"Author: {author}")
        print(f"Tags: {tags}\n")
        
    # You can now save extracted_data to a file (CSV, JSON)
    
except requests.exceptions.RequestException as e:
    print(f"Error fetching page: {e}")
except Exception as e:
    print(f"Error parsing page: {e}")
```

**Browser Automation with `Selenium`**

Some websites heavily rely on JavaScript to load content dynamically. `requests` and `BeautifulSoup` only fetch the initial HTML source and don't execute JavaScript. For these sites, we need a tool that can control a real web browser – this is where `Selenium` comes in.

`Selenium` automates browser actions like clicking buttons, filling forms, scrolling, and waiting for elements to appear. It requires a browser driver (e.g., `chromedriver` for Chrome, `geckodriver` for Firefox) to be installed and accessible in your system's PATH.

Installation:

```python
# !pip install selenium
# Download appropriate WebDriver (chromedriver/geckodriver) and place in PATH
```

Example usage:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException, NoSuchElementException
import time

# --- Setup WebDriver ---
# Ensure chromedriver is in your PATH or specify the path
# options = webdriver.ChromeOptions()
# options.add_argument('--headless') # Run Chrome in headless mode (no GUI)
# options.add_argument('--disable-gpu')
# driver = webdriver.Chrome(options=options)

# Using Firefox (ensure geckodriver is in PATH)
options = webdriver.FirefoxOptions()
options.add_argument('--headless')
driver = webdriver.Firefox(options=options)

print("WebDriver initialized.")

url = "https://quotes.toscrape.com/js/" # A JavaScript-rendered version of the quotes site

scraped_quotes = []

try:
    driver.get(url)
    print(f"Navigated to {url}")
    
    # Wait for the main content area to be present
    WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.CLASS_NAME, "quote"))
    )
    print("Quotes container found.")

    while True:
        # Find all quote elements currently visible
        quote_elements = driver.find_elements(By.CLASS_NAME, "quote")
        print(f"Found {len(quote_elements)} quotes on this page.")
        
        for quote_element in quote_elements:
            try:
                text = quote_element.find_element(By.CLASS_NAME, "text").text
                author = quote_element.find_element(By.CLASS_NAME, "author").text
                tags = [tag.text for tag in quote_element.find_elements(By.CLASS_NAME, "tag")]
                scraped_quotes.append({"text": text, "author": author, "tags": tags})
            except NoSuchElementException:
                print("Could not parse a quote element fully.")
                continue
        
        # Check if there is a "Next" button
        try:
            next_button = driver.find_element(By.CSS_SELECTOR, "li.next > a")
            print("Found 'Next' button. Clicking...")
            next_button.click()
            # Wait for the next page's quotes to load (implicitly handled by loop start)
            time.sleep(2) # Add a small delay for stability
        except NoSuchElementException:
            print("No 'Next' button found. End of pages.")
            break # Exit the loop if no next button
        except Exception as click_err:
            print(f"Error clicking next button: {click_err}")
            break

except TimeoutException:
    print("Timed out waiting for page elements to load.")
except Exception as e:
    print(f"An error occurred during scraping: {e}")
finally:
    # Always close the browser window
    driver.quit()
    print("WebDriver closed.")

# Display scraped quotes
print(f"\nTotal quotes scraped: {len(scraped_quotes)}")
# for i, quote in enumerate(scraped_quotes[:5]): # Print first 5
#     print(f"\nQuote {i+1}:")
#     print(f"  Text: {quote['text']}")
#     print(f"  Author: {quote['author']}")
#     print(f"  Tags: {quote['tags']}")
```

**Ethical Scraping:** Always check a website's `robots.txt` file (e.g., `https://example.com/robots.txt`) for scraping rules. Respect rate limits, identify your scraper in the User-Agent if possible, and avoid overwhelming the target server. Never scrape data that requires login credentials you aren't authorized to use or data protected by copyright or privacy laws without permission.

### Working with APIs

APIs (Application Programming Interfaces) are often the preferred way to collect data programmatically. They provide structured data (usually JSON) in a reliable format, designed for machine consumption.

**Understanding RESTful APIs**

Many web APIs follow REST (Representational State Transfer) principles. Key concepts include:

*   **Resources:** Data entities accessible via URLs (e.g., `/users`, `/posts/123`).
*   **HTTP Methods:** Standard verbs defining actions on resources:
    *   `GET`: Retrieve data.
    *   `POST`: Create new data.
    *   `PUT`: Update existing data (replace).
    *   `PATCH`: Partially update existing data.
    *   `DELETE`: Remove data.
*   **Status Codes:** Indicate the outcome of a request (e.g., `200 OK`, `201 Created`, `400 Bad Request`, `401 Unauthorized`, `404 Not Found`, `500 Internal Server Error`).
*   **Request/Response Structure:** Data is typically exchanged in JSON format in the request body (for POST/PUT/PATCH) or response body.

**Authentication Methods**

Most useful APIs require authentication to identify the user and control access:

*   **API Keys:** A unique string passed in request headers or URL parameters (`?apiKey=YOUR_KEY`). Simple but less secure if exposed.
*   **OAuth (Open Authorization):** A more complex but secure standard involving tokens. Users grant limited access to an application without sharing their credentials. Requires multiple steps (authorization request, token exchange).
*   **Bearer Tokens:** A type of token (often obtained via OAuth or another login process) included in the `Authorization` header: `Authorization: Bearer YOUR_TOKEN`.

**Using `requests` for API Interaction**

`requests` makes API interaction straightforward.

```python
import requests
import json

# Example: Using the GitHub API (public endpoint, no auth needed for basic info)
username = "octocat"
api_url = f"https://api.github.com/users/{username}"

headers = {
    "Accept": "application/vnd.github.v3+json" # Specify API version
}

try:
    response = requests.get(api_url, headers=headers, timeout=10)
    response.raise_for_status()
    
    user_data = response.json() # Parse JSON response
    
    print(f"Successfully retrieved data for user: {user_data.get('login')}")
    print(f"Name: {user_data.get('name')}")
    print(f"Public Repos: {user_data.get('public_repos')}")
    print(f"Followers: {user_data.get('followers')}")
    print(f"Profile URL: {user_data.get('html_url')}")
    
    # Save the full response
    # with open(f"{username}_github_data.json", "w") as f:
    #     json.dump(user_data, f, indent=4)

except requests.exceptions.RequestException as e:
    print(f"Error accessing GitHub API: {e}")
except json.JSONDecodeError:
    print("Error decoding JSON response.")
```

**Example with API Key (Conceptual)**

```python
# Conceptual example - replace with a real API and key
# api_key = "YOUR_SECRET_API_KEY" 
# api_endpoint = "https://api.exampleosintservice.com/v1/lookup"
# 
# headers = {
#     "Authorization": f"Bearer {api_key}" 
#     # Or sometimes: "X-Api-Key": api_key
# }
# 
# params = {
#     "query": "target_indicator",
#     "type": "domain"
# }
# 
# try:
#     response = requests.get(api_endpoint, headers=headers, params=params)
#     response.raise_for_status()
#     data = response.json()
#     print(json.dumps(data, indent=2))
# except Exception as e:
#     print(f"API Error: {e}")
```

### Social Media Data Collection

Collecting data from social media platforms presents unique challenges:

*   **API Restrictions:** Most platforms heavily restrict what data can be accessed via their official APIs, often requiring developer accounts, approval processes, and adhering to strict rate limits and usage policies.
*   **Terms of Service:** Scraping social media sites often violates their Terms of Service, potentially leading to account suspension or legal action.
*   **Privacy:** Accessing private profiles or circumventing privacy settings is unethical and illegal.
*   **Dynamic Content:** Many platforms use JavaScript extensively, making simple scraping difficult.

**Using Official APIs (Example: Twitter API v2 with `tweepy`)**

If you have legitimate access (e.g., academic research, approved application), using the official API is the most reliable and ethical method.

Installation:

```python
# !pip install tweepy
```

Example (Conceptual - Requires Developer Account & Bearer Token):

```python
import tweepy
import json

# Replace with your actual Bearer Token
# bearer_token = "YOUR_TWITTER_API_V2_BEARER_TOKEN"

# client = tweepy.Client(bearer_token)

# query = '"open source intelligence" lang:en -is:retweet'

# try:
#     # Search recent tweets
#     response = client.search_recent_tweets(
#         query,
#         tweet_fields=["created_at", "public_metrics", "author_id"],
#         max_results=10 # Get up to 10 results
#     )

#     if response.data:
#         print(f"Found {len(response.data)} tweets matching the query.\n")
#         for tweet in response.data:
#             print(f"ID: {tweet.id}")
#             print(f"Author ID: {tweet.author_id}")
#             print(f"Created At: {tweet.created_at}")
#             print(f"Text: {tweet.text}")
#             print(f"Metrics: {tweet.public_metrics}")
#             print("---")
#             
#             # You can save tweet data to a file here
#             # with open("tweets.jsonl", "a") as f:
#             #     f.write(json.dumps(tweet.data) + "\n")
#     else:
#         print("No tweets found matching the query.")

# except tweepy.errors.TweepyException as e:
#     print(f"Twitter API Error: {e}")
# except Exception as e:
#     print(f"An unexpected error occurred: {e}")

print("\nNote: This is a conceptual example. Run with a valid Bearer Token.") 
```

**Ethical Scraping Considerations & Alternatives**

If official APIs are insufficient or unavailable, scraping might be considered, but *only* with extreme caution:

*   **Focus on Public Data:** Never attempt to access private information.
*   **Respect `robots.txt` and ToS:** Understand the risks if you choose to violate them.
*   **Rate Limiting:** Scrape slowly and mimic human behavior to avoid detection and bans.
*   **Use `Selenium`:** Often necessary for JavaScript-heavy sites.
*   **Consider Alternatives:** Are there public archives, datasets, or third-party services that provide the data legally?
*   **Legal Counsel:** If unsure about the legality, consult legal professionals.

### Assignment 2: Automated Data Gathering

**Objective:** Practice automated data collection using web scraping and API interaction.

**Instructions:**
1.  **Part 1: Web Scraping**
    *   Choose a website with structured data suitable for scraping (e.g., a public directory, e-commerce category page, news archive - *ensure you check `robots.txt` and scrape responsibly*). Avoid sites requiring login unless using dummy accounts on practice sites.
    *   Use `requests` and `BeautifulSoup` (or `Selenium` if necessary) to scrape relevant data points (e.g., names, prices, dates, headlines, links).
    *   Extract data from at least two pages (e.g., by following pagination links).
    *   Store the scraped data in a structured format (list of dictionaries is recommended).
2.  **Part 2: API Interaction**
    *   Find a public API that provides data related to your scraped topic or another area of interest (e.g., a news API, weather API, government data portal API). Many require a free API key.
    *   Use the `requests` library to interact with the API.
    *   Make at least one API call, retrieve the data, and parse the JSON response.
    *   Extract relevant information from the API response.
3.  **Documentation & Saving:**
    *   Document your entire process in a Jupyter Notebook, explaining your choices and code.
    *   Save the scraped data and the API data into separate files (e.g., CSV or JSON).

**Deliverables:**
- A Jupyter Notebook (.ipynb file) containing your code, explanations, and outputs.
- The data files (CSV/JSON) generated from scraping and the API.

**Evaluation Criteria:**
- Successful scraping of data from the chosen website.
- Successful interaction with the chosen API.
- Correct parsing and structuring of collected data.
- Code quality, clarity, and comments.
- Adherence to ethical scraping principles.
- Clear documentation within the notebook.
