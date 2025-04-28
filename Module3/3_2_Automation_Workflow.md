## 3.2 Automation and Workflow Building

As your OSINT investigations grow in complexity, automation becomes essential for efficiency, consistency, and scalability. This section focuses on structuring your code for reusability, building end-to-end OSINT pipelines, creating interactive dashboards, and implementing robust error handling and scheduling mechanisms within Jupyter Notebook.

### Structuring OSINT Code

Well-structured code is easier to maintain, debug, and reuse across investigations. Let's explore how to organize your OSINT code using functions and classes.

**Writing Modular Functions**

Functions allow you to encapsulate specific tasks, making your code more readable and reusable.

```python
# Example of modular OSINT functions
import requests
import json
import pandas as pd
from bs4 import BeautifulSoup
import time
import random
import os

# Define headers to mimic a browser
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
    "Accept-Language": "en-US,en;q=0.9",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8"
}

def fetch_url(url, retry_count=3, delay_range=(1, 3)):
    """
    Fetch content from a URL with retry logic and random delays.
    
    Args:
        url (str): The URL to fetch
        retry_count (int): Number of retry attempts
        delay_range (tuple): Range for random delay between retries (min, max)
        
    Returns:
        tuple: (success (bool), response object or None, error message or None)
    """
    for attempt in range(retry_count):
        try:
            # Random delay to avoid rate limiting
            if attempt > 0:  # No delay on first attempt
                time.sleep(random.uniform(*delay_range))
                
            response = requests.get(url, headers=headers, timeout=10)
            response.raise_for_status()  # Raise exception for 4XX/5XX status codes
            
            return True, response, None
            
        except requests.exceptions.RequestException as e:
            error_msg = f"Attempt {attempt+1}/{retry_count} failed: {str(e)}"
            print(error_msg)
            
            # If it's the last attempt, return the error
            if attempt == retry_count - 1:
                return False, None, error_msg
    
    # Should never reach here due to return in the loop
    return False, None, "Unknown error"

def extract_links(html_content, base_url=None, filter_pattern=None):
    """
    Extract links from HTML content with optional filtering.
    
    Args:
        html_content (str): HTML content to parse
        base_url (str, optional): Base URL to resolve relative links
        filter_pattern (str, optional): Regex pattern to filter links
        
    Returns:
        list: List of extracted links
    """
    try:
        soup = BeautifulSoup(html_content, 'html.parser')
        links = []
        
        for a_tag in soup.find_all('a', href=True):
            href = a_tag['href']
            
            # Resolve relative URLs if base_url is provided
            if base_url and href.startswith('/'):
                href = base_url.rstrip('/') + '/' + href.lstrip('/')
                
            # Apply filter if provided
            if filter_pattern:
                import re
                if not re.search(filter_pattern, href):
                    continue
                    
            links.append({
                'url': href,
                'text': a_tag.get_text(strip=True) or "[No Text]",
                'title': a_tag.get('title', '')
            })
            
        return links
        
    except Exception as e:
        print(f"Error extracting links: {e}")
        return []

def save_results(data, filename, format='json'):
    """
    Save data to a file in the specified format.
    
    Args:
        data: The data to save
        filename (str): Output filename
        format (str): Output format ('json', 'csv', 'txt')
        
    Returns:
        bool: Success status
    """
    try:
        os.makedirs(os.path.dirname(os.path.abspath(filename)), exist_ok=True)
        
        if format.lower() == 'json':
            with open(filename, 'w', encoding='utf-8') as f:
                json.dump(data, f, indent=2, ensure_ascii=False)
                
        elif format.lower() == 'csv':
            if isinstance(data, list) and data:
                df = pd.DataFrame(data)
                df.to_csv(filename, index=False, encoding='utf-8')
            else:
                print("Error: Data must be a non-empty list for CSV format")
                return False
                
        elif format.lower() == 'txt':
            with open(filename, 'w', encoding='utf-8') as f:
                if isinstance(data, list):
                    for item in data:
                        f.write(str(item) + '\n')
                else:
                    f.write(str(data))
        else:
            print(f"Unsupported format: {format}")
            return False
            
        print(f"Data saved to {filename}")
        return True
        
    except Exception as e:
        print(f"Error saving data: {e}")
        return False

# Example usage of these functions
def demo_functions():
    target_url = "https://example.com"
    
    # Fetch the URL
    success, response, error = fetch_url(target_url)
    
    if success:
        print(f"Successfully fetched {target_url}")
        
        # Extract links
        links = extract_links(response.text, base_url=target_url)
        print(f"Found {len(links)} links")
        
        # Save results
        save_results(links, "results/example_links.json")
        
    else:
        print(f"Failed to fetch {target_url}: {error}")

# Uncomment to run the demo
# demo_functions()
```

