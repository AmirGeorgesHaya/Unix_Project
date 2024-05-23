# Developpement History
## Starting point
This project started as a Media Center that would send the user a notification once a day. These notifications would show the user its viewing/activity history. When brainstorming, this was our first idea to create since we've seen it as an example during the lectures and thought it would be interesting to tackle this idea. We first decided about which version of Raspberry Pi we needed. Getting the most recent one was in our minds but we wanted this Rapberry to be useful in the long run. We basically wanted to ba able to use it during IoT classes as well. Our teacher talked to the IoT teacher and let us know that the IoT class used Raspberry Pi 4 for the semester. So we decided to buy a starting kit online : https://www.amazon.ca/-/fr/Raspberry-Pi-Model-Cortex-A72-dalimentation/dp/B0CDBTPJYR. 


We proceeded to follow a guide to install OSMC(https://www.youtube.com/watch?v=ngGkVd-_LP8&ab_channel=ETAPRIME)  The reason we chose this Media Center was for it's simplicity and comptability with Debian. However, these were things that we saw online reviews about. We had to actually download it to know if it worked or not. Our backup plan would be to try with LibreELEC.

## Problem Occured
Since this is supposed to be a Media Center that would send notifications to the user about its activity, we needed a script that the Raspberry Pi would be running consistently. To make this possible, we needed to use SSH between a Debian console and OSCM. We would also need to use crontab to make the script execute on a regular basis and finally, we needed the script itself which will be written in Python since the libraries we used for the activities hisotry only exists with Python. They are already created. Basically, we could have used Java, or C++ but that means we would have to create the libraries ourselves and this would slow the pace of our developpement by at least 5 days. After initilizing the scripts, crontab, SSH and executable rights.

Connection to the SSH. The password is the password that you used to log in. If you don't have a password, the default password was: osmc.
```
ssh root@libreelec_ip
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
Since this wouldn't work, we decided to try it with LibreELEC, however after using the same configuration as before, we realized that LibreELEC wasn't compatible with Debian. Sadly, this meant that most basic commands on Debian would not register once we connected to LibreELEC. Commands such as su- or sudo would not even exist and can't be downloaded. Even though we really liked the UI provided by LibreELEC, it had too many problems: The time and date were not synchronized with real date and time which lead to media related problems. Some YouTube videos did not exist after a certain date. It was acting like the Internet Explorer.

## Review project list
After reviewing everything, we decided to go back to OSMC and try it again with different script. We decided to test a script first. The test consists of making shuffled playlists to be sent to us. Looking back our our steps previously, all we did was add a script and also modify the pre-existing ones. Basically making project have two features instead of one since the test was successful. 

### Updated Code
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
This makes it so we would get shuffled playlists everytime.

### Remade Code





