## 2.3 Data Visualization for OSINT

Analysis often generates complex data and insights that are difficult to grasp through tables or text alone. Data visualization translates this information into visual contexts, such as maps or graphs, making it easier for the human brain to understand trends, outliers, and patterns. In OSINT, effective visualization is crucial for exploring data, communicating findings, and producing compelling intelligence reports. This section covers key Python libraries for creating static and interactive visualizations within Jupyter Notebook.

### Principles of Effective Visualization

Before diving into specific libraries, consider these principles for creating impactful visualizations:

*   **Know Your Audience and Purpose:** Who are you communicating with? What key message do you want to convey? Tailor the visualization complexity and type accordingly.
*   **Choose the Right Chart Type:** Different charts excel at showing different types of data and relationships:
    *   **Bar Charts:** Comparing discrete categories.
    *   **Line Charts:** Showing trends over time or continuous data.
    *   **Scatter Plots:** Revealing relationships and correlations between two numerical variables.
    *   **Pie Charts:** Showing parts of a whole (use sparingly, often less effective than bar charts for comparison).
    *   **Histograms:** Displaying the distribution of a single numerical variable.
    *   **Box Plots:** Comparing distributions across categories, showing median, quartiles, and outliers.
    *   **Heatmaps:** Visualizing matrix data, showing intensity or correlation.
    *   **Network Graphs:** Representing relationships and connections between entities.
    *   **Maps:** Displaying geospatial data.
*   **Clarity and Simplicity:** Avoid clutter ("chart junk"). Use clear labels, titles, legends, and annotations. Ensure text is readable.
*   **Accuracy:** Represent the data truthfully. Be mindful of misleading scales or representations.
*   **Aesthetics:** Use color effectively (consider color blindness), maintain consistency, and strive for a clean, professional look.

### `matplotlib`: The Foundation

`matplotlib` is the most established Python plotting library. It provides a low-level interface for creating a wide variety of static, animated, and interactive visualizations. While sometimes verbose, it offers fine-grained control over every aspect of a plot.

Installation (usually included with Anaconda):

```python
# !pip install matplotlib
```

Basic Plotting Example:

```python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd # Assuming df from section 2.2 exists

# Ensure plots appear inline in Jupyter
# %matplotlib inline 
# (Often not needed in modern Jupyter versions, but good practice)

# --- Line Plot ---
# Example: Simulate website visits over time
dates = pd.date_range(start="2025-04-01", periods=10, freq="D")
visits = np.random.randint(50, 200, size=10)

plt.figure(figsize=(10, 4)) # Set figure size
plt.plot(dates, visits, marker="o", linestyle="-", color="b")
plt.title("Website Visits Over Time")
plt.xlabel("Date")
plt.ylabel("Number of Visits")
plt.grid(True)
plt.xticks(rotation=45)
plt.tight_layout() # Adjust layout
plt.show() # Display the plot

# --- Bar Chart ---
# Example: Visits per site type (using df from 2.2, assuming cleaned)
try:
    # Reload or ensure df_filled exists and is cleaned
    # df_filled = pd.read_csv("cleaned_findings.csv") # Example reload
    # If df_filled doesn't exist, recreate a sample for plotting
    if 'df_filled' not in locals():
         data_filled = {
            "URL": ["site-a.com", "site-b.org", "site-c.net", "site-a.com"],
            "Type": ["Blog", "Forum", "News", "Blog"],
            "Visits": [150, 80, 220, 165],
            "Date": pd.to_datetime(["2025-04-28", "2025-04-28", "2025-04-27", "2025-04-29"])
         }
         df_filled = pd.DataFrame(data_filled)
         df_filled["Type"].fillna("Unknown", inplace=True)
         df_filled["Visits"].fillna(df_filled["Visits"].mean(), inplace=True)
         df_filled["Visits"] = df_filled["Visits"].astype(int)

    type_counts = df_filled["Type"].value_counts()

    plt.figure(figsize=(8, 5))
    plt.bar(type_counts.index, type_counts.values, color=["skyblue", "lightcoral", "lightgreen"])
    plt.title("Number of Sites per Type")
    plt.xlabel("Site Type")
    plt.ylabel("Count")
    plt.show()
except Exception as e:
    print(f"Error plotting bar chart (ensure df_filled exists and is clean): {e}")

# --- Scatter Plot ---
# Example: Relationship between two numerical variables (synthetic data)
num_points = 50
x_data = np.random.rand(num_points) * 100
y_data = 2.5 * x_data + np.random.randn(num_points) * 20 + 10 # Linear relationship with noise

plt.figure(figsize=(7, 7))
plt.scatter(x_data, y_data, alpha=0.7, edgecolors="k")
plt.title("Scatter Plot Example")
plt.xlabel("Variable X")
plt.ylabel("Variable Y")
plt.grid(True)
plt.show()

# --- Histogram ---
# Example: Distribution of visits
try:
    plt.figure(figsize=(8, 4))
    plt.hist(df_filled["Visits"], bins=5, color="purple", edgecolor="black")
    plt.title("Distribution of Website Visits")
    plt.xlabel("Number of Visits")
    plt.ylabel("Frequency")
    plt.show()
except Exception as e:
     print(f"Error plotting histogram (ensure df_filled exists): {e}")
```