**Creating OSINT Classes**

For more complex functionality, organizing related functions into classes provides better structure and state management.

```python
# Example of an OSINT class for domain reconnaissance
class DomainRecon:
    """Class for performing domain reconnaissance."""
    
    def __init__(self, domain, output_dir="results"):
        """
        Initialize the domain recon object.
        
        Args:
            domain (str): Target domain name
            output_dir (str): Directory to save results
        """
        self.domain = domain
        self.output_dir = output_dir
        self.results = {
            "whois": None,
            "dns_records": {},
            "subdomains": [],
            "headers": {},
            "links": []
        }
        
        # Create output directory if it doesn't exist
        os.makedirs(output_dir, exist_ok=True)
        
        # Log initialization
        print(f"Initialized DomainRecon for {domain}")
        
    def run_whois(self):
        """Perform WHOIS lookup on the domain."""
        try:
            import whois
            self.results["whois"] = whois.whois(self.domain)
            print(f"WHOIS lookup completed for {self.domain}")
            return True
        except Exception as e:
            print(f"WHOIS lookup failed: {e}")
            return False
            
    def check_dns_records(self, record_types=None):
        """
        Check DNS records for the domain.
        
        Args:
            record_types (list): List of record types to check
        """
        if record_types is None:
            record_types = ["A", "AAAA", "MX", "TXT", "NS", "CNAME"]
            
        try:
            import dns.resolver
            
            for rtype in record_types:
                try:
                    answers = dns.resolver.resolve(self.domain, rtype)
                    self.results["dns_records"][rtype] = [rdata.to_text() for rdata in answers]
                except dns.resolver.NoAnswer:
                    self.results["dns_records"][rtype] = []
                except Exception as e:
                    print(f"Error querying {rtype} records: {e}")
                    
            print(f"DNS records checked for {self.domain}")
            return True
            
        except Exception as e:
            print(f"DNS record check failed: {e}")
            return False
            
    def fetch_website(self):
        """Fetch the website and extract basic information."""
        url = f"https://{self.domain}"
        
        success, response, error = fetch_url(url)  # Using function from previous example
        
        if success:
            # Store headers
            self.results["headers"] = dict(response.headers)
            
            # Extract links
            self.results["links"] = extract_links(response.text, base_url=url)
            
            print(f"Website fetched for {self.domain}")
            return True
        else:
            print(f"Website fetch failed: {error}")
            return False
            
    def save_results(self):
        """Save all results to files."""
        base_filename = os.path.join(self.output_dir, self.domain.replace(".", "_"))
        
        # Save overall results
        save_results(self.results, f"{base_filename}_all_results.json")
        
        # Save individual components
        if self.results["whois"]:
            save_results(self.results["whois"], f"{base_filename}_whois.json")
            
        if self.results["dns_records"]:
            save_results(self.results["dns_records"], f"{base_filename}_dns.json")
            
        if self.results["links"]:
            save_results(self.results["links"], f"{base_filename}_links.csv", format="csv")
            
        print(f"Results saved for {self.domain}")
        
    def run_all(self):
        """Run all reconnaissance methods."""
        print(f"Starting full reconnaissance for {self.domain}")
        
        self.run_whois()
        self.check_dns_records()
        self.fetch_website()
        self.save_results()
        
        print(f"Reconnaissance completed for {self.domain}")
        return self.results

# Example usage
def demo_domain_recon():
    recon = DomainRecon("example.com")
    results = recon.run_all()
    
    # Access specific results
    if results["whois"]:
        print(f"Domain registrar: {results['whois'].registrar}")
        
    if "A" in results["dns_records"]:
        print(f"A records: {results['dns_records']['A']}")

# Uncomment to run the demo
# demo_domain_recon()
```

### Building OSINT Pipelines

