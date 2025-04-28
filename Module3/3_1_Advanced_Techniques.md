# Module 3: Advanced - Specialized Techniques and Automation

Welcome to Module 3, where we elevate your OSINT capabilities to an expert level. Building upon the data collection, analysis, and visualization skills from the previous modules, we now delve into specialized OSINT domains, advanced analytical methods, workflow automation, and a deeper examination of the critical ethical and legal landscape. This module aims to equip you with the techniques and mindset required for complex investigations and strategic intelligence gathering using Jupyter Notebook.

## 3.1 Advanced OSINT Techniques

This section explores sophisticated techniques tailored to specific types of OSINT investigations, leveraging specialized tools and Python libraries within the Jupyter environment.

### Social Media Intelligence (SOCMINT)

SOCMINT involves gathering and analyzing information from social media platforms to gain insights into individuals, groups, events, or public sentiment. It goes beyond simple profile browsing.

*   **Deeper Analysis:**
    *   **Network Analysis:** Mapping connections between users (followers, friends, interactions) using `NetworkX`. This can reveal communities, influencers, and hidden relationships. Data might come from APIs (if available and permitted) or careful scraping (ethically sourced).
    *   **Content Analysis:** Analyzing the text, images, and metadata associated with posts. This includes identifying topics, keywords, locations, and timestamps.
    *   **Sentiment Analysis:** Determining the emotional tone (positive, negative, neutral) expressed in social media text. This is useful for brand monitoring, political analysis, or understanding public reaction.
        *   **Using NLP Libraries:** Libraries like `NLTK` (Natural Language Toolkit), `spaCy`, or pre-trained models (e.g., from Hugging Face Transformers) can be used. Simple approaches might use lexicon-based methods (like VADER), while more advanced techniques involve machine learning models.

```python
# Conceptual Example: Sentiment Analysis using VADER (Valence Aware Dictionary and sEntiment Reasoner)
# !pip install vaderSentiment

from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
import pandas as pd

# Sample social media posts (replace with actual collected data)
posts = [
    "This new OSINT tool is amazing! So powerful and easy to use.",
    "I am incredibly frustrated with the lack of privacy on this platform.",
    "Just attended a great webinar on Jupyter for OSINT.",
    "The recent data breach is deeply concerning.",
    "The weather today is quite pleasant."
]

df_posts = pd.DataFrame({"text": posts})

# Initialize VADER analyzer
analyzer = SentimentIntensityAnalyzer()

# Function to get sentiment scores
def get_sentiment(text):
    vs = analyzer.polarity_scores(text)
    return vs["compound"] # Compound score ranges from -1 (most negative) to +1 (most positive)

# Apply sentiment analysis
df_posts["sentiment_score"] = df_posts["text"].apply(get_sentiment)

# Categorize sentiment
def categorize_sentiment(score):
    if score >= 0.05:
        return "Positive"
    elif score <= -0.05:
        return "Negative"
    else:
        return "Neutral"

df_posts["sentiment_category"] = df_posts["sentiment_score"].apply(categorize_sentiment)

print("--- Sentiment Analysis Results ---")
print(df_posts)

# Visualize sentiment distribution
import matplotlib.pyplot as plt
import seaborn as sns

sns.countplot(x="sentiment_category", data=df_posts, palette=["red", "grey", "green"])
plt.title("Distribution of Sentiment in Posts")
plt.show()
```

*   **Challenges:** API limitations, platform transience, fake accounts, information overload, privacy concerns.

### Geolocation and Mapping

Determining the geographic location associated with OSINT data points.

*   **EXIF Metadata:** Exchangeable Image File Format data embedded in images (and some videos) by cameras and smartphones. Can contain GPS coordinates, timestamps, camera model, etc.
    *   **Using `ExifRead` or `Pillow`:**

