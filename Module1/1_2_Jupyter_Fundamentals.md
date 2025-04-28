## 1.2 Jupyter Notebook Fundamentals

Now that we have a foundational understanding of OSINT, let's dive into our primary tool for this course: Jupyter Notebook. Jupyter provides an interactive, web-based environment where you can combine live code (primarily Python in our case), explanatory text (using Markdown), mathematical equations, visualizations, and other rich media in a single document. This makes it exceptionally well-suited for OSINT work, allowing you to document your investigation process, execute analysis code, visualize results, and present findings all in one place, promoting reproducibility and clear communication.

**Review: Installation, Interface, and Core Concepts**

As covered in the Setup section (or if you're already familiar), Jupyter Notebook is typically installed as part of the Anaconda distribution, which simplifies managing Python environments and packages. Once installed, you launch Jupyter Notebook (or its more modern counterpart, JupyterLab) from your terminal or Anaconda Navigator. This opens a new tab in your web browser displaying the Jupyter Dashboard.

*   **Dashboard:** This is your file browser. You can navigate directories, create new notebooks (`.ipynb` files), text files, folders, and terminals.
*   **Notebook Editor:** Opening or creating a notebook takes you to the editor view. The core components are:
    *   **Cells:** The building blocks of a notebook. They can contain either Code or Markdown.
    *   **Kernel:** The computational engine that executes the code in a notebook. For this course, it will be a Python kernel (specifically, the version installed in your environment, likely Python 3.x).
    *   **Toolbar:** Provides quick access to common actions like saving, adding cells, cutting/copying/pasting cells, running cells, changing cell types, and restarting the kernel.
    *   **Menu Bar:** Offers more comprehensive options for file operations, editing, viewing, inserting cells, managing the kernel, and help resources.

*   **Cell Types:**
    *   **Code Cells:** Contain executable code (Python in our case). You run them by selecting the cell and pressing `Shift + Enter` (runs the cell and selects the next one), `Ctrl + Enter` (runs the cell and keeps it selected), or using the "Run" button in the toolbar. Output (text, plots, errors) appears directly below the cell.
    *   **Markdown Cells:** Contain text formatted using Markdown syntax. Running a Markdown cell renders the formatted text. This is where you'll write your explanations, notes, and reports.
    *   **Raw NBConvert Cells:** Less commonly used; content passes through NBConvert unmodified.

Remember to save your notebooks frequently (`Ctrl + S` or the save icon).

**Markdown for OSINT Reporting**

Markdown is a lightweight markup language with plain-text formatting syntax. It's essential for documenting your OSINT investigations within Jupyter. Here are key formatting elements you'll use:

*   **Headings:** Use `#` symbols (`# H1`, `## H2`, `### H3`, etc.)
*   **Emphasis:** `*italic*` or `_italic_`, `**bold**` or `__bold__`, `***bold italic***` or `___bold italic___`
*   **Lists:**
    *   Unordered: Use `-`, `*`, or `+` followed by a space (e.g., `- Item 1`)
    *   Ordered: Use numbers followed by a period (e.g., `1. First Item`)
*   **Links:** `[Link Text](URL)` (e.g., `[Google](https://www.google.com)`) 
*   **Images:** `![Alt Text](Image URL or Local Path)` (e.g., `![Logo](/path/to/logo.png)`)
*   **Code:**
    *   Inline: Wrap code with backticks (`` `code` ``) (e.g., `print("Hello")`)
    *   Block: Use triple backticks (```) or indent lines with four spaces.
    ```python
    def greet(name):
        print(f"Hello, {name}!")
    ```
*   **Blockquotes:** Use `>` (e.g., `> This is a quote.`)
*   **Horizontal Rules:** Use three or more hyphens, asterisks, or underscores (`---`, `***`, `___`)

Mastering Markdown allows you to create well-structured, readable reports directly alongside your analysis code.

**Python Refresher for OSINT**

While this course isn't a full Python introduction, a solid grasp of the basics is crucial. We'll primarily use Python for data collection, manipulation, analysis, and visualization. Here's a quick recap of essential concepts:

*   **Variables & Data Types:** Assigning values to names (e.g., `url = "https://example.com"`, `count = 10`). Common types: `str` (strings), `int` (integers), `float` (floating-point numbers), `bool` (True/False), `list`, `dict` (dictionary), `tuple`, `set`.
*   **Lists:** Ordered, mutable sequences. `my_list = [1, "a", True]`. Access elements with index (`my_list[0]`), slice (`my_list[1:]`), append (`.append()`), etc.
*   **Dictionaries:** Unordered (in older Python versions) key-value pairs. `my_dict = {"name": "Alice", "age": 30}`. Access values by key (`my_dict["name"]`), add/update pairs, iterate over keys/values/items.
*   **Conditionals:** `if`/`elif`/`else` statements for decision making based on conditions (e.g., `if status_code == 200:`).
*   **Loops:**
    *   `for` loop: Iterate over sequences (lists, strings, dictionaries). `for item in my_list: print(item)`
    *   `while` loop: Repeat as long as a condition is true. `while count < 5: count += 1`
*   **Functions:** Defining reusable blocks of code. `def function_name(parameters): ... return value`. Helps organize code and avoid repetition.
*   **Comments:** Use `#` for single-line comments to explain your code.

We will introduce more advanced Python features and specific libraries as needed throughout the course.

**Working with Libraries**

Python's power comes from its extensive collection of libraries (also called packages or modules). Libraries provide pre-written code for various tasks.

*   **Importing:** You make a library's functionality available using the `import` statement.
    *   `import library_name` (Access functions via `library_name.function()`)
    *   `import library_name as alias` (Use a shorter alias: `alias.function()` - very common, e.g., `import pandas as pd`, `import numpy as np`)
    *   `from library_name import specific_function` (Use `specific_function()` directly)
    *   `from library_name import *` (Imports everything - generally discouraged as it can lead to name conflicts)
*   **Installation:** If a library isn't built-in or part of Anaconda, you install it using `pip` (Python's package installer) or `conda` (Anaconda's package manager) in your terminal: `pip install library_name` or `conda install library_name`.
*   **Common Standard Libraries for OSINT:**
    *   `os`: Interact with the operating system (file paths, directories, environment variables).
    *   `datetime`: Work with dates and times.
    *   `json`: Encode and decode JSON data (very common for APIs).
    *   `re`: Regular expressions for pattern matching in text.
    *   `csv`: Read and write CSV files.

**Basic File Input/Output (I/O)**

OSINT often involves reading data from files or saving findings. Python makes this straightforward.

*   **Reading Text Files:**
    ```python
    # Recommended way using 'with' (automatically handles closing the file)
    try:
        with open("my_data.txt", "r") as f: # "r" for read mode
            content = f.read() # Read entire file
            # or lines = f.readlines() # Read file into a list of lines
            # or for line in f: # Iterate line by line (memory efficient)
            #     print(line.strip()) 
        print("File read successfully.")
    except FileNotFoundError:
        print("Error: File not found.")
    except Exception as e:
        print(f"An error occurred: {e}")
    ```
*   **Writing Text Files:**
    ```python
    data_to_write = "This is the information to save.\nAnother line."
    try:
        with open("output.txt", "w") as f: # "w" for write mode (overwrites existing file)
            # Use "a" for append mode (adds to the end of the file)
            f.write(data_to_write)
        print("File written successfully.")
    except Exception as e:
        print(f"An error occurred: {e}")
    ```
*   **Working with CSV Files:** The `csv` module helps handle comma-separated values.
    ```python
    import csv

    # Writing to CSV
    header = ["Name", "URL", "Timestamp"]
    data_rows = [
        ["Site A", "https://site-a.com", "2025-04-28 14:00:00"],
        ["Site B", "https://site-b.org", "2025-04-28 14:05:00"]
    ]
    try:
        with open("findings.csv", "w", newline="", encoding="utf-8") as f:
            writer = csv.writer(f)
            writer.writerow(header)
            writer.writerows(data_rows)
        print("CSV written successfully.")
    except Exception as e:
        print(f"An error occurred writing CSV: {e}")

    # Reading from CSV
    try:
        with open("findings.csv", "r", newline="", encoding="utf-8") as f:
            reader = csv.reader(f)
            header = next(reader) # Read the header row
            print(f"Header: {header}")
            for row in reader:
                print(f"Data Row: {row}")
        print("CSV read successfully.")
    except FileNotFoundError:
        print("Error: CSV file not found.")
    except Exception as e:
        print(f"An error occurred reading CSV: {e}")
    ```
    *(Note: While the `csv` module is useful, we will heavily rely on the `pandas` library in later modules for more powerful CSV and data manipulation.)*

This grounding in Jupyter and Python basics prepares you for the next step: applying these tools to initial OSINT collection tasks.
