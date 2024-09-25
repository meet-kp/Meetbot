import pyttsx3
import speech_recognition as sr
import datetime
import wikipedia
import webbrowser
import os
import smtplib
import requests
import logging

# Initialize the speech engine
engine = pyttsx3.init('sapi5')
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[0].id)  # Change this to voices[1].id for a female voice

# Setup logging
logging.basicConfig(filename='assistant.log', level=logging.INFO)

def speak(audio):
    """Converts text to speech."""
    engine.say(audio)
    engine.runAndWait()

def wishMe():
    """Wishes the user according to the time of day."""
    hour = int(datetime.datetime.now().hour)
    if 0 <= hour < 12:
        speak("Good Morning!")
    elif 12 <= hour < 18:
        speak("Good Afternoon!")
    else:
        speak("Good Evening!")
    speak("I am MeetBot. Please tell me how may I assist you.")

def takeCommand():
    """Listens for voice input and returns the command as a string."""
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        r.pause_threshold = 1
        audio = r.listen(source)
    try:
        print("Recognizing...")
        query = r.recognize_google(audio, language='en-in')
        print(f"User said: {query}\n")
    except Exception as e:
        print("Say that again please...")
        return "None"
    return query.lower()

def sendEmail(to, content):
    """Sends an email to the specified address."""
    try:
        server = smtplib.SMTP('smtp.gmail.com', 587)
        server.ehlo()
        server.starttls()
        server.login(os.environ.get('EMAIL_USER'), os.environ.get('EMAIL_PASS'))
        server.sendmail(os.environ.get('EMAIL_USER'), to, content)
        server.close()
        speak("Email has been sent!")
    except Exception as e:
        logging.error(f"Error sending email: {e}")
        speak("Sorry, I couldn't send the email.")

def open_website(url):
    """Opens the specified website."""
    webbrowser.open(url)
    speak(f"Opening {url}")

def get_weather(city):
    """Fetches weather information for a given city."""
    api_key = os.environ.get('WEATHER_API_KEY')
    base_url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}"
    response = requests.get(base_url).json()
    if response["cod"] != "404":
        main = response['main']
        temperature = main['temp'] - 273.15  # Convert from Kelvin to Celsius
        speak(f"The temperature in {city} is {temperature:.2f} degrees Celsius.")
    else:
        speak(f"Sorry, I couldn't fetch weather for {city}.")

if __name__ == "__main__":
    wishMe()
    while True:
        query = takeCommand()

        if 'wikipedia' in query:
            speak('Searching Wikipedia...')
            query = query.replace("wikipedia", "")
            results = wikipedia.summary(query, sentences=2)
            speak("According to Wikipedia")
            print(results)
            speak(results)

        elif 'open youtube' in query:
            open_website("youtube.com")

        elif 'open google' in query:
            open_website("google.com")

        elif 'open stackoverflow' in query:
            open_website("stackoverflow.com")

        elif 'play music' in query:
            music_dir = 'D:\\Non Critical\\songs\\Favorite Songs2'  # Modify path as needed
            songs = os.listdir(music_dir)
            os.startfile(os.path.join(music_dir, songs[0]))

        elif 'the time' in query:
            strTime = datetime.datetime.now().strftime("%H:%M:%S")
            speak(f"Sir, the time is {strTime}")

        elif 'open code' in query:
            codePath = "C:\\Users\\YourUser\\AppData\\Local\\Programs\\Microsoft VS Code\\Code.exe"  # Modify path as needed
            os.startfile(codePath)

        elif 'email to' in query:
            try:
                speak("What should I say?")
                content = takeCommand()
                to = "recipient@example.com"  # Replace with the recipient's email
                sendEmail(to, content)
            except Exception as e:
                speak("Sorry, I am not able to send this email.")

        elif 'weather in' in query:
            speak("Which city?")
            city = takeCommand().lower()
            get_weather(city)
