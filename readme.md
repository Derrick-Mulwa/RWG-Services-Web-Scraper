RWG Services Web Scraper
Overview
The RWG Services Web Scraper is a Python-based automation tool designed to extract data from the RWG Services Object Status page (https://rwgservices.rwg.nl/Status/ObjectStatus). This tool leverages SeleniumBase for browser automation, handles CAPTCHA challenges, and collects tabular data into a consolidated CSV file. It is built to navigate the website, interact with UI elements, and scrape paginated data efficiently while managing proxies and CAPTCHA resolution.
The scraper is designed for users who need to extract structured data from the RWG Services website, such as logistics professionals, data analysts, or system administrators monitoring object statuses. It ensures robust handling of dynamic web elements, CAPTCHAs, and pagination, with output saved in a timestamped CSV file for easy analysis.
Features
Automated Browser Navigation: Utilizes SeleniumBase to interact with the RWG Services website, including clicking buttons, selecting dropdowns, and handling CAPTCHAs.

Proxy Support: Reads proxy configurations from a proxy.txt file to enable scraping through proxies, enhancing anonymity and bypassing potential restrictions.

CAPTCHA Handling: Automatically detects and resolves CAPTCHAs using SeleniumBase's built-in CAPTCHA-solving capabilities, with fallback to manual mouse clicks via PyAutoGUI.

Dynamic Pagination: Scrapes data across multiple pages by navigating through pagination controls and concatenating results into a single DataFrame.

Data Extraction: Extracts tabular data from the website and saves it as a CSV file with a timestamp for versioning.

Error Handling: Implements retry mechanisms for CAPTCHA resolution, page loading, and element interaction to ensure robust operation.

Configurable Delays: Includes customizable sleep times (sleep_time and reconnect_time) to handle network latency and website responsiveness.

Workflow
The scraper follows a structured workflow to ensure reliable data extraction:
Initialization:
Reads proxy settings from proxy.txt.

Initializes a SeleniumBase Driver instance with optional proxy configuration and headless mode disabled (uc=True for undetectable Chrome).

Opens the target URL: https://rwgservices.rwg.nl/Status/ObjectStatus.

python

with open('proxy.txt', 'r') as file:
    proxies_list = file.read().split('\n')
proxy = proxies_list[0]
if proxy == '':
    driver = Driver(uc=True)
else:
    driver = Driver(uc=True, proxy=proxy)
driver.implicitly_wait(30)
url = "https://rwgservices.rwg.nl/Status/ObjectStatus"
driver.maximize_window()
driver.uc_open_with_reconnect(url, reconnect_time=reconnect_time)

Cookie Handling:
Detects and dismisses the cookie consent popup to access the main content.

Retries if the popup is not immediately available, incorporating CAPTCHA resolution if needed.

python

dismiss_cookie_button = (By.CSS_SELECTOR, 'a[aria-label="dismiss cookie message"]')
driver.find_element(*dismiss_cookie_button).click()

CAPTCHA Resolution:
Attempts to solve CAPTCHAs using SeleniumBase's uc_gui_click_captcha() method.

Falls back to a simulated mouse click via PyAutoGUI if necessary.

Loops until the CAPTCHA is resolved or the page is fully accessible.

python

try:
    driver.uc_gui_click_captcha()
    time.sleep(sleep_time)
    pyautogui.leftClick()
except:
    pass

Data Table Configuration:
Clicks the "Go" button to load the data table.

Sets the table to display 100 rows per page using the dropdown menu for efficient scraping.

python

go_button = (By.CSS_SELECTOR, 'button[id="gobutton"]')
driver.find_element(*go_button).click()
dropdown = driver.find_element(By.CSS_SELECTOR, 'select[name="barelist_length"]')
Select(dropdown).select_by_value("100")

Paginated Data Extraction:
Extracts the HTML of the data table on each page and converts it to a Pandas DataFrame using pd.read_html.

Concatenates DataFrames from all pages into a single main_df.

Navigates to the next page until the "Next" button is disabled, indicating the end of pagination.

python

table = driver.find_element(By.CSS_SELECTOR, 'table[id="barelist"')
table_data = StringIO(table.get_attribute("outerHTML"))
df = pd.read_html(table_data)[0]
main_df = pd.concat([main_df, df])
driver.find_element(By.CSS_SELECTOR, 'li[id="barelist_next"]').click()

Data Storage:
Saves the consolidated DataFrame to a CSV file with a timestamp in the format data DD-MM-YYYY HH;MM.csv.

Closes the browser and outputs a success message.

python

main_df.to_csv(f'data {datetime.now().strftime("%d-%m-%Y %H;%M")}.csv')
driver.quit()
print(f"Program ran successfully")

Benefits to Users
Automation: Eliminates manual data collection, saving time and reducing errors.

Scalability: Handles large datasets by navigating through paginated tables and consolidating results.

Reliability: Robust error handling ensures the scraper can recover from CAPTCHAs, network issues, or missing elements.

Flexibility: Proxy support allows users to bypass IP-based restrictions or enhance privacy.

Data Accessibility: Outputs data in a structured CSV format, compatible with data analysis tools like Excel, Python, or R.

Versioning: Timestamped output files enable tracking of data snapshots over time.

Prerequisites
Before running the scraper, ensure the following are installed and configured:
Python 3.8+: The programming language used for the script.

SeleniumBase: For browser automation and CAPTCHA handling.

Pandas: For data manipulation and CSV export.

PyAutoGUI: For simulating mouse clicks during CAPTCHA resolution.

Chrome Browser: Required by SeleniumBase for web automation.

Proxy Configuration (optional): A proxy.txt file containing a proxy address (if applicable).

Installation
Clone the Repository:
bash

git clone <repository-url>
cd <repository-directory>

Set Up a Virtual Environment (optional but recommended):
bash

python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

Install Dependencies:
Create a requirements.txt file with the following content:

seleniumbase
pandas
pyautogui

Then run:
bash

pip install -r requirements.txt

Configure Proxy (optional):
Create a proxy.txt file in the project root.

Add a single proxy address (e.g., http://username:password@host:port) or leave it empty for no proxy.

Run the Scraper:
bash

python scraper.py

The script will execute, scrape the data, and save it to a timestamped CSV file in the project directory.

Usage
Ensure the proxy.txt file is configured (or empty if no proxy is needed).

Run the script using the command above.

The scraper will:
Open the RWG Services website.

Handle cookies and CAPTCHAs.

Extract paginated table data.

Save the data to a CSV file (e.g., data 03-06-2025 00;10.csv).

Check the console for the "Program ran successfully" message to confirm completion.

Notes
Sleep Times: Adjust sleep_time (default: 30 seconds) and reconnect_time (default: 5 seconds) in the script to match your network speed or website responsiveness.

CAPTCHA Challenges: The script attempts automated CAPTCHA resolution but may require manual intervention via PyAutoGUI clicks in some cases.

Output: The CSV file is saved in the project directory with a timestamp for easy identification.

Error Handling: The script retries failed actions (e.g., CAPTCHA resolution, page navigation) to ensure robustness, but persistent issues may require manual debugging.

License
This project is licensed under the MIT License. See the LICENSE file for details.
Contributing
Contributions are welcome! Please submit a pull request or open an issue to discuss improvements or bug fixes.
Contact
For questions or support, contact the project maintainer at <your-email> or open an issue in the repository.

Explain proxy handling

Web scraping ethics

More concise prerequisites

