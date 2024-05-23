Materiel used for this project:


![Screenshot 2024-05-16 191917](https://github.com/AmirGeorgesHaya/Unix_Project/assets/129766673/08cf83a3-fe7d-480f-8aba-06c913f93b48) ![eseries_slider_512GB_1](https://github.com/AmirGeorgesHaya/Unix_Project/assets/129766673/3153385d-e5a9-4ebb-bb34-b91035d765b7)

1: Buy  the Raspberry Pi 4 starter kit

2: This Raspberry Pie comes with Kodi alreasy built-in. However we will flash it to have OSMC.

3: After flashing, take your Micro SD Card and plug it in your SD card reader.

4:Plug the SD card reader in a USB-C port.

5:  Go to osmc.tv to get the installer matching your pc. After intsalling and running it, a few questions on the setup and flashing will be asked in advanced.

6: Make sure to choose the current pi your using, the version you want to install, what will you be flashing it on(E.g. SD Card, USB stick, NFS share, etc..), and what device (E.g. D:/Kingston).

7: It will ask a few more questions about your set up, such as choosing between a wired or wireless connection and such.

14: Unplug your the micro SD card and plug it into your Raspberry Pi.

15: Connect the Raspberry Pi to the power and connect the HDMI cable to the wanted monitor

16: On boot, go to system's OSMC to add the network. Configure your network if you are using wirelessly or else the Raspberry should automitcally reconize your Ethernet cable. If no network appear, try rebooting the system.

17: Go to System -> My OSMC -> Services. Turn on SSH since it will be used to connect a computer to the Media player easily.

18: Go to the system information and find the IP Address of your OSMC. 

19: Using a Linux-based system, go to the terminal and write this command. replace libreelec_ip with the IP address from the OSMC

ssh root@osmc_ip

20: the password is osmc by default. Make sure you also respon yes when they ask you for conifirmation of the connection

21: Next write this command to not only create the script, but also edit it.

nano activity_log_file.py

22: Use this following code to recieve notifications of your viewing experience:

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
password = amir_sen123. "your-email-password"  # Replace with your Gmail password or app-specific password
smtp_server = "smtp.gmail.com"
smtp_port = 587  # Use 587 for TLS

# Send the email
send_email(subject, body, sender_email, receiver_email, password, smtp_server, smtp_port)

print("Activity log has been sent via email.")  # Optional: for manual run feedback

23: Replace the "your-sender-email@gmail.com" with sendoesamircle.gmail.com and use the password  and make your receiever email your personal email.

24: Press CTRL + O to confirm the script and press ENTER to save the files.

25: Make sure you can execute this file. For this, use this command:

chmod +x log_user_activity_email.py

26: use this following command to test the file and make sure theirs no errors. 

27: use crontab to give a timer to your notification

crontab -e

28: in the crontab editor the first wo numbers showcase the time in a 24h format. in our example, it is 8 am. when you're done, click CTRL + O to save and click enter to confirm.a

0 8 * * * /usr/bin/python3 /path/to/your/log_user_activity_email.py

29: write "exit" to exit the connection with LibreELEC. When done everything is good and set up.
