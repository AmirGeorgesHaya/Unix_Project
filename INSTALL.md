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

19: Using a Linux-based system, go to the terminal and write this command. replace osmc_ip with the IP address from the OSMC

ssh osmc@osmc_ip

20: The password is osmc by default. Make sure you also respon yes when they ask you for conifirmation of the connection

21: Next write this command to not only create the script, but also edit it.

nano activity_log_file.py

22: Use this following code to recieve notifications of your viewing experience:

import os
import re
import smtplib
from datetime import datetime
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

# Function to send email
def send_email(email_address, email_password, recipients, subject, body):
	msg = MIMEMultipart()
	msg['From'] = email_address
	msg['To'] = ', '.join(recipients)
	msg['Subject'] = subject

	body = body.replace("\n", "\r\n")

	msg.attach(MIMEText(body, 'plain'))

try:
    	with smtplib.SMTP('smtp.office365.com', 587) as smtp:
        	smtp.starttls()
        	smtp.login(email_address, email_password)
        	smtp.send_message(msg)
    	print(f"Email sent to {', '.join(recipients)}")
	except Exception as e:
    	print(f"Failed to send email: {e}")

# Main function
def main():
	email_address = 'YOUR OUTLOOK EMAIL'
	email_password = 'YOUR OUTLOOK EMAIL'S PASSWORD'
	recipients = ['example@example.com', 'example@example.com']
	log_file_path = '/home/osmc/.kodi/temp/kodi.log'  # Update this path if necessary

	accessed_addons = parse_kodi_log(log_file_path)

	if accessed_addons:
    	body = "Programs that were running in the log:\n\n"
    	for line in accessed_addons:
        	body += f"- {line}\n"
	else:
    	body = "No programs were found running in the log."

	send_email(email_address, email_password, recipients, "Kodi Accessed Add-ons", body)

if __name__ == "__main__":
	main()


23: Replace every email fill-in (YOUR OUTLOOK EMAIL, PASSWORD, example@example.com).

24: Press CTRL + X -> Y -> ENTER to confirm the script and press ENTER to save the files.

25: Make sure you can execute this file. For this, use this command:

chmod +x activity_log_file.py

26: use this following command to test the file and make sure theirs no errors. 

python activity_log_file.py

27: use crontab to give a timer to your notification

crontab -e

28: in the crontab editor the first wo numbers showcase the time in a 24h format. in our example, it is 8 am. when you're done, click CTRL + X to save and click enter to confirm.a

0 8 * * * /usr/bin/python3 /path/to/your/activity_log_file.py

29: write "exit" to exit the connection with OSMC. When done everything is good and set up.