OSINT investigations often involve a sequence of steps that can be chained together into a pipeline. This approach allows for more complex workflows while maintaining modularity.

```python
# Example of a simple OSINT pipeline
class OSINTPipeline:
    """Class for building and executing OSINT pipelines."""
    
    def __init__(self, name, output_dir="pipeline_results"):
        """
        Initialize the pipeline.
        
        Args:
            name (str): Pipeline name
            output_dir (str): Directory to save results
        """
        self.name = name
        self.output_dir = output_dir
        self.steps = []
        self.results = {}
        self.start_time = None
        self.end_time = None
        
        # Create output directory
        os.makedirs(output_dir, exist_ok=True)
        
    def add_step(self, name, function, **kwargs):
        """
        Add a step to the pipeline.
        
        Args:
            name (str): Step name
            function (callable): Function to execute
            **kwargs: Arguments to pass to the function
        """
        self.steps.append({
            "name": name,
            "function": function,
            "kwargs": kwargs,
            "status": "pending",
            "result": None,
            "error": None,
            "start_time": None,
            "end_time": None
        })
        
        print(f"Added step '{name}' to pipeline '{self.name}'")
        
    def execute(self):
        """Execute all steps in the pipeline."""
        import datetime
        
        self.start_time = datetime.datetime.now()
        print(f"Starting pipeline '{self.name}' at {self.start_time}")
        
        for i, step in enumerate(self.steps):
            step_num = i + 1
            print(f"\nExecuting step {step_num}/{len(self.steps)}: {step['name']}")
            
            step["start_time"] = datetime.datetime.now()
            step["status"] = "running"
            
            try:
                # Execute the function with its arguments
                step["result"] = step["function"](**step["kwargs"])
                step["status"] = "completed"
                
                # Store result in pipeline results
                self.results[step["name"]] = step["result"]
                
                print(f"Step '{step['name']}' completed successfully")
                
            except Exception as e:
                step["status"] = "failed"
                step["error"] = str(e)
                print(f"Step '{step['name']}' failed: {e}")
                
            step["end_time"] = datetime.datetime.now()
            duration = step["end_time"] - step["start_time"]
            print(f"Step duration: {duration}")
            
        self.end_time = datetime.datetime.now()
        total_duration = self.end_time - self.start_time
        print(f"\nPipeline '{self.name}' completed at {self.end_time}")
        print(f"Total duration: {total_duration}")
        
        # Save pipeline results
        self.save_results()
        
        return self.results
        
    def save_results(self):
        """Save pipeline results and metadata."""
        import datetime
        
        # Prepare metadata
        metadata = {
            "pipeline_name": self.name,
            "start_time": str(self.start_time) if self.start_time else None,
            "end_time": str(self.end_time) if self.end_time else None,
            "steps": [
                {
                    "name": step["name"],
                    "status": step["status"],
                    "start_time": str(step["start_time"]) if step["start_time"] else None,
                    "end_time": str(step["end_time"]) if step["end_time"] else None,
                    "error": step["error"]
                }
                for step in self.steps
            ]
        }
        
        # Save metadata
        filename = os.path.join(self.output_dir, f"{self.name}_metadata.json")
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(metadata, f, indent=2)
            
        # Save results for each step
        for step in self.steps:
            if step["result"] is not None:
                step_filename = os.path.join(self.output_dir, f"{self.name}_{step['name']}.json")
                try:
                    with open(step_filename, 'w', encoding='utf-8') as f:
                        json.dump(step["result"], f, indent=2, default=str)
                except Exception as e:
                    print(f"Error saving results for step '{step['name']}': {e}")
                    
        print(f"Pipeline results saved to {self.output_dir}")

# Example functions for the pipeline
def search_domain(domain):
    """Search for information about a domain."""
    print(f"Searching for domain: {domain}")
    # Simulate search results
    return {
        "domain": domain,
        "registrar": "Example Registrar, Inc.",
        "creation_date": "2020-01-01"
    }

def find_subdomains(domain, limit=5):
    """Find subdomains for a domain."""
    print(f"Finding subdomains for: {domain} (limit: {limit})")
    # Simulate subdomain discovery
    return {
        "domain": domain,
        "subdomains": [f"sub{i}.{domain}" for i in range(1, limit+1)]
    }

def analyze_content(domain):
    """Analyze website content."""
    print(f"Analyzing content for: {domain}")
    # Simulate content analysis
    return {
        "domain": domain,
        "title": "Example Domain",
        "keywords": ["example", "domain", "test"],
        "links": 10
    }

# Example pipeline usage
def demo_pipeline():
    pipeline = OSINTPipeline("domain_investigation")
    
    # Add steps
    pipeline.add_step("domain_search", search_domain, domain="example.com")
    pipeline.add_step("subdomain_discovery", find_subdomains, domain="example.com", limit=3)
    pipeline.add_step("content_analysis", analyze_content, domain="example.com")
    
    # Execute pipeline
    results = pipeline.execute()
    
    # Access results
    if "subdomain_discovery" in results:
        subdomains = results["subdomain_discovery"]["subdomains"]
        print(f"\nDiscovered subdomains: {subdomains}")

# Uncomment to run the demo
# demo_pipeline()
```