```python
# !pip install ExifRead Pillow

import exifread
from PIL import Image
from PIL.ExifTags import TAGS, GPSTAGS
import io
import requests # To fetch image from URL

# Function to extract EXIF using exifread (more comprehensive for raw tags)
def get_exif_data_exifread(image_bytes):
    try:
        tags = exifread.process_file(io.BytesIO(image_bytes))
        if not tags:
            print("No EXIF metadata found (exifread).")
            return None
        
        print("EXIF Data (exifread):")
        exif_data = {}
        for tag, value in tags.items():
            if tag not in (".*Thumbnail.*", ".*MakerNote.*"): # Exclude bulky tags
                 print(f"  {tag}: {value}")
                 exif_data[tag] = str(value) # Convert to string for consistency
        return exif_data
    except Exception as e:
        print(f"Error reading EXIF with exifread: {e}")
        return None

# Function to extract and decode GPS info using Pillow
def get_gps_info_pillow(image_bytes):
    try:
        img = Image.open(io.BytesIO(image_bytes))
        exif_data = img._getexif()
        if not exif_data:
            print("No EXIF metadata found (Pillow).")
            return None

        gps_info = {}
        for k, v in exif_data.items():
            tag_name = TAGS.get(k)
            if tag_name == "GPSInfo":
                for gps_k, gps_v in v.items():
                    gps_tag_name = GPSTAGS.get(gps_k, gps_k)
                    gps_info[gps_tag_name] = gps_v
                break # Found GPSInfo
        
        if not gps_info:
            print("No GPS tags found in EXIF data.")
            return None

        # Decode GPS coordinates
        def dms_to_decimal(dms, ref):
            degrees = dms[0]
            minutes = dms[1] / 60.0
            seconds = dms[2] / 3600.0
            decimal = degrees + minutes + seconds
            if ref in ["S", "W"]:
                decimal = -decimal
            return decimal

        lat = dms_to_decimal(gps_info["GPSLatitude"], gps_info["GPSLatitudeRef"])
        lon = dms_to_decimal(gps_info["GPSLongitude"], gps_info["GPSLongitudeRef"])
        
        print(f"GPS Coordinates (Pillow): Latitude={lat}, Longitude={lon}")
        return {"latitude": lat, "longitude": lon, "raw_gps": gps_info}

    except Exception as e:
        print(f"Error reading GPS info with Pillow: {e}")
        return None

# --- Example Usage ---
# Replace with a URL of an image known to have EXIF GPS data
image_url = "https://raw.githubusercontent.com/ianare/exif-samples/master/jpg/gps/DSCN0010.jpg" 

try:
    response = requests.get(image_url)
    response.raise_for_status()
    image_bytes = response.content
    print(f"Fetched image from {image_url}\n")
    
    # Get general EXIF
    # get_exif_data_exifread(image_bytes)
    
    # Get decoded GPS
    gps_coords = get_gps_info_pillow(image_bytes)
    
    # If GPS found, plot on map using Folium (from section 2.3)
    if gps_coords:
        import folium
        m_gps = folium.Map(location=[gps_coords["latitude"], gps_coords["longitude"]], zoom_start=15)
        folium.Marker(
            location=[gps_coords["latitude"], gps_coords["longitude"]],
            popup="Image Location",
            tooltip="Click for info"
        ).add_to(m_gps)
        # Display map
        # m_gps 
        m_gps.save("image_location_map.html")
        print("\nGPS map saved to image_location_map.html")

except requests.exceptions.RequestException as e:
    print(f"Error fetching image: {e}")
except Exception as e:
    print(f"An error occurred: {e}")
```
*   **IP Geolocation:** Estimating the geographic location of an IP address using databases (e.g., MaxMind GeoIP) or APIs. Accuracy varies significantly (country level is usually reliable, city level less so, street level is highly unlikely).
    *   **Using `ipwhois` or external APIs:**

```python
# !pip install ipwhois

from ipwhois import IPWhois
import json

ip_address = "8.8.8.8" # Example: Google Public DNS

try:
    obj = IPWhois(ip_address)
    results = obj.lookup_rdap(depth=1)
    
    print(f"--- IP Geolocation for {ip_address} ---")
    # Extract relevant geo fields (availability varies)
    asn_info = results.get("asn_description", "N/A")
    country = results.get("asn_country_code", "N/A")
    # Network info might contain city/region, but RDAP is often less detailed than commercial DBs
    network_info = results.get("network", {})
    city = network_info.get("city", "N/A")
    name = network_info.get("name", "N/A")
    
    print(f"ASN Description: {asn_info}")
    print(f"Country Code: {country}")
    print(f"Network Name: {name}")
    print(f"City (from network, if available): {city}")
    
    # Print full results for inspection
    # print("\nFull RDAP Results:")
    # print(json.dumps(results, indent=4))
    
except Exception as e:
    print(f"Error performing IP lookup for {ip_address}: {e}")

# Note: For more precise geolocation, commercial databases/APIs 
# (like MaxMind GeoLite2/GeoIP2) are often needed.
# Example using a hypothetical API:
# try:
#     geo_api_url = f"https://api.ipgeolocation.io/ipgeo?apiKey=YOUR_KEY&ip={ip_address}"
#     response = requests.get(geo_api_url)
#     geo_data = response.json()
#     print(f"\nGeolocation from API ({geo_api_url.split('/')[2]}):")
#     print(f"  City: {geo_data.get("city")}")
#     print(f"  Region: {geo_data.get("state_prov")}")
#     print(f"  Lat/Lon: {geo_data.get("latitude")}/{geo_data.get("longitude")}")
# except Exception as e:
#     print(f"Error using geolocation API: {e}")
```
*   **Correlating Data Points:** Combining location data from multiple sources (social media check-ins, photo metadata, IP logs) to build a more accurate picture.
*   **Advanced Mapping:** Using `geopandas` for spatial analysis (e.g., proximity analysis, overlaying datasets) and creating sophisticated thematic maps.

