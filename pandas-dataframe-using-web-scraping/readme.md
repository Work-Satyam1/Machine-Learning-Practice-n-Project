# AmbitionBox Company Data Scraper

A Python web scraping project that collects company information from AmbitionBox company listing pages using Requests, BeautifulSoup, NumPy, and Pandas.

## Project Overview

The goal of this project is to practice:

- Web scraping with Python
- Sending HTTP requests
- Using HTTP headers and User-Agent
- Parsing HTML using BeautifulSoup
- Finding HTML elements using tags and classes
- Extracting structured information from web pages
- Handling missing data
- Working with Pandas DataFrames
- Scraping multiple pages using pagination
- Combining data from multiple pages
- Exporting scraped data to CSV

## Tech Stack

- Python
- Requests
- BeautifulSoup
- lxml
- Pandas
- NumPy
- Jupyter Notebook / VS Code

## Libraries Installation

```bash
pip install requests beautifulsoup4 lxml pandas numpy
```

## Imports

```python
import requests
import numpy as np
import pandas as pd
from bs4 import BeautifulSoup
```

## Website

The project scrapes the AmbitionBox company listing pages:

```text
https://www.ambitionbox.com/list-of-companies?page=1
```

The website uses the `page` parameter for pagination:

```text
?page=1
?page=2
?page=3
...
```

## User-Agent Header

The scraper uses a browser-like User-Agent in the request headers:

```python
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36'
}
```

The header is passed with the request:

```python
response = requests.get(
    url,
    headers=headers
)
```

### Why is the User-Agent important?

When a request is sent using Python, the server can identify it as an automated request.

Without a suitable User-Agent, the website may return an `Access Denied` page instead of the actual webpage.

Using a browser-like User-Agent helps the request identify itself similarly to a normal browser request.

A User-Agent does not guarantee access because websites may also use rate limiting, CAPTCHA, cookies, JavaScript challenges, IP restrictions, or other anti-bot mechanisms.

## Scraping Workflow

```text
Send HTTP Request
       ↓
Receive HTML
       ↓
Parse HTML with BeautifulSoup
       ↓
Find Company Cards
       ↓
Extract Company Information
       ↓
Create Page DataFrame
       ↓
Concatenate DataFrames
       ↓
Final Dataset
       ↓
Export CSV
```

## Finding Company Cards

Company cards are identified using:

```python
company = soup.find_all(
    'div',
    class_='companyCardWrapper'
)
```

Each element represents one company card.

## Extracted Data

The scraper extracts:

```python
Name = []
Rating = []
Review = []
Salaries = []
Interviews = []
Jobs = []
Benefits = []
Photos = []
```

### Company Name

```python
Name.append(
    i.find('h2').get_text(strip=True)
)
```

### Rating

```python
Rating.append(
    i.find(
        'div',
        class_='rating_star_container'
    ).get_text(strip=True)
)
```

### Reviews

```python
Review.append(
    i.find(
        'span',
        class_='companyCardWrapper__companyRatingCount'
    ).get_text(strip=True)
)
```

## Salary, Interview, Jobs, Benefits and Photos

These values are located inside:

```python
info = i.find(
    'div',
    class_='companyCardWrapper__tertiaryInformation'
)
```

The `<span>` elements are extracted:

```python
items = info.find_all('span')
```

The current HTML structure uses:

```python
Salaries.append(items[2].get_text(strip=True))
Interviews.append(items[4].get_text(strip=True))
Jobs.append(items[6].get_text(strip=True))
Benefits.append(items[8].get_text(strip=True))
Photos.append(items[10].get_text(strip=True))
```

### Current Index Mapping

| Index | Field |
|---:|---|
| `items[2]` | Salaries |
| `items[4]` | Interviews |
| `items[6]` | Jobs |
| `items[8]` | Benefits |
| `items[10]` | Photos |

These indexes depend on the current HTML structure of the website.

If the HTML structure changes, inspect the spans again:

```python
for index, item in enumerate(items):
    print(index, item.get_text(strip=True))
```

## Error Handling

`try-except` is used so that missing information does not stop the scraper.

Example:

```python
try:
    Name.append(
        i.find('h2').get_text(strip=True)
    )
except:
    Name.append(np.nan)
```

If a value is unavailable, `np.nan` is stored.

## Pagination

The scraper processes pages from 1 to 1000:

```python
for page_no in range(1, 1001):
```

The URL is generated dynamically:

```python
url = f'https://www.ambitionbox.com/list-of-companies?page={page_no}'
```

## Creating the DataFrame

After scraping one page:

```python
page_data = pd.DataFrame({
    'Name': Name,
    'Rating': Rating,
    'Review': Review,
    'Salaries': Salaries,
    'Interviews': Interviews,
    'Jobs': Jobs,
    'Benefits': Benefits,
    'Photos': Photos
})
```

## Combining All Pages

An empty DataFrame is created:

```python
all_companies = pd.DataFrame()
```

Data from every page is combined using:

```python
all_companies = pd.concat(
    [all_companies, page_data],
    ignore_index=True
)
```

`pd.concat()` is used instead of the older `DataFrame.append()` method because `append()` has been removed from modern Pandas versions.

## Complete Scraping Code

