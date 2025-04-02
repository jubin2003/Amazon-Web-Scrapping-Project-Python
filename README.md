# Amazon-Web-Scrapping-Project-Python
![Project Banner](public/project_thumbnail.jpeg)
## Overview
This script is an Amazon price tracker that scrapes the price, product name, and rating of an item from Amazon. It logs the data into a CSV file and sends an email notification if the price drops below $15.

## Features
- Scrapes product details (title, price, and rating) from an Amazon page.
- Saves data to a CSV file for tracking price trends.
- Sends an email alert when the price drops below a specified amount.
- Runs automatically every 24 hours.

## Prerequisites
Before running the script, make sure you have the following installed:
- Python 3.x
- Required libraries:
  ```sh
  pip install requests beautifulsoup4 smtplib csv datetime
  ```
- A Gmail account with an app-specific password for sending email notifications.

## Setup and Usage

### 1. Update Amazon Product URL
Modify the `URL` variable in the `check_price` function to match the product you want to track:
```python
URL = 'https://www.amazon.com/dp/B07N12FMPX'
```

### 2. Configure Email Alerts
Replace `example@gmail.com` and `'**********'` in the `send_mail` function with your actual Gmail credentials:
```python
server.login('your-email@gmail.com', 'your-app-password')
```
Also, update the recipient email address accordingly.

### 3. Run the Script
Execute the script using:
```sh
python amazon_price_tracker.py
```

## Code Explanation

### Function to Check Price
This function:
- Sends an HTTP request to Amazon to fetch product details.
- Parses the HTML response using BeautifulSoup.
- Extracts the product title, price, and rating.
- Saves this data into a CSV file.
- Triggers an email alert if the price drops below $15.

```python
import requests
from bs4 import BeautifulSoup
import csv
from datetime import datetime
import smtplib
import time

def check_price():
    URL = 'https://www.amazon.com/dp/B07N12FMPX'
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36",
        "Accept-Language": "en-US,en;q=0.9",
    }
    response = requests.get(URL, headers=headers)
    
    if response.status_code == 200:
        soup = BeautifulSoup(response.content, "html.parser")
        title = soup.find("span", id="productTitle")
        product_name = title.get_text(strip=True) if title else "Title not found"
        price_element = soup.find("span", class_="a-offscreen")
        price = price_element.get_text(strip=True)[1:] if price_element else "Price not found"
        rating_element = soup.find("span", class_="a-icon-alt")
        product_rating = rating_element.get_text(strip=True) if rating_element else "Rating not found"
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        
        csv_filename = "AmazonWebScraping.csv"
        data = [timestamp, product_name, price , product_rating]
        with open(csv_filename, 'a+', newline='', encoding='UTF8') as f:
            writer = csv.writer(f)
            writer.writerow(data)
        
        if price != "Price not found" and float(price) < 14.99:
            send_mail()
    else:
        print("Failed to retrieve the webpage. Status code:", response.status_code)
```

### Function to Send Email Alerts
This function:
- Logs into the Gmail SMTP server.
- Sends an email notification with product details if the price drops below $15.
- Closes the SMTP connection after sending.

```python
def send_mail():
    server = smtplib.SMTP_SSL('smtp.gmail.com', 465)
    server.login('example@gmail.com', '***********')
    subject = "The Shirt you want is below $15! Now is your chance to buy!"
    body = "Alex, This is the moment we have been waiting for. Now is your chance to pick up the shirt of your dreams. Don't mess it up! Link here: https://www.amazon.com/Funny-Data-Systems-Business-Analyst/dp/B07FNW9FGJ"
    msg = f"Subject: {subject}\n\n{body}"
    server.sendmail('example@gmail.com', 'example@gmail.com', msg)
    server.quit()
    print("Email has been sent!")
```

### Running the Script Periodically
The script runs in an infinite loop, checking the price every hour:

```python
while True:
    check_price()
    time.sleep(3600)  # Wait for an hour before checking again
```

## Notes
- Make sure to comply with Amazon's Terms of Service when scraping data.
- Use a dedicated email account for sending notifications to avoid security risks.
- Store your credentials securely using environment variables instead of hardcoding them.

## License
This project is open-source and free to use.