### Image & Video Forensics

Analyzing images and videos for authenticity, origin, and hidden information.

*   **Advanced Metadata Analysis:** Going beyond basic EXIF. Looking for embedded thumbnails, software used for editing (e.g., Photoshop metadata), timestamps across different metadata standards.
*   **Reverse Image Searching:** Finding the origin or other instances of an image online. Can be automated by interacting with services like Google Images, TinEye, Yandex Images (often requires `Selenium` or unofficial APIs).
*   **Identifying Manipulation (Error Level Analysis - ELA, etc.):** Detecting inconsistencies that suggest digital alteration. Specialized tools or libraries exist, but visual inspection and understanding compression artifacts are key. (Implementing ELA in Python is complex and beyond basic libraries).
*   **Video Analysis Concepts:** Extracting frames, analyzing video metadata, searching for video content online, analyzing audio tracks. Libraries like `OpenCV` (`cv2`) can be used for frame extraction and basic analysis.

```python
# Example: Frame extraction using OpenCV
# !pip install opencv-python

import cv2
import os

video_path = "path/to/your/video.mp4" # Replace with actual video path
output_folder = "video_frames"

if not os.path.exists(output_folder):
    os.makedirs(output_folder)

try:
    vidcap = cv2.VideoCapture(video_path)
    success, image = vidcap.read()
    count = 0
    frame_interval = 30 # Extract every 30th frame (adjust as needed)

    print(f"Extracting frames from {video_path}...")
    while success:
        if count % frame_interval == 0:
            frame_filename = os.path.join(output_folder, f"frame_{count:06d}.jpg")
            cv2.imwrite(frame_filename, image) # save frame as JPEG file
            # print(f"Saved {frame_filename}")
        success, image = vidcap.read()
        count += 1
    
    print(f"Finished extracting frames. Total frames processed: {count}")
    vidcap.release()

except Exception as e:
    print(f"Error processing video {video_path}: {e}")
```

### Domain & Infrastructure Analysis

Investigating websites, servers, and the underlying infrastructure.

*   **WHOIS Lookups:** Retrieving registration information for domain names (registrant details often redacted now due to privacy laws, but registrar, registration/expiry dates, nameservers are useful).
    *   **Using `python-whois` or APIs:**

```python
# !pip install python-whois

import whois

domain = "google.com"

try:
    w = whois.whois(domain)
    print(f"--- WHOIS for {domain} ---")
    print(f"Registrar: {w.registrar}")
    print(f"Creation Date: {w.creation_date}")
    print(f"Expiration Date: {w.expiration_date}")
    print(f"Name Servers: {w.name_servers}")
    # print("\nFull WHOIS Record:")
    # print(w)
except Exception as e:
    print(f"Error performing WHOIS lookup for {domain}: {e}")
```
*   **DNS Analysis:**
    *   **Current Records:** Querying DNS records (A, AAAA, MX, TXT, NS, CNAME) to find IP addresses, mail servers, verification codes, etc. Use libraries like `dnspython`.
    *   **Reverse DNS:** Finding domain names associated with an IP address (PTR records).
    *   **Historical DNS:** Using services like SecurityTrails, RiskIQ, VirusTotal to find past DNS records, revealing infrastructure changes or related domains.
*   **SSL/TLS Certificate Analysis:** Examining certificates associated with a domain/IP. Certificate transparency logs (e.g., crt.sh) can reveal subdomains and related certificates issued to an organization.
*   **Identifying Related Infrastructure:** Looking for shared IP addresses, hosting providers, analytics IDs (e.g., Google Analytics UA-XXXX), common server headers, or WHOIS details to link different websites or services.

```python
# Example: Basic DNS lookup using dnspython
# !pip install dnspython

import dns.resolver

domain = "google.com"
record_types = ["A", "AAAA", "MX", "TXT", "NS"]

print(f"\n--- DNS Records for {domain} ---")
for rtype in record_types:
    try:
        answers = dns.resolver.resolve(domain, rtype)
        print(f"  {rtype} Records:")
        for rdata in answers:
            print(f"    {rdata.to_text()}")
    except dns.resolver.NoAnswer:
        print(f"  No {rtype} records found.")
    except Exception as e:
        print(f"  Error querying {rtype} records: {e}")
```

