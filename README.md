import requests
from bs4 import BeautifulSoup
import csv
import json
import os

# Web Scraper Class
class WebScraper:
    def __init__(self, url):
        self.url = url

    def fetch_html(self):
        try:
            response = requests.get(self.url)
            response.raise_for_status()
            return response.text
        except requests.exceptions.RequestException as e:
            print(f"Error fetching data: {e}")
            return None

    def parse_html(self, html):
        soup = BeautifulSoup(html, 'html.parser')
        return soup

    def scrape_data(self):
        html = self.fetch_html()
        if html:
            soup = self.parse_html(html)
            data = []
            for element in soup.find_all('h2'):  # Adjust tag and class as needed
                headline = element.get_text(strip=True)
                link = element.find('a')['href'] if element.find('a') else None
                data.append({"headline": headline, "link": link})
            return data
        return []

    def save_to_csv(self, data, filename):
        with open(filename, mode='w', newline='', encoding='utf-8') as file:
            writer = csv.DictWriter(file, fieldnames=["headline", "link"])
            writer.writeheader()
            writer.writerows(data)

    def save_to_json(self, data, filename):
        with open(filename, mode='w', encoding='utf-8') as file:
            json.dump(data, file, indent=4)

# Console-Based Calculator Class
class Calculator:
    def add(self, a, b):
        return a + b

    def subtract(self, a, b):
        return a - b

    def multiply(self, a, b):
        return a * b

    def divide(self, a, b):
        try:
            return a / b
        except ZeroDivisionError:
            return "Error: Division by zero is not allowed."

# Logger
class Logger:
    def __init__(self, filename="operations.log"):
        self.filename = filename

    def log(self, operation, result):
        with open(self.filename, mode='a') as file:
            file.write(f"{operation} = {result}\n")

# Main Application
if __name__ == "__main__":
    print("Select Mode: 1. Web Scraper 2. Calculator")
    choice = input("Enter choice (1 or 2): ")

    if choice == "1":
        url = input("Enter the website URL to scrape: ")
        scraper = WebScraper(url)
        scraped_data = scraper.scrape_data()

        if scraped_data:
            print("Data scraped successfully. Choose output format:")
            print("1. CSV\n2. JSON")
            format_choice = input("Enter choice: ")

            if format_choice == "1":
                scraper.save_to_csv(scraped_data, "scraped_data.csv")
                print("Data saved to scraped_data.csv")
            elif format_choice == "2":
                scraper.save_to_json(scraped_data, "scraped_data.json")
                print("Data saved to scraped_data.json")
            else:
                print("Invalid choice.")
        else:
            print("No data scraped.")

    elif choice == "2":
        calculator = Calculator()
        logger = Logger()

        while True:
            print("\nCalculator Menu:")
            print("1. Add\n2. Subtract\n3. Multiply\n4. Divide\n5. Exit")
            calc_choice = input("Enter choice: ")

            if calc_choice == "5":
                print("Exiting Calculator.")
                break

            try:
                num1 = float(input("Enter first number: "))
                num2 = float(input("Enter second number: "))

                if calc_choice == "1":
                    result = calculator.add(num1, num2)
                    operation = f"{num1} + {num2}"
                elif calc_choice == "2":
                    result = calculator.subtract(num1, num2)
                    operation = f"{num1} - {num2}"
                elif calc_choice == "3":
                    result = calculator.multiply(num1, num2)
                    operation = f"{num1} * {num2}"
                elif calc_choice == "4":
                    result = calculator.divide(num1, num2)
                    operation = f"{num1} / {num2}"
                else:
                    print("Invalid choice.")
                    continue

                print(f"Result: {result}")
                logger.log(operation, result)

            except ValueError:
                print("Invalid input. Please enter numeric values.")

    else:
        print("Invalid choice. Exiting application.")