`matplotlib` offers extensive customization options for colors, markers, labels, legends, subplots, and more. Refer to its documentation for details.

### `seaborn`: Statistical Visualization Made Easy

`seaborn` is built on top of `matplotlib` and provides a higher-level interface for drawing attractive and informative statistical graphics. It integrates seamlessly with `pandas` DataFrames and simplifies the creation of complex plots.

Installation:

```python
# !pip install seaborn
```

Example Usage:

```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd # Assuming df_filled exists

# Set seaborn style
sns.set_theme(style="whitegrid")

# --- Box Plot (Comparing distributions) ---
try:
    plt.figure(figsize=(8, 5))
    sns.boxplot(x="Type", y="Visits", data=df_filled)
    plt.title("Distribution of Visits per Site Type")
    plt.show()
except Exception as e:
    print(f"Error plotting boxplot (ensure df_filled exists): {e}")

# --- Violin Plot (Similar to box plot, shows density) ---
try:
    plt.figure(figsize=(8, 5))
    sns.violinplot(x="Type", y="Visits", data=df_filled)
    plt.title("Distribution Density of Visits per Site Type")
    plt.show()
except Exception as e:
    print(f"Error plotting violinplot (ensure df_filled exists): {e}")

# --- Heatmap (Visualizing correlations or matrices) ---
# Create a sample correlation matrix (replace with real data)
np.random.seed(42)
data_corr = pd.DataFrame(np.random.rand(5, 5), columns=[f"Var{i}" for i in range(1, 6)])
corr_matrix = data_corr.corr()

plt.figure(figsize=(7, 6))
sns.heatmap(corr_matrix, annot=True, cmap="coolwarm", fmt=".2f")
plt.title("Correlation Matrix Heatmap")
plt.show()

# --- Pair Plot (Scatter plots for pairs of variables, histograms on diagonal) ---
# Use a sample dataset from seaborn
tips = sns.load_dataset("tips")

sns.pairplot(tips, hue="sex") # Color points by gender
plt.suptitle("Pair Plot of Tips Dataset", y=1.02) # Add title above plots
plt.show()
```

`seaborn` simplifies creating visually appealing plots for common statistical analysis tasks.

### Interactive Visualizations: `plotly` and `bokeh`

Static plots are useful, but interactive visualizations allow users to explore data dynamically – zooming, panning, hovering for details, filtering. `plotly` and `bokeh` are two popular libraries for creating interactive plots in Jupyter.

**`plotly`**

`plotly` creates rich, interactive charts. It has a high-level interface (`plotly.express`) for quick plotting and a lower-level graph objects interface for more control.

Installation:

```python
# !pip install plotly
```

Example using `plotly.express`:

```python
import plotly.express as px
import pandas as pd # Assuming df_filled exists

# --- Interactive Scatter Plot ---
try:
    fig_scatter = px.scatter(df_filled, x="Date", y="Visits", color="Type", 
                           hover_data=["URL"], title="Website Visits Over Time (Interactive)")
    fig_scatter.show()
except Exception as e:
    print(f"Error plotting interactive scatter (ensure df_filled exists): {e}")

# --- Interactive Bar Chart ---
try:
    type_counts = df_filled["Type"].value_counts().reset_index()
    type_counts.columns = ["Type", "Count"]
    fig_bar = px.bar(type_counts, x="Type", y="Count", color="Type",
                     title="Number of Sites per Type (Interactive)")
    fig_bar.show()
except Exception as e:
    print(f"Error plotting interactive bar (ensure df_filled exists): {e}")
```