### Creating Simple Dashboards

Interactive dashboards can make your OSINT findings more accessible and easier to explore. Jupyter provides several options for creating dashboards directly within notebooks.

**Using `ipywidgets` for Interactive Controls**

`ipywidgets` allows you to create interactive controls (sliders, dropdowns, buttons) that can manipulate your data and visualizations in real-time.

```python
# !pip install ipywidgets

import ipywidgets as widgets
from IPython.display import display
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

# Create sample data
def generate_sample_data():
    # Simulate OSINT data for demonstration
    dates = pd.date_range(start='2025-01-01', periods=100, freq='D')
    sources = ['Twitter', 'News', 'Forums', 'Blogs', 'GitHub']
    
    data = []
    for _ in range(500):
        source = np.random.choice(sources)
        date = np.random.choice(dates)
        sentiment = np.random.choice(['Positive', 'Neutral', 'Negative'], 
                                    p=[0.3, 0.5, 0.2])
        relevance = np.random.randint(1, 11)
        
        data.append({
            'date': date,
            'source': source,
            'sentiment': sentiment,
            'relevance': relevance,
            'mentions': np.random.randint(1, 100)
        })
    
    return pd.DataFrame(data)

# Sample data for the dashboard
df_dashboard = generate_sample_data()

# Create widgets
source_dropdown = widgets.SelectMultiple(
    options=sorted(df_dashboard['source'].unique()),
    value=['Twitter', 'News'],
    description='Sources:',
    disabled=False
)

sentiment_dropdown = widgets.SelectMultiple(
    options=sorted(df_dashboard['sentiment'].unique()),
    value=['Positive', 'Negative', 'Neutral'],
    description='Sentiment:',
    disabled=False
)

date_range_slider = widgets.DateRangeSlider(
    value=[df_dashboard['date'].min(), df_dashboard['date'].max()],
    min=df_dashboard['date'].min(),
    max=df_dashboard['date'].max(),
    step=pd.Timedelta('1d'),
    description='Date Range:'
)

plot_type_dropdown = widgets.Dropdown(
    options=['Bar Chart', 'Line Chart', 'Scatter Plot'],
    value='Bar Chart',
    description='Plot Type:',
    disabled=False
)

# Function to update the plot based on widget values
def update_plot(sources, sentiments, date_range, plot_type):
    # Filter data based on selections
    filtered_df = df_dashboard[
        (df_dashboard['source'].isin(sources)) &
        (df_dashboard['sentiment'].isin(sentiments)) &
        (df_dashboard['date'] >= date_range[0]) &
        (df_dashboard['date'] <= date_range[1])
    ]
    
    if filtered_df.empty:
        plt.figure(figsize=(10, 6))
        plt.text(0.5, 0.5, "No data matches the selected filters", 
                 horizontalalignment='center', verticalalignment='center',
                 transform=plt.gca().transAxes, fontsize=14)
        plt.tight_layout()
        return
    
    # Aggregate data
    if plot_type == 'Line Chart':
        # Group by date for line chart
        agg_df = filtered_df.groupby(pd.Grouper(key='date', freq='W'))['mentions'].sum().reset_index()
        
        plt.figure(figsize=(12, 6))
        plt.plot(agg_df['date'], agg_df['mentions'], marker='o', linestyle='-')
        plt.title('Weekly Mentions Over Time')
        plt.xlabel('Date')
        plt.ylabel('Total Mentions')
        plt.grid(True)
        
    elif plot_type == 'Bar Chart':
        # Group by source and sentiment
        agg_df = filtered_df.groupby(['source', 'sentiment'])['mentions'].sum().reset_index()
        
        plt.figure(figsize=(12, 6))
        pivot_df = agg_df.pivot(index='source', columns='sentiment', values='mentions')
        pivot_df.plot(kind='bar', ax=plt.gca())
        plt.title('Mentions by Source and Sentiment')
        plt.xlabel('Source')
        plt.ylabel('Total Mentions')
        plt.legend(title='Sentiment')
        
    elif plot_type == 'Scatter Plot':
        plt.figure(figsize=(12, 6))
        for source in filtered_df['source'].unique():
            source_df = filtered_df[filtered_df['source'] == source]
            plt.scatter(source_df['relevance'], source_df['mentions'], 
                       label=source, alpha=0.7)
        
        plt.title('Mentions vs. Relevance by Source')
        plt.xlabel('Relevance Score')
        plt.ylabel('Mentions')
        plt.legend(title='Source')
        plt.grid(True)
    
    plt.tight_layout()

# Create interactive output
def create_dashboard():
    # Interactive output
    output = widgets.Output()
    
    # Update function for widgets
    def on_change(change):
        with output:
            output.clear_output(wait=True)
            update_plot(
                source_dropdown.value,
                sentiment_dropdown.value,
                date_range_slider.value,
                plot_type_dropdown.value
            )
    
    # Register callbacks
    source_dropdown.observe(on_change, names='value')
    sentiment_dropdown.observe(on_change, names='value')
    date_range_slider.observe(on_change, names='value')
    plot_type_dropdown.observe(on_change, names='value')
    
    # Initial plot
    with output:
        update_plot(
            source_dropdown.value,
            sentiment_dropdown.value,
            date_range_slider.value,
            plot_type_dropdown.value
        )
    
    # Layout
    controls = widgets.VBox([
        widgets.HBox([source_dropdown, sentiment_dropdown]),
        widgets.HBox([date_range_slider, plot_type_dropdown])
    ])
    
    # Display dashboard
    display(widgets.VBox([controls, output]))

# Uncomment to create the dashboard
# create_dashboard()
```

