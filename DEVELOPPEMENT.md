# Starting point
This project started as a Media Center that would send the user a notification once a day. These notifications would show the user its viewing/activity history. When brainstorming, this was our first idea to create since we've seen it as an example during the lectures and thought it would be interesting to tackle this idea. We first decided about which version of Raspberry Pi we needed. Getting the most recent one was in our minds but we wanted this Rapberry to be useful in the long run. We basically wanted to ba able to use it during IoT classes as well. Our teacher talked to the IoT teacher and let us know that the IoT class used Raspberry Pi 4 for the semester. So we decided to buy a starting kit online : https://www.amazon.ca/-/fr/Raspberry-Pi-Model-Cortex-A72-dalimentation/dp/B0CDBTPJYR 


We proceeded to follow a guide to install OSMC(https://www.youtube.com/watch?v=ngGkVd-_LP8&ab_channel=ETAPRIME)  The reason we chose this Media Center was for it's simplicity and comptability with Debian. However, these were things that we saw online reviews about. We had to actually download it to know if it worked or not. Our backup plan would be to try with LibreELEC.

# Problem Occured
Since this is supposed to be a Media Center that would send notifications to the user about its activity, we needed a script that the Raspberry Pi would be running consistently. To make this possible, we needed to use SSH between a Debian console and OSCM. We would also need to use crontab to make the script execute on a regular basis and finally, we needed the script itself which will be written in Python since the libraries we used for the activities hisotry only exists with Python. They are already created. Basically, we could have used Java, or C++ but that means we would have to create the libraries ourselves and this would slow the pace of our developpement by at least 5 days. After initilizing the scripts, crontab, SSH and executable rights.

Connection to the SSH. The password is the password that you used to log in. If you don't have a password, the default password was: osmc
```
ssh root@libreelec_ip
```

This was the script used
```
#!/usr/bin/env python3
import subprocess
import datetime
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# Function to send email
def send_email(subject, body, sender_email, receiver_email, password, smtp_server, smtp_port):
    message = MIMEMultipart()
    message["From"] = sender_email
    message["To"] = receiver_email
    message["Subject"] = subject
    message.attach(MIMEText(body, "plain"))
    
    with smtplib.SMTP(smtp_server, smtp_port) as server:
        server.starttls()  # Start TLS encryption
        server.login(sender_email, password)
        server.sendmail(sender_email, receiver_email, message.as_string())

# Get current time
now = datetime.datetime.now()
date_str = now.strftime("%Y-%m-%d %H:%M:%S")

# Get user activity, using 'who' command
result = subprocess.run(['who'], stdout=subprocess.PIPE)
user_activity = result.stdout.decode('utf-8')

# Email content
subject = f"User Activity Log on {date_str}"
body = f"Activity on {date_str}:\n{user_activity}\n"

# Email credentials and server configuration for Gmail
sender_email = "your-sender-email@gmail.com"  # Replace with your Gmail address
receiver_email = "your-receiver-email@example.com"  # Replace with the recipient's email address
password = "your-email-password"  # Replace with your Gmail password or app-specific password
smtp_server = "smtp.gmail.com"
smtp_port = 587  # Use 587 for TLS

# Send the email
send_email(subject, body, sender_email, receiver_email, password, smtp_server, smtp_port)

print("Activity log has been sent via email.")  # Optional: for manual run feedback

```

We added the executable rights
```
chmod +x log_user_activity_email.py
```

We open the crontab editior and add python script to it (named log_user_activity_email.py)
```
crontab -e

0 8 * * * /usr/bin/python3 /path/to/your/log_user_activity_email.py
```



