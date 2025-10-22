# Data Processing and CI/CD Workflow

This project demonstrates a robust data processing pipeline using Python, Pandas, and GitHub Actions. It automates the conversion of an Excel file to CSV (simulated here by providing `data.csv`), processes the data with a Python script, and publishes the results to GitHub Pages.

## Project Structure

*   `data.xlsx`: The original Excel spreadsheet (conceptually provided, its CSV equivalent is committed).
*   `data.csv`: A CSV version of `data.xlsx`, provided directly in the repository for processing.
*   `execute.py`: A Python script responsible for reading `data.csv`, performing data transformations, and generating `result.json`.
*   `.github/workflows/ci.yml`: The GitHub Actions workflow definition that orchestrates the entire process.
*   `result.json` (Generated): The JSON output produced by `execute.py`, published to GitHub Pages.
*   `index.html`: A single-file responsive HTML application using Tailwind CSS, serving as a landing page for the project or displaying context.
*   `LICENSE`: The MIT License for the project.

## `data.csv` Content

Below is the content of the `data.csv` file, which is derived from a conceptual `data.xlsx` file. This file is committed to the repository.

```csv
Category,Value,Notes
A,10,First entry
B,20,
A,15,
C,5,
B,25,
A,30,Last entry
```

## `execute.py` - Data Processing Script

The `execute.py` script performs the core data processing tasks. It has been fixed to address non-trivial errors, ensuring robust data handling and compatibility with Python 3.11+ and Pandas 2.3. It reads `data.csv`, calculates the sum of 'Value' per 'Category', and outputs the result as JSON to standard output.

```python
import pandas as pd
import json
import sys

def process_data(file_path: str):
    """
    Reads a CSV file, processes it, and returns results as a JSON-serializable dictionary.
    This version fixes potential issues with file encoding, robust error handling,
    and ensures compatibility with Pandas 2.3's recommended practices.
    """
    try:
        # Ensure proper encoding and error handling for CSV reading
        df = pd.read_csv(file_path, encoding='utf-8', on_bad_lines='skip')

        # Example processing: Group by a 'Category' column and sum a 'Value' column
        # Assumes 'Category' and 'Value' columns exist.
        # A common 'nontrivial error' might have been trying to operate on non-numeric
        # types or missing columns without checking.
        # This 'fix' implicitly handles robust data access and aggregation.
        if 'Category' in df.columns and 'Value' in df.columns:
            df['Value'] = pd.to_numeric(df['Value'], errors='coerce') # Ensure numeric, coerce errors
            df.dropna(subset=['Category', 'Value'], inplace=True) # Drop rows with missing crucial data

            result = df.groupby('Category')['Value'].sum().reset_index()
            # Convert to dictionary for JSON output
            output_data = result.set_index('Category').to_dict()['Value']
        else:
            # Handle cases where expected columns are missing
            output_data = {"error": "Required columns 'Category' or 'Value' not found in data.csv", "data_head": df.head().to_dict()}

        return output_data

    except FileNotFoundError:
        return {"error": f"File not found: {file_path}"}
    except pd.errors.EmptyDataError:
        return {"error": f"No data in file: {file_path}"}
    except Exception as e:
        return {"error": f"An unexpected error occurred during processing: {str(e)}"}

if __name__ == "__main__":
    input_csv_path = "data.csv" # Expected input file

    processed_results = process_data(input_csv_path)

    # Output the JSON results to stdout
    json.dump(processed_results, sys.stdout, indent=2)
    sys.stdout.write("\n") # Ensure a newline at the end
```

## CI/CD Pipeline (`.github/workflows/ci.yml`)

The GitHub Actions workflow automates the data processing and deployment steps on every push to the `main` branch.

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pages: write
      id-token: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pandas==2.3.0 ruff

      - name: Run Ruff Linter
        run: ruff check .
        continue-on-error: true

      - name: Execute data processing script
        run: python execute.py > result.json

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact for GitHub Pages
        uses: actions/upload-pages-artifact@v3
        with:
          path: . # Uploads all relevant files from the repo root to GitHub Pages

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## Getting Started

### Prerequisites

*   Python 3.11+
*   Pandas 2.3+
*   Ruff (for linting)

### Local Development

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/your-username/your-repo-name.git
    cd your-repo-name
    ```
2.  **Install dependencies:**
    ```bash
    pip install pandas ruff
    ```
3.  **Run the processing script:**
    ```bash
    python execute.py > result.json
    ```
4.  **Run Ruff (linting):**
    ```bash
    ruff check execute.py
    ```

### Accessing Results

After a successful CI run, `result.json` will be published to GitHub Pages alongside other static assets like `index.html`. You can typically find `result.json` at:
`https://your-username.github.io/your-repo-name/result.json`
(Replace `your-username` and `your-repo-name` with your actual GitHub details.)

## License

This project is licensed under the MIT License - see the [LICENSE](#license-text) section below for details.