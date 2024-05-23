# Developpement History
## Starting point
This project started as a Media Center that would send the user a notification once a day. These notifications would show the user its viewing/activity history. When brainstorming, this was our first idea to create since we've seen it as an example during the lectures and thought it would be interesting to tackle this idea. We first decided about which version of Raspberry Pi we needed. Getting the most recent one was in our minds but we wanted this Rapberry to be useful in the long run. We basically wanted to ba able to use it during IoT classes as well. Our teacher talked to the IoT teacher and let us know that the IoT class used Raspberry Pi 4 for the semester. So we decided to buy a starting kit online : https://www.amazon.ca/-/fr/Raspberry-Pi-Model-Cortex-A72-dalimentation/dp/B0CDBTPJYR. 


We proceeded to follow a guide to install OSMC(https://www.youtube.com/watch?v=ngGkVd-_LP8&ab_channel=ETAPRIME)  The reason we chose this Media Center was for it's simplicity and comptability with Debian. However, these were things that we saw online reviews about. We had to actually download it to know if it worked or not. Our backup plan would be to try with LibreELEC.

## Problem Occured
Since this is supposed to be a Media Center that would send notifications to the user about its activity, we needed a script that the Raspberry Pi would be running consistently. To make this possible, we needed to use SSH between a Debian console and OSCM. We would also need to use crontab to make the script execute on a regular basis and finally, we needed the script itself which will be written in Python since the libraries we used for the activities hisotry only exists with Python. They are already created. Basically, we could have used Java, or C++ but that means we would have to create the libraries ourselves and this would slow the pace of our developpement by at least 5 days. After initilizing the scripts, crontab, SSH and executable rights.

Connection to the SSH. The password is the password that you used to log in. If you don't have a password, the default password was: osmc.
```
ssh osmc@osmc_ip
```

This was the script used.
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

## Send the email
send_email(subject, body, sender_email, receiver_email, password, smtp_server, smtp_port)

print("Activity log has been sent via email.")  # Optional: for manual run feedback

```

We added the executable rights
```
chmod +x log_user_activity_email.py
```

We open the crontab editior and add python script to it (named log_user_activity_email.py).
```
crontab -e

0 8 * * * /usr/bin/python3 /path/to/your/log_user_activity_email.py
```
Sadly, after configuring it, we were not able to get any notifications to work. we even waited 1-2 days to make sure maybe the system has to boot it or had to be used, but it still wouldn't work.

## Modification trial
Since this wouldn't work, we decided to try it with LibreELEC, however after using the same configuration as before, we realized that LibreELEC wasn't compatible with Debian. Sadly, this meant that most basic commands on Debian would not register once we connected to LibreELEC. Commands such as su- or sudo would not even exist and can't be downloaded. Even though we really liked the UI provided by LibreELEC, it had too many problems: The time and date were not synchronized with real date and time which lead to media related problems and the bluetooth was very buggy and didn't work well for us. Even trying the speakers provided in class wouldn't fix the problem. Some YouTube videos did not exist after a certain date. It was acting like the Internet Explorer.

## Review project list
After reviewing everything, we decided to go back to OSMC and try it again with different script. We decided to test a script first. The test consists of making shuffled playlists to be sent to us. Looking back our our steps previously, all we did was add a script and also modify the pre-existing ones. Basically making project have two features instead of one since the test was successful. The script would've also sent a playlist to be shuffled on youtube plugin on the raspberrybi. Unfortunately that didn't turn out as we hoped as youtube ang gmail have multitudes of security that forbid the techniques we used.


This makes it so we would get shuffled playlists everytime; however, we realized that the videos were not randomized and that it was the same video everytime. In hindsight, this wasn't a bad thing since all we were trying to do was to make OSMC send emails to us and it suceeded in doing so. We got emails saying that a random playlist was created, but it just led to the same video. We still call this test a success.

### Updated Code which does work for OSMC but only sends an email with the playlist

```
ssh root@libreelec_ip

nano shuffled_playlist.py

```

```
import xbmc
import xbmcgui
import xbmcplugin
import random
from pytube import Playlist

# Function to send a shuffled list via email
def send_shuffled_list_via_email(email_address, email_password, recipients, shuffled_list):
    import smtplib
    from email.mime.multipart import MIMEMultipart
    from email.mime.text import MIMEText

    # Shuffle the list
    random.shuffle(shuffled_list)

    # Create the email message
    msg = MIMEMultipart()
    msg['From'] = email_address
    msg['To'] = ', '.join(recipients)
    msg['Subject'] = 'Shuffled Playlist'

    # Convert the shuffled list to a string
    shuffled_list_str = '\n'.join(shuffled_list)

    # Add the shuffled list to the email body
    body = f"The shuffled playlist order is:\n\n{shuffled_list_str}"
    msg.attach(MIMEText(body, 'plain'))

    # Connect to SMTP server and send the email
    with smtplib.SMTP('smtp.office365.com', 587) as smtp:
        smtp.starttls()
        smtp.login(email_address, email_password)
        smtp.send_message(msg)

# Function to play a shuffled playlist in the YouTube add-on
def play_shuffled_playlist_in_kodi(video_urls):
    playlist = xbmc.PlayList(xbmc.PLAYLIST_VIDEO)
    playlist.clear()
    for video_url in video_urls:
        listitem = xbmcgui.ListItem(path=video_url)
        playlist.add(video_url, listitem)
    xbmc.Player().play(playlist)

# Main function
def main():
    # Email configuration
    email_address = '******@outlook.com'  # Fill in with your Outlook email address
    email_password = '******'  # Fill in with your Outlook email password
    recipients = [donissaint13@gmail.com']  # Fill in with actual recipient email addresses

    # Predefined YouTube playlist URL
    playlist_url = "https://www.youtube.com/playlist?list=PLcZCyfhxwUBo376IvSmasg_WmC8LkPeVW"

    # Fetch the playlist and extract video URLs
    playlist = Playlist(playlist_url)
    video_urls = [video.watch_url for video in playlist.videos]

    # Send the shuffled playlist via email
    send_shuffled_list_via_email(email_address, email_password, recipients, video_urls)

    # Shuffle the video URLs before playing
    random.shuffle(video_urls)

    # Play the shuffled playlist in Kodi
    play_shuffled_playlist_in_kodi(video_urls)

if __name__ == "__main__":
    main()


```

This was the script done in order for the user (us) to recieve emails by  OSMC. We received about 15 notifications until we stopped it completly or else it would put our emails into a spam folder.
This sadly happened actually and it stopped sending messages to us since. OSMC noticed that it was going into a spam folder and blocked itself from sending it again. This code fetched all lines in kodi.log that included the keyword "plugin" used for finding all plugins running. We limited those lines to the latest 6 one as kodi.log contains all logs from the beginning.
```
### Remade Code
import os
import re
import smtplib
from datetime import datetime
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

# Function to parse the Kodi log file
log_file_path = '/home/osmc/.kodi/temp/kodi.log'
def parse_kodi_log(log_file_path):
	if not os.path.exists(log_file_path):
    	print(f"Log file not found: {log_file_path}")
    	return {}

	running_lines = []

	with open(log_file_path, 'r') as log_file:
    	for line in log_file:
        	if "plugin" in line:
            	if len(running_lines) >= 6:
                	running_lines.pop(0)
            	running_lines.append(line.strip()+"\n")

	return running_lines
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
	email_address = 'senamiracle@outlook.com'
	email_password = 'amir_sen123'
	recipients = ['musither@gmail.com', 'kirbywerby482@gmail.com', 'senamiracle@outlook.com']
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

```
Here is how the error looks like:

![image](https://github.com/AmirGeorgesHaya/Unix_Project/assets/129766673/ef0ea3f3-1fa7-45ea-9755-199001a61c66)