### Username & Email Analysis

Investigating user identifiers across the internet.

*   **Username Checking:** Searching for the presence of a specific username across multiple online platforms. Tools like `Sherlock`, `WhatsMyName.app` automate this. Can be implemented in Python by scraping or using APIs of checker sites (respect their terms).

```python
# Conceptual implementation of username checking (simplified)
import requests
import time

def check_username(username, site_url_template):
    """Checks if a username exists on a site based on URL pattern."""
    check_url = site_url_template.format(username=username)
    try:
        response = requests.get(check_url, headers=headers, timeout=5) # Use headers from earlier
        # Check based on status code or content (site-specific)
        if response.status_code == 200: # Simple check, might need refinement
            return True, check_url
        else:
            return False, check_url
    except requests.exceptions.RequestException:
        return False, check_url

# List of sites with known URL patterns (example)
sites = {
    "GitHub": "https://github.com/{username}",
    "Twitter": "https://twitter.com/{username}",
    "Reddit": "https://www.reddit.com/user/{username}"
}

target_username = "octocat" # Example

print(f"\n--- Checking username: {target_username} ---")
found_on = []
for site_name, url_template in sites.items():
    exists, checked_url = check_username(target_username, url_template)
    if exists:
        print(f"[+] Found on {site_name}: {checked_url}")
        found_on.append(site_name)
    else:
        print(f"[-] Not found or error on {site_name}")
    time.sleep(0.5) # Be polite
```
*   **Email Header Analysis:** Examining the full headers of an email to trace its path, identify originating IP addresses, and detect potential spoofing.
*   **Breach Data Searching:** Checking if an email address or username appears in known data breaches using services like Have I Been Pwned (HIBP - has an official API). This can reveal associated passwords (never store/use these!), other usernames, and compromised services.

### Phone Number Intelligence

Gathering information related to a phone number.

*   **Using the `phonenumbers` library:** Validation, determining country and carrier, potential timezone.

```python
# !pip install phonenumbers

import phonenumbers
from phonenumbers import geocoder, carrier, timezone

phone_number_str = "+14155552671" # Example number (Twilio Docs)

try:
    # Parse the number
    parsed_number = phonenumbers.parse(phone_number_str, None)
    
    print(f"--- Phone Number Intelligence for {phone_number_str} ---")
    
    # Validation
    is_valid = phonenumbers.is_valid_number(parsed_number)
    print(f"Is Valid: {is_valid}")
    
    if is_valid:
        # Geolocation (general region)
        region = geocoder.description_for_number(parsed_number, "en")
        print(f"Region: {region}")
        
        # Carrier
        carrier_name = carrier.name_for_number(parsed_number, "en")
        print(f"Carrier: {carrier_name}")
        
        # Timezone
        time_zones = timezone.time_zones_for_number(parsed_number)
        print(f"Time Zones: {time_zones}")
        
        # Formatting
        formatted_national = phonenumbers.format_number(parsed_number, phonenumbers.PhoneNumberFormat.NATIONAL)
        formatted_international = phonenumbers.format_number(parsed_number, phonenumbers.PhoneNumberFormat.INTERNATIONAL)
        print(f"Formatted (National): {formatted_national}")
        print(f"Formatted (International): {formatted_international}")
        
except phonenumbers.phonenumberutil.NumberParseException as e:
    print(f"Error parsing phone number: {e}")
except Exception as e:
    print(f"An error occurred: {e}")
```
*   **Reverse Lookups:** Using online services (often paid, legality/ethics vary) to attempt to find the owner associated with a number.
*   **Social Media/App Associations:** Checking if the number is linked to accounts on platforms like WhatsApp, Telegram, Signal, or social media recovery options.

### Dark Web OSINT

Exploring the parts of the internet requiring specific software (like Tor) for access.

*   **Understanding Tor:** How it works (onion routing), its purpose (anonymity, censorship circumvention), and associated risks.
*   **Safety Precautions:** **Crucial.** Use a dedicated, isolated environment (like a VM), disable scripts (e.g., via Tor Browser settings), avoid downloading files, do not use personal information, be aware of malware and phishing risks, understand the legal implications in your jurisdiction.
*   **Specific Search Engines:** Ahmia, Torch, Haystak (effectiveness varies).
*   **Marketplaces and Forums:** Exploring illicit markets (for threat intelligence, stolen data research) and forums. Requires extreme caution and awareness of illegal content.
*   **Limitations and Risks:** Information is often unreliable, outdated, or intentionally misleading. High risk of encountering illegal/disturbing content and malware. Attribution is extremely difficult.
*   **Jupyter Integration:** Direct interaction with Tor from standard Python `requests` requires proxies (e.g., `socks5h://localhost:9050` if Tor service is running locally). `Selenium` can be configured to use the Tor Browser. **This should only be attempted by advanced users with a full understanding of the risks and necessary security measures.**

