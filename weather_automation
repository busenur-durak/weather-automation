from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager
import csv
import json
import schedule
import time
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from datetime import datetime, timedelta
import matplotlib.pyplot as plt

# **📌 Weather Data Fetching Function**
def fetch_weather_data():
    print("🔄 Weather data fetching function executed")  # Log message
    city = "Istanbul"  # Change the city
    url = f"https://www.mgm.gov.tr/tahmin/il-ve-ilceler.aspx?il={city}"

    options = Options()
    options.headless = True
    driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=options)

    driver.get(url)
    time.sleep(5)  # Wait for the page to load

    try:
        # Fetch data
        temperature = driver.find_element(By.CLASS_NAME, "anlik-sicaklik-deger").text.strip().replace(",", ".")
        humidity = driver.find_element(By.CLASS_NAME, "anlik-nem-deger-kac").text.strip()
        wind_speed = driver.find_element(By.CLASS_NAME, "anlik-ruzgar-deger-kac").text.strip()
        wind_direction = driver.find_element(By.CLASS_NAME, "anlik-ruzgar-ikon").get_attribute("title").strip()

        # Current date and time
        date = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

        # Print data to console
        print(f"📅 {date} - 🌡️ Temperature: {temperature} - 💧 Humidity: {humidity} - 🌬️ Wind Speed: {wind_speed} - 🔄 Wind Direction: {wind_direction}")

        # Save data
        save_data(date, temperature, humidity, wind_speed, wind_direction)

        # If the weather is too cold, send a warning
        try:
            temperature_value = float(temperature)  # Convert to float after replacing comma with dot

            if temperature_value < 10:
                message = f"Warning! The temperature dropped to {temperature} degrees. Dress warmly!"
                send_email(message)

        except ValueError:
            print("Error! Temperature value could not be retrieved properly.")
    except Exception as e:
        print(f"An error occurred: {e}")
    finally:
        driver.quit()

# **📌 Save Data Function**
def save_data(date, temperature, humidity, wind_speed, wind_direction):
    data = {
        "date": date,
        "temperature": temperature,
        "humidity": humidity,
        "wind_speed": wind_speed,
        "wind_direction": wind_direction
    }

    # Save to JSON file
    with open("weather_data.json", "a") as json_file:
        json.dump(data, json_file)
        json_file.write("\n")

    # Save to CSV file
    with open("weather_data.csv", "a", newline="") as csv_file:
        fields = ["date", "temperature", "humidity", "wind_speed", "wind_direction"]
        writer = csv.DictWriter(csv_file, fieldnames=fields)

        # Write headers if the file is empty
        if csv_file.tell() == 0:
            writer.writeheader()

        writer.writerow(data)

# **📌 Send Email Function**
def send_email(message):
    GMAIL_ADDRESS = "your_email@gmail.com"  # Enter your email address
    GMAIL_PASSWORD = "your_app_password"  # Enter your Gmail app password
    RECIPIENT_EMAIL = "recipient_email@gmail.com"  # Enter the recipient's email address

    try:
        server = smtplib.SMTP("smtp.gmail.com", 587)
        server.starttls()
        server.login(GMAIL_ADDRESS, GMAIL_PASSWORD)

        email = MIMEMultipart()
        email["From"] = GMAIL_ADDRESS
        email["To"] = RECIPIENT_EMAIL
        email["Subject"] = "Weather Alert"

        email.attach(MIMEText(message, "plain"))
        server.sendmail(GMAIL_ADDRESS, RECIPIENT_EMAIL, email.as_string())
        server.quit()

        print(f"Warning Sent: {message}")

    except Exception as e:
        print(f"An error occurred while sending the email: {e}")

# **📌 Update Temperature Data of the Last 7 Days**
def update_temperature_data():
    print("🔄 Updating temperature data for the last 7 days...")
    date = datetime.now()
    data = []

    # Read existing CSV file to update data every day
    try:
        with open("last_7_days_temperature.csv", "r") as csv_file:
            reader = csv.DictReader(csv_file)
            data = list(reader)
    except FileNotFoundError:
        pass

    # Fetch and add the latest data
    try:
        fetch_weather_data()
        with open("weather_data.csv", "r") as csv_file:
            reader = csv.DictReader(csv_file)
            for row in reader:
                if row["date"].startswith(date.strftime("%Y-%m-%d")):
                    temperature = float(row["temperature"])
                    data.append({"date": date.strftime("%Y-%m-%d"), "temperature": temperature})
                    break
    except Exception as e:
        print(f"Data fetching error: {e}")

    # Keep data for the last 7 days
    if len(data) > 7:
        data = data[-7:]

    # Save data to CSV file
    with open("last_7_days_temperature.csv", "w", newline="") as csv_file:
        fields = ["date", "temperature"]
        writer = csv.DictWriter(csv_file, fieldnames=fields)
        writer.writeheader()
        writer.writerows(data)

    print("📊 Temperature data for the last 7 days has been saved.")

# **📌 Create Graphical Report**
def create_graphical_report():
    print("📈 Creating graphical report...")
    date = []
    temperature = []

    # Read data from CSV file
    with open("last_7_days_temperature.csv", "r") as csv_file:
        reader = csv.DictReader(csv_file)
        for row in reader:
            date.append(row["date"])
            temperature.append(float(row["temperature"]))

    # Create graph
    plt.figure(figsize=(10, 5))
    plt.plot(date, temperature, marker="o", linestyle="-")
    plt.xlabel("Date")
    plt.ylabel("Temperature (°C)")
    plt.title("Temperature Trend of the Last 7 Days")
    plt.grid(True)
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.savefig("temperature_trend.png")
    plt.show()
    print("📊 Graphical report created and saved.")

# **📌 Start Automation**
schedule.every().day.at("13:10").do(fetch_weather_data)  # Fetch weather data every day at 13:10
schedule.every().day.at("13:13").do(update_temperature_data)  # Update temperature data every day at 13:13
schedule.every().monday.at("13:16").do(create_graphical_report)  # Create graphical report every Monday at 13:16

print("Automatic weather data logging started...")

while True:
    print(f"⏳ Waiting... {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")  # Log message
    schedule.run_pending()
    time.sleep(10)  # Check every 10 seconds