**Using `voila` or `Panel` for Standalone Dashboards**

For more sophisticated dashboards that can be shared outside of Jupyter, libraries like `voila` and `Panel` can convert notebooks into standalone web applications.

```python
# !pip install voila panel

# Example of a simple Panel dashboard
import panel as pn
import pandas as pd
import hvplot.pandas  # For interactive plots

# Initialize Panel
pn.extension()

# Sample data (reuse from previous example)
# df_dashboard = generate_sample_data()

# Create Panel widgets
source_select = pn.widgets.MultiSelect(
    name='Sources',
    options=list(df_dashboard['source'].unique()),
    value=['Twitter', 'News']
)

sentiment_select = pn.widgets.MultiSelect(
    name='Sentiment',
    options=list(df_dashboard['sentiment'].unique()),
    value=list(df_dashboard['sentiment'].unique())
)

date_range = pn.widgets.DateRangeSlider(
    name='Date Range',
    start=df_dashboard['date'].min(),
    end=df_dashboard['date'].max(),
    value=(df_dashboard['date'].min(), df_dashboard['date'].max())
)

# Create a function to filter the data
@pn.depends(source_select.param.value, sentiment_select.param.value, date_range.param.value)
def filter_data(sources, sentiments, date_range):
    filtered_df = df_dashboard[
        (df_dashboard['source'].isin(sources)) &
        (df_dashboard['sentiment'].isin(sentiments)) &
        (df_dashboard['date'] >= date_range[0]) &
        (df_dashboard['date'] <= date_range[1])
    ]
    return filtered_df

# Create interactive plots
@pn.depends(filter_data)
def create_bar_chart(data):
    if data.empty:
        return pn.pane.Markdown("No data matches the selected filters")
    
    agg_df = data.groupby(['source', 'sentiment'])['mentions'].sum().reset_index()
    return agg_df.hvplot.bar(
        x='source', y='mentions', by='sentiment', 
        title='Mentions by Source and Sentiment',
        height=400, width=600
    )

@pn.depends(filter_data)
def create_line_chart(data):
    if data.empty:
        return pn.pane.Markdown("No data matches the selected filters")
    
    agg_df = data.groupby(pd.Grouper(key='date', freq='W'))['mentions'].sum().reset_index()
    return agg_df.hvplot.line(
        x='date', y='mentions', 
        title='Weekly Mentions Over Time',
        height=400, width=600
    )

# Create the dashboard layout
def create_panel_dashboard():
    dashboard = pn.Column(
        pn.pane.Markdown("# OSINT Data Dashboard"),
        pn.Row(
            pn.Column(
                pn.pane.Markdown("## Filters"),
                source_select,
                sentiment_select,
                date_range
            ),
            pn.Column(
                pn.pane.Markdown("## Visualizations"),
                create_bar_chart,
                create_line_chart
            )
        )
    )
    
    return dashboard

# To display in the notebook
# panel_dashboard = create_panel_dashboard()
# panel_dashboard

# To serve as a standalone app (uncomment to run)
# panel_dashboard.servable()
# print("Run 'panel serve notebook_name.ipynb' to start the dashboard server")

print("\nDashboard examples are conceptual. Run the code in a Jupyter environment to see the interactive elements.")
```