```python
import requests
import numpy as np
import pandas as pd
from bs4 import BeautifulSoup


all_companies = pd.DataFrame()

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36'
}


for page_no in range(1, 1001):

    url = f'https://www.ambitionbox.com/list-of-companies?page={page_no}'

    response = requests.get(
        url,
        headers=headers
    )

    soup = BeautifulSoup(
        response.text,
        'lxml'
    )

    company = soup.find_all(
        'div',
        class_='companyCardWrapper'
    )

    Name = []
    Rating = []
    Review = []
    Salaries = []
    Interviews = []
    Jobs = []
    Benefits = []
    Photos = []

    for i in company:

        try:
            Name.append(
                i.find('h2').get_text(strip=True)
            )
        except:
            Name.append(np.nan)

        try:
            Rating.append(
                i.find(
                    'div',
                    class_='rating_star_container'
                ).get_text(strip=True)
            )
        except:
            Rating.append(np.nan)

        try:
            Review.append(
                i.find(
                    'span',
                    class_='companyCardWrapper__companyRatingCount'
                ).get_text(strip=True)
            )
        except:
            Review.append(np.nan)

        try:
            info = i.find(
                'div',
                class_='companyCardWrapper__tertiaryInformation'
            )

            items = info.find_all('span')

            Salaries.append(items[2].get_text(strip=True))
            Interviews.append(items[4].get_text(strip=True))
            Jobs.append(items[6].get_text(strip=True))
            Benefits.append(items[8].get_text(strip=True))
            Photos.append(items[10].get_text(strip=True))

        except:
            Salaries.append(np.nan)
            Interviews.append(np.nan)
            Jobs.append(np.nan)
            Benefits.append(np.nan)
            Photos.append(np.nan)

    page_data = pd.DataFrame({
        'Name': Name,
        'Rating': Rating,
        'Review': Review,
        'Salaries': Salaries,
        'Interviews': Interviews,
        'Jobs': Jobs,
        'Benefits': Benefits,
        'Photos': Photos
    })

    all_companies = pd.concat(
        [all_companies, page_data],
        ignore_index=True
    )
```

## Checking the Dataset

Check the number of rows and columns:

```python
all_companies.shape
```

View the first rows:

```python
all_companies.head()
```

View the last rows:

```python
all_companies.tail()
```

Check column names:

```python
all_companies.columns
```

Check data types:

```python
all_companies.info()
```

Check missing values:

```python
all_companies.isnull().sum()
```

## Dataset Columns

| Column | Description |
|---|---|
| `Name` | Company name |
| `Rating` | Company rating |
| `Review` | Number of company reviews |
| `Salaries` | Number of salary records |
| `Interviews` | Number of interview records |
| `Jobs` | Number of job records |
| `Benefits` | Number of benefits records |
| `Photos` | Number of company photos |

## Example Output

| Name | Rating | Review | Salaries | Interviews | Jobs | Benefits | Photos |
|---|---:|---:|---:|---:|---:|---:|---:|
| TCS | 3.2 | (1.2L) | 10.4L | 11.4k | 4.6k | 11.1k | 100 |
| Accenture | 3.7 | (77.6k) | 7.3L | 9.6k | 14.6k | 7k | 49 |
| Wipro | 3.6 | (68.3k) | 4.9L | 7k | 5k | 123 | 123 |
| Cognizant | 3.7 | (64.5k) | 6.1L | 6.6k | 726 | 5.7k | 105 |
| Capgemini | 3.6 | (56.7k) | 5L | 5.7k | 2k | 3.9k | 54 |

## Export Dataset to CSV

```python
all_companies.to_csv(
    'ambitionbox_companies.csv',
    index=False
)
```

`index=False` prevents Pandas from adding the DataFrame index as an additional column.

## Important Scraping Considerations

### 1. Website HTML Can Change

The scraper depends on HTML class names such as:

```text
companyCardWrapper
rating_star_container
companyCardWrapper__companyRatingCount
companyCardWrapper__tertiaryInformation
```

If the website changes its HTML structure, the selectors may need to be updated.

### 2. Span Indexes Can Change

The current indexes are:

```python
items[2]
items[4]
items[6]
items[8]
items[10]
```

If the extracted values become incorrect, inspect the available spans:

```python
for index, item in enumerate(items):
    print(index, item.get_text(strip=True))
```

### 3. Check HTTP Status Codes

It is useful to check:

```python
print(response.status_code)
```

A successful response generally returns:

```text
200
```

If the website returns an error such as `403` or an Access Denied page, the request may have been blocked.

### 4. Avoid Excessive Requests

Sending a large number of requests quickly can result in:

- Rate limiting
- Temporary blocking
- Failed requests
- Incomplete datasets

Large scraping jobs should be performed responsibly with appropriate delays and without attempting to bypass technical restrictions.

## Future Improvements

- Clean values such as `10.4L` and `11.4k`
- Convert ratings to numeric values
- Convert review counts to numeric values
- Handle HTTP errors
- Add request delays
- Detect blocked pages
- Remove duplicate companies
- Perform Exploratory Data Analysis (EDA)
- Visualize company ratings
- Find companies with the highest number of jobs
- Compare company ratings and salary records
- Analyze review distribution
- Export a cleaned dataset
- Build a dashboard using the scraped data

## Learning Outcomes

This project helped practice:

- HTTP requests
- HTTP headers
- User-Agent
- HTML parsing
- BeautifulSoup
- HTML tags and classes
- Web scraping
- Python loops
- Exception handling
- NumPy
- Pandas
- DataFrames
- Data concatenation
- Pagination
- Missing-value handling
- CSV export
- Basic data collection workflow

## Disclaimer

This project is created for educational and data-analysis purposes.

When scraping websites, respect the website's terms of service, applicable policies, access restrictions, and rate limits. Do not attempt to bypass CAPTCHA, authentication, access controls, or other technical restrictions.
