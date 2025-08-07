# 🌤️ AWS Serverless Weather Alert System

A fully automated serverless project that fetches real-time weather data for a city using **OpenWeatherMap API** and sends alerts via **Amazon SNS** when certain conditions are met — all powered by **AWS Lambda**, **EventBridge**, and **IAM**.

---

## 🛠️ Tech Stack

- **AWS Lambda**
- **Amazon SNS**
- **Amazon EventBridge (CloudWatch Events)**
- **OpenWeatherMap API**
- **IAM Roles/Policies**
- **Serverless Architecture**

---

## 📌 Project Use Case

Many times we want alerts when it's going to **rain heavily**, **storm**, or **be extremely hot/cold**. This project lets you:

✅ Receive **automated email alerts** based on weather  
✅ Run a **Lambda function on a schedule** (hourly, daily etc.)  
✅ Integrate **external APIs with AWS**

---

## 🧠 How It Works

```
User specifies city → Lambda fetches weather via OpenWeatherMap API
→ If condition (e.g. Rain, Thunderstorm) is met → SNS triggers Email
→ Done!
```

---

## 📐 Architecture Diagram

<img width="1536" height="1024" alt="ChatGPT Image Aug 7, 2025, 01_02_39 PM" src="https://github.com/user-attachments/assets/59bad292-9def-4b58-8564-352504f6fbd9" />

```
+-------------------+       HTTP Request      +-----------------------+
|   EventBridge     |  -------------------->  |    AWS Lambda         |
| (Scheduled Rule)  |                         | (Python weather fetch)|
+-------------------+                         +-----------+-----------+
                                                          |
                                                          | SNS Publish
                                                          ▼
                                                +------------------+
                                                |  Amazon SNS      |
                                                |  (WeatherAlert)  |
                                                +------------------+
                                                         |
                                                Email Alert Sent!
```

---

## 📦 Setup Guide

### 🔹 1. Create SNS Topic

- Go to **Amazon SNS** → **Create topic**
- Choose **Standard**
- Name: `WeatherAlert`
- Click **Create Topic**

📸 
<img width="1917" height="979" alt="Screenshot 2025-07-18 171121" src="https://github.com/user-attachments/assets/4b07a62b-d3d9-4bee-9135-54dc9fc8f77b" />
<img width="1919" height="974" alt="Screenshot 2025-07-18 171137" src="https://github.com/user-attachments/assets/4337850e-ac80-4eb1-af78-cfd4a89c583e" />

### 🔹 2. Subscribe an Email to SNS

- Inside your topic → Click **Create subscription**
- Protocol: **Email**
- Endpoint: **Your email address**
- Confirm subscription from your inbox.

📸 
<img width="1919" height="984" alt="Screenshot 2025-07-18 171228" src="https://github.com/user-attachments/assets/3b7df46b-cdfe-4317-a20e-b7b65e431ed4" />

---

### 🔹 3. Get API Key from OpenWeatherMap

- Go to: [https://openweathermap.org/api](https://openweathermap.org/api)
- Sign up → Copy the **API Key**

📸 
<img width="1919" height="970" alt="Screenshot 2025-08-07 120501" src="https://github.com/user-attachments/assets/09f44ed9-e8b7-4186-9b9d-50dbf7a06a9a" />

---

### 🔹 4. Create Lambda Function

- Runtime: `Python 3.12`
- Name: `weatheralertfunction`
- Paste the following code:

```python
import json
import urllib.request
import boto3

sns = boto3.client("sns")

def lambda_handler(event, context):
    city = "Mumbai"  # You can replace this with your city.
    api_key = "YOUR_OPENWEATHERMAP_API_KEY"
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}&units=metric"

    try:
        with urllib.request.urlopen(url) as response:
            data = json.loads(response.read().decode("utf-8"))

        temperature = data["main"]["temp"]
        weather = data["weather"][0]["description"]

        message = f"🌦️ Weather Alert for {city}:\nTemperature: {temperature}°C\nCondition: {weather}"

        if "rain" in weather.lower() or temperature > 40:
            message += "\n⚠️ Warning: Extreme conditions!"

        # Replace with your actual SNS Topic ARN
        sns.publish(
            TopicArn="YOUR_SNS_TOPIC_ARN",
            Message=message,
            Subject="🚨 Weather Alert"
        )

        return {
            "statusCode": 200,
            "body": json.dumps({"message": "Alert sent successfully", "details": message})
        }

    except Exception as e:
        return {
            "statusCode": 500,
            "body": json.dumps({"error": str(e)})
        }

```

📸 
<img width="1919" height="979" alt="Screenshot 2025-07-18 170524" src="https://github.com/user-attachments/assets/bf5cdc72-8a90-4ce3-9a12-8b26966f8e6c" />
<img width="1918" height="983" alt="Screenshot 2025-07-18 170710" src="https://github.com/user-attachments/assets/306d44e3-398e-4077-af1f-9baf2d68644b" />

---

### 🔹 5. Add IAM Permissions to Lambda and Edit Environment Variables.

- Go to Lambda → **Permissions tab**
- Click on **Execution Role**
- Attach Policy: In JSON or VISUAL
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:us-east-1:YOUR_ACCOUNT_ID:WeatherAlert"
    }
  ]
}
```

📸 
<img width="1919" height="980" alt="Screenshot 2025-07-18 171522" src="https://github.com/user-attachments/assets/3b52d7b1-0ac7-4b0b-aacd-af8a90ff763a" />
<img width="1919" height="983" alt="Screenshot 2025-07-18 171636" src="https://github.com/user-attachments/assets/8b386a82-c7df-43dc-aef4-4d4736e9a798" />
<img width="1919" height="979" alt="Screenshot 2025-07-18 172725" src="https://github.com/user-attachments/assets/592f3fdf-8794-4583-9955-9f194af09abb" />
<img width="1919" height="971" alt="Screenshot 2025-07-18 172735" src="https://github.com/user-attachments/assets/7625dd58-be77-44d1-ad0b-4db5ea047d48" />
<img width="1919" height="980" alt="Screenshot 2025-07-18 173039" src="https://github.com/user-attachments/assets/32372578-1d12-4306-8114-73385787d397" />
<img width="1919" height="984" alt="Screenshot 2025-07-18 173200" src="https://github.com/user-attachments/assets/1c2e3aea-56a1-46e2-89fc-9b211902a17c" />

---

### 🔹 6. Create Scheduled Trigger (EventBridge)

- Go to Lambda → **Triggers** → **+ Add trigger**
- Choose: `EventBridge (CloudWatch Events)`
- Create a new rule:
  - Name: `HourlyWeatherCheck`
  - Schedule expression: `rate(1 hour)`
- Save trigger.

📸 
<img width="1919" height="978" alt="Screenshot 2025-07-18 173329" src="https://github.com/user-attachments/assets/e9221b27-7602-4773-8ae5-69529b896b24" />
<img width="1919" height="975" alt="Screenshot 2025-07-18 173606" src="https://github.com/user-attachments/assets/7f2eca6d-4e3a-49e7-b1a9-8c7ae042199e" />

---

## ✅ Testing the System

- Click **Test** inside Lambda to trigger manually.
- Or wait for EventBridge to invoke it on schedule.
- Check your inbox for email like:

```
Subject: ⚠️ Weather Alert
Body: Weather Alert for Pune: Thunderstorm, Temperature: 31°C
```

📸 
<img width="1919" height="971" alt="Screenshot 2025-07-18 173816" src="https://github.com/user-attachments/assets/a75bd556-a85e-4a99-9310-2813d45f4f10" />

---


## 🏁 Conclusion

This project gave me hands-on experience with:
- **Real-world API integration**
- **AWS SNS notifications**
- **Scheduled event triggers**
- **IAM roles & permissions**

---

## 🧑‍💻 Author

**UJJWAL WADHAI**  

🔗 [GitHub](https://github.com/Ujjwal-04)
🔗 [LinkedIn](www.linkedin.com/in/ujjwal-wadhai)

---