### Error Handling and Logging

Robust OSINT scripts need proper error handling and logging to ensure reliability and provide an audit trail.

```python
import logging
import traceback
import sys
import os
from datetime import datetime

class OSINTLogger:
    """Class for handling logging in OSINT scripts."""
    
    def __init__(self, name, log_dir="logs", console_level=logging.INFO, file_level=logging.DEBUG):
        """
        Initialize the logger.
        
        Args:
            name (str): Logger name
            log_dir (str): Directory to save log files
            console_level: Logging level for console output
            file_level: Logging level for file output
        """
        self.name = name
        self.log_dir = log_dir
        
        # Create log directory
        os.makedirs(log_dir, exist_ok=True)
        
        # Create logger
        self.logger = logging.getLogger(name)
        self.logger.setLevel(logging.DEBUG)  # Capture all levels
        self.logger.propagate = False  # Don't propagate to parent loggers
        
        # Clear any existing handlers
        if self.logger.handlers:
            self.logger.handlers.clear()
        
        # Create console handler
        console_handler = logging.StreamHandler(sys.stdout)
        console_handler.setLevel(console_level)
        console_format = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
        console_handler.setFormatter(console_format)
        self.logger.addHandler(console_handler)
        
        # Create file handler
        log_file = os.path.join(log_dir, f"{name}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.log")
        file_handler = logging.FileHandler(log_file)
        file_handler.setLevel(file_level)
        file_format = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
        file_handler.setFormatter(file_format)
        self.logger.addHandler(file_handler)
        
        self.logger.info(f"Logger initialized: {name}")
        self.logger.info(f"Log file: {log_file}")
        
    def get_logger(self):
        """Get the logger instance."""
        return self.logger

# Example of a function with proper error handling and logging
def safe_osint_function(func):
    """
    Decorator for OSINT functions to add error handling and logging.
    
    Args:
        func: The function to decorate
        
    Returns:
        Wrapped function with error handling
    """
    def wrapper(*args, **kwargs):
        # Get logger - either from kwargs or create a default one
        logger = kwargs.pop('logger', logging.getLogger(__name__))
        
        func_name = func.__name__
        logger.info(f"Starting {func_name}")
        
        try:
            # Execute the function
            result = func(*args, **kwargs)
            logger.info(f"Successfully completed {func_name}")
            return result
            
        except Exception as e:
            # Log the error with traceback
            logger.error(f"Error in {func_name}: {str(e)}")
            logger.debug(f"Traceback: {traceback.format_exc()}")
            
            # Re-raise or return None/error indicator
            # Uncomment to re-raise: raise
            return None
            
    return wrapper

# Example usage
@safe_osint_function
def risky_operation(target, depth=1, logger=None):
    """Example function that might fail."""
    if logger:
        logger.info(f"Processing target: {target} with depth {depth}")
        
    if depth > 3:
        raise ValueError("Depth too high")
        
    # Simulate processing
    result = f"Processed {target} at depth {depth}"
    
    if logger:
        logger.debug(f"Result: {result}")
        
    return result

def demo_error_handling():
    # Create logger
    osint_logger = OSINTLogger("demo_osint").get_logger()
    
    # Test successful operation
    result1 = risky_operation("example.com", depth=2, logger=osint_logger)
    print(f"Result 1: {result1}")
    
    # Test failing operation
    result2 = risky_operation("example.org", depth=5, logger=osint_logger)
    print(f"Result 2: {result2}")

# Uncomment to run the demo
# demo_error_handling()
```