### Integrating External Tools

Many powerful OSINT tools run standalone (e.g., Recon-ng, theHarvester, Maltego). Jupyter can be used to process their output.

*   **Parsing Output:** Most tools can export results to formats like CSV, JSON, or XML. Use `pandas` or standard libraries (`csv`, `json`, `xml.etree.ElementTree`) to read this data into Jupyter.
*   **Combining Results:** Merge data from multiple tools into a single DataFrame for unified analysis.
*   **Visualization:** Use `matplotlib`, `seaborn`, `NetworkX`, etc., to visualize the combined data.
*   **Automation (Advanced):** Use Python's `subprocess` module to run command-line tools directly from Jupyter and capture their output (handle errors carefully).

```python
# Conceptual Example: Running a command-line tool
import subprocess
import pandas as pd
import io

# Example: Run theHarvester (assuming it's installed and in PATH)
# target_domain = "example.com"
# output_file = "harvester_output.csv"
# command = [
#     "theHarvester",
#     "-d", target_domain,
#     "-b", "google,bing", # Specify sources
#     "-f", output_file # Save results to CSV
# ]
# 
# try:
#     print(f"Running theHarvester for {target_domain}...")
#     # Run the command
#     result = subprocess.run(command, capture_output=True, text=True, check=True, timeout=120)
#     print("theHarvester finished.")
#     print("Output:")
#     print(result.stdout[-500:]) # Print last 500 chars of output
#     
#     # Read the output file (assuming CSV format)
#     # Note: theHarvester's output format might vary; adjust parsing accordingly
#     # df_harvester = pd.read_csv(output_file)
#     # print("\nData read from harvester output:")
#     # print(df_harvester.head())
#     
# except FileNotFoundError:
#     print("Error: theHarvester command not found. Is it installed and in PATH?")
# except subprocess.CalledProcessError as e:
#     print(f"Error running theHarvester: {e}")
#     print(f"Stderr: {e.stderr}")
# except subprocess.TimeoutExpired:
#     print("theHarvester command timed out.")
# except Exception as e:
#     print(f"An unexpected error occurred: {e}")

print("\nExternal tool integration example is conceptual.")
```

### Assignment 5: Multi-Faceted OSINT Investigation

**Objective:** Conduct a comprehensive OSINT investigation using a combination of advanced techniques learned in this module.

**Instructions:**
1.  Choose or be assigned a realistic OSINT scenario (e.g., investigate a suspicious domain/IP address, profile a public online persona, analyze a potential disinformation campaign, map the infrastructure of a known phishing kit).
2.  Develop an investigation plan within your Jupyter Notebook.
3.  Apply at least **three** different advanced techniques from section 3.1 (e.g., SOCMINT analysis, geolocation, domain/infrastructure analysis, username checking, metadata extraction).
4.  Use appropriate Python libraries (`NetworkX`, `ExifRead`/`Pillow`, `whois`, `dnspython`, `phonenumbers`, `requests`, `BeautifulSoup`/`Selenium`, `pandas`, visualization libraries) within Jupyter to execute your plan.
5.  Integrate data from multiple sources (e.g., combine WHOIS data with DNS records and website content analysis).
6.  Thoroughly document your entire process: methodology, tools used, code, justifications for actions, and raw findings.
7.  Analyze and synthesize your findings, drawing connections and potential conclusions.
8.  Visualize key findings using appropriate charts, graphs, or maps.
9.  Pay strict attention to ethical and legal considerations throughout the investigation. Document any ethical dilemmas encountered and how you addressed them.

**Deliverables:**
- A detailed Jupyter Notebook (.ipynb file) documenting the investigation from planning to conclusion, including code, analysis, visualizations, and ethical considerations.
- Any relevant data files collected or generated (e.g., CSVs, JSON files, maps).

**Evaluation Criteria:**
- Clarity and feasibility of the investigation plan.
- Appropriate application of at least three advanced OSINT techniques.
- Correct use of relevant Python libraries and tools.
- Successful integration and analysis of data from multiple sources.
- Depth and quality of analysis and insights derived.
- Effectiveness of visualizations in communicating findings.
- Thoroughness and clarity of documentation.
- Demonstrated awareness and handling of ethical and legal considerations.