**`bokeh`**

`bokeh` is another powerful library focused on creating interactive visualizations for modern web browsers. It can produce elegant and versatile plots.

Installation:

```python
# !pip install bokeh
```

Example Usage:

```python
from bokeh.plotting import figure, show, output_notebook
from bokeh.models import ColumnDataSource, HoverTool
import pandas as pd # Assuming df_filled exists

# Ensure Bokeh plots display in the notebook
output_notebook()

try:
    # Prepare data source
    source = ColumnDataSource(df_filled)

    # Define tooltips for hover
tooltips = [
        ("URL", "@URL"),
        ("Date", "@Date{%F}"), # Format date
        ("Visits", "@Visits"),
        ("Type", "@Type"),
    ]

    # Create figure
p = figure(height=300, width=600, title="Website Visits (Bokeh)", 
             x_axis_label="Date", y_axis_label="Visits", 
             x_axis_type="datetime", tooltips=tooltips)

    # Add scatter glyphs
p.scatter(x="Date", y="Visits", source=source, legend_field="Type", 
              fill_alpha=0.6, size=10, 
              color=factor_cmap('Type', palette=Category10[len(df_filled['Type'].unique())], factors=df_filled['Type'].unique()))

    p.legend.location = "top_left"
    p.legend.click_policy="hide" # Allow hiding types by clicking legend

    show(p)
except Exception as e:
    print(f"Error plotting Bokeh scatter (ensure df_filled exists): {e}")
```

Interactive plots are excellent for exploratory analysis and creating engaging reports or dashboards.

### Network Visualization with `NetworkX`

OSINT often involves understanding relationships – between people, organizations, websites, infrastructure, etc. `NetworkX` is the standard Python library for creating, manipulating, and studying the structure, dynamics, and functions of complex networks.

Installation:

```python
# !pip install networkx
```

Example: Visualizing a simple social network

```python
import networkx as nx
import matplotlib.pyplot as plt

# Create an empty graph
G = nx.Graph()

# Add nodes (e.g., individuals or entities)
G.add_node("Alice")
G.add_node("Bob")
G.add_node("Charlie")
G.add_node("David")
G.add_node("Eve")

# Add edges (relationships)
G.add_edge("Alice", "Bob")
G.add_edge("Alice", "Charlie")
G.add_edge("Bob", "Charlie")
G.add_edge("Bob", "David")
G.add_edge("Charlie", "David")
G.add_edge("David", "Eve")

print(f"Nodes: {G.nodes()}")
print(f"Edges: {G.edges()}")
print(f"Number of nodes: {G.number_of_nodes()}")
print(f"Number of edges: {G.number_of_edges()}")

# Basic visualization using matplotlib
plt.figure(figsize=(8, 6))
pos = nx.spring_layout(G, seed=42) # Positioning algorithm
nx.draw(G, pos, with_labels=True, node_color="lightblue", 
        node_size=2000, edge_color="gray", font_size=12, font_weight="bold")
plt.title("Simple Network Graph")
plt.show()

# Basic analysis
print(f"\nDegree (number of connections) for each node:")
print(G.degree())

print(f"\nNodes connected to Alice: {list(G.neighbors(\"Alice\"))}")
```

`NetworkX` can handle large graphs and offers many algorithms for analysis (e.g., finding shortest paths, centrality measures, community detection), which are invaluable for analyzing complex OSINT data.

### Geospatial Visualization: `folium` and `geopandas`

Mapping is essential for visualizing location-based OSINT data (e.g., IP address locations, photo geotags, event locations).

**`folium`**

`folium` builds on the data wrangling strengths of Python and the mapping strengths of the `Leaflet.js` library. It makes it easy to visualize data on an interactive Leaflet map.

Installation:

```python
# !pip install folium
```

Example:

```python
import folium
import pandas as pd

# Sample location data (replace with real geocoded data)
location_data = {
    "Name": ["Location A", "Location B", "Location C"],
    "Latitude": [34.0522, 40.7128, 51.5074],
    "Longitude": [-118.2437, -74.0060, -0.1278],
    "Info": ["City Hall, LA", "City Hall, NYC", "Trafalgar Square, London"]
}
df_locations = pd.DataFrame(location_data)

# Create a map centered around an average location
map_center = [df_locations["Latitude"].mean(), df_locations["Longitude"].mean()]
m = folium.Map(location=map_center, zoom_start=2)

# Add markers for each location
for idx, row in df_locations.iterrows():
    folium.Marker(
        location=[row["Latitude"], row["Longitude"]],
        popup=f"<b>{row['Name']}</b><br>{row['Info']}", # HTML in popup
        tooltip=row["Name"], # Text on hover
        icon=folium.Icon(color="blue", icon="info-sign")
    ).add_to(m)

# Display the map (will render inline in Jupyter)
# m

# Save map to an HTML file
m.save("locations_map.html")
print("Map saved to locations_map.html")
```

**`geopandas` (More Advanced)**

`geopandas` extends `pandas` to allow spatial operations on geometric types. It depends on several geospatial libraries (`shapely`, `fiona`, `pyproj`) which can sometimes be tricky to install. It's powerful for spatial analysis and creating static maps (often using `matplotlib` underneath) or choropleth maps (where regions are colored based on data values).

Installation (often easier with `conda`):

```bash
# conda install geopandas
# or
# pip install geopandas
```

Conceptual Example (Choropleth):

```python
# import geopandas as gpd
# import matplotlib.pyplot as plt

# # Load a shapefile with geometries (e.g., countries, states)
# # world = gpd.read_file(gpd.datasets.get_path('naturalearth_lowres'))

# # Sample data to map (e.g., OSINT event counts per country)
# # data_to_map = pd.DataFrame({
# #     'iso_a2': ['US', 'GB', 'DE', 'FR', 'JP'], # Country codes
# #     'event_count': [150, 80, 120, 70, 90]
# # })

# # Merge data with geometries
# # world_merged = world.merge(data_to_map, left_on='iso_a2', right_on='iso_a2', how='left')
# # world_merged['event_count'] = world_merged['event_count'].fillna(0) # Fill missing countries

# # Plot choropleth map
# # fig, ax = plt.subplots(1, 1, figsize=(15, 10))
# # world_merged.plot(column='event_count', cmap='viridis', linewidth=0.8, ax=ax, 
# #                   edgecolor='0.8', legend=True,
# #                   legend_kwds={'label': "Number of OSINT Events",
# #                                'orientation': "horizontal"})
# # ax.set_title("OSINT Events per Country")
# # ax.set_axis_off()
# # plt.show()

print("\nGeoPandas example is conceptual and requires geospatial data.")
```

Choosing the right visualization tool depends on the data type, the insights you want to convey, and whether interactivity is needed.

### Assignment 4: Visualizing OSINT Insights

**Objective:** Create various visualizations to communicate findings from the cleaned and analyzed data from Assignment 3.

**Instructions:**
1.  Use the cleaned DataFrame from Assignment 3.
2.  Create at least **four** different types of visualizations using `matplotlib`, `seaborn`, `plotly`, or `bokeh`. Choose chart types appropriate for the data and the insights you want to highlight.
3.  Potential visualizations (choose based on your data):
    *   A bar chart showing counts or sums by category.
    *   A line chart showing trends over time (if applicable).
    *   A scatter plot exploring the relationship between two numerical variables.
    *   A histogram or density plot showing the distribution of a numerical variable.
    *   A box plot or violin plot comparing distributions across categories.
    *   A heatmap (if you have matrix data or can calculate correlations).
    *   An interactive plot using `plotly` or `bokeh`.
    *   A basic network graph using `NetworkX` (if your data represents relationships).
    *   A map using `folium` (if your data includes geographic coordinates).
4.  For each visualization:
    *   Ensure it has a clear title, axis labels, and a legend if necessary.
    *   Customize the appearance (colors, styles) for clarity and impact.
    *   Add a Markdown cell explaining what the visualization shows and interpreting the key insights derived from it.
5.  Organize your visualizations logically within the Jupyter Notebook.

**Deliverables:**
- A Jupyter Notebook (.ipynb file) containing the code for generating the visualizations and the accompanying interpretations.

**Evaluation Criteria:**
- Creation of at least four distinct and appropriate visualizations.
- Correct use of plotting libraries (`matplotlib`, `seaborn`, `plotly`, `bokeh`, `NetworkX`, `folium`).
- Clarity and correctness of plots (titles, labels, legends).
- Effective communication of insights through visualization.
- Quality of interpretation and explanation provided for each plot.