### Scheduling Tasks

For long-term monitoring or periodic data collection, scheduling your OSINT scripts to run automatically is valuable.

**Conceptual Overview of Scheduling Options:**

1. **Using `cron` on Linux/macOS:**
   - Edit crontab with `crontab -e`
   - Add entry like `0 9 * * * cd /path/to/project && python3 osint_script.py`
   - This runs the script daily at 9 AM

2. **Using Task Scheduler on Windows:**
   - Create a batch file (.bat) that runs your Python script
   - Open Task Scheduler and create a new task
   - Set trigger (e.g., daily at specific time)
   - Set action to run the batch file

3. **Using Cloud Services:**
   - AWS Lambda with CloudWatch Events
   - Google Cloud Functions with Cloud Scheduler
   - Azure Functions with Timer triggers

4. **Using Python Libraries:**
   - `schedule` library for in-process scheduling
   - `APScheduler` for more advanced scheduling

```python
# Example using the schedule library
# !pip install schedule

import schedule
import time
import datetime

def osint_job():
    """Example OSINT job to be scheduled."""
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    print(f"[{timestamp}] Running scheduled OSINT job...")
    
    # Simulate OSINT task
    print("Collecting data...")
    time.sleep(2)
    print("Analyzing results...")
    time.sleep(1)
    print("Saving report...")
    time.sleep(1)
    
    print(f"[{timestamp}] Job completed")

def demo_scheduling():
    print("Setting up scheduled job...")
    
    # Schedule job to run every minute for demo purposes
    schedule.every(1).minutes.do(osint_job)
    
    print("Job scheduled. Press Ctrl+C to exit.")
    
    # Run the job immediately once
    osint_job()
    
    # Keep the script running to execute scheduled jobs
    try:
        while True:
            schedule.run_pending()
            time.sleep(1)
    except KeyboardInterrupt:
        print("\nScheduling stopped.")

# Note: This is for demonstration only
# In a real scenario, you would use system schedulers for reliability
print("Scheduling example is conceptual. Run in a proper environment for actual scheduling.")
# demo_scheduling()
```

### Assignment 6: Automation and Workflow Building

**Objective:** Develop an automated OSINT workflow that performs a specific, repetitive task and presents the results in an organized manner.

**Instructions:**
1.  Choose a specific OSINT task that would benefit from automation (e.g., monitoring specific keywords on social media, tracking changes on a list of websites, periodically checking domain registrations, monitoring GitHub repositories for specific organizations).
2.  Design and implement a modular, reusable solution using the techniques covered in this section:
    *   Create well-structured functions and/or classes to encapsulate specific tasks.
    *   Build a pipeline that chains these functions together in a logical workflow.
    *   Implement proper error handling and logging.
    *   (Optional but recommended) Add a simple dashboard or interactive elements to explore the results.
3.  Your solution should:
    *   Accept parameters to customize its behavior (e.g., target domains, keywords, time periods).
    *   Save results in a structured format (CSV, JSON, database).
    *   Generate a summary report or visualization of findings.
    *   Include documentation on how to use and extend the tool.
4.  Test your solution with real-world data (within ethical boundaries).
5.  Discuss how the solution could be scheduled to run periodically (conceptually, no need to implement actual scheduling).

**Deliverables:**
- A Jupyter Notebook (.ipynb file) containing your code, documentation, and example usage.
- Any Python module files (.py) created for your solution.
- Sample output files generated by your tool.
- A brief report discussing the design decisions, challenges encountered, and potential improvements.

**Evaluation Criteria:**
- Modularity and reusability of the code.
- Effectiveness of the automation in addressing the chosen task.
- Quality of error handling and logging.
- Clarity and organization of results presentation.
- Documentation quality and usability of the solution.
- Adherence to ethical and legal considerations in the automation design.
