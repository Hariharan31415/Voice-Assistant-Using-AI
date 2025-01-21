import subprocess
import speech_recognition as sr
from gtts import gTTS
import pygetwindow as gw
import os
import time
import pyautogui
import webbrowser
import cv2
import random
import ctypes
from pydub import AudioSegment
from pydub.playback import play
from datetime import datetime
import requests
from bs4 import BeautifulSoup
from transformers import pipeline
import threading
from threading import Timer
import nltk
import pygame
import googlesearch
import serial
import serial.tools.list_ports

recognizer = sr.Recognizer()
tasks = []
pygame.mixer.init()
correct_password = "charlin"
user_info = {'nickname': 'charlin Sparky', 'year_of_birth': 'charlin 2004', 'month_of_birth': 'charlin 11', 'favorite_food': 'charlin biryani'}

def speak(text, lang='en'):
    tts = gTTS(text=text, lang=lang)
    tts.save("response.mp3")
    sound = AudioSegment.from_mp3("response.mp3")
    play(sound)

def play_song(song_name):
    try:
        s_name = song_name[7:]
        print(s_name)
        song_path = f"music/{s_name}.mp3"
        pygame.mixer.music.load(song_path)
        pygame.mixer.music.play()
        speak(f"Now playing {song_name}")
        print(f"Now playing {song_name}")
        while pygame.mixer.music.get_busy():
            continue
    except pygame.error as e:
        speak("Sorry, I couldn't play the song.")
        print("Error:", e)

def open_google_maps(location):
    if location:
        loc = location[7:]
        search_url = f"https://www.google.com/maps/search/?api=1&query={loc}"
        webbrowser.open(search_url)
        speak("Opening Google Maps")
        print("Opening Google Maps")
    else:
        speak("Please provide a location.")
        print("Please provide a location.")

def take_screenshot():
    screenshot_path = "screenshot.png"
    screenshot = pyautogui.screenshot()
    screenshot.save(screenshot_path)
    speak("Screenshot taken. Check your desktop for the screenshot.")
    print("Screenshot taken. Check your desktop for the screenshot.")

def gen_response(query):
    query = query.lower()
    global tasks
    if "exit" in query or "stop" in query:
        speak("Goodbye. Have a great day!")
        print("Goodbye. Have a great day!")
        return "exit"
    elif "add a task" in query:
        speak("Sure, what is the task?")
        print("Sure, what is the task?")
        task = listen()
        speak("When should the task be completed (in minutes from now)?")
        print("When should the task be completed (in minutes from now)?")
        time_input = listen()
        if time_input != "None":
            time_numeric = ''.join(filter(str.isdigit, time_input))
            if time_numeric:
                try:
                    time_minutes = int(time_numeric)
                    add_task({'task': task, 'time': time_minutes})
                except ValueError:
                    speak("Invalid input for time. Please provide a valid number of minutes.")
                    print("Invalid input for time. Please provide a valid number of minutes.")
            else:
                speak("No numeric input received for time. Task not added.")
                print("No numeric input received for time. Task not added.")
        else:
            speak("No input received for time. Task not added.")
            print("No input received for time. Task not added.")
    elif "list tasks" in query:
        if tasks:
            speak("Sure. Your tasks are:")
            print("Sure. Your tasks are:")
            for task in tasks:
                speak(task['task'])
        else:
            speak("You don't have any tasks.")
            print("You don't have any tasks.")
    elif "what's your name" in query:
        speak("I am your AI assistant, how can I help you?")
        print("I am your AI assistant, how can I help you?")
    elif "what's the time" in query:
        current_time = datetime.now().strftime("%I:%M:%S %p")
        speak("The current time is " + current_time)
        print("The current time is " + current_time)
    elif "what's the date" in query:
        current_date = datetime.now().strftime("%d-%m-%Y")
        speak("Today's date is " + current_date)
        print("Today's date is " + current_date)
    elif "open" in query:
        domain = query.split("open")[-1].strip()
        if "chrome" in domain:
            webbrowser.open("http://www.google.com")
            speak("Opening Google Chrome")
            print("Opening Google Chrome")
        elif "youtube" in domain:
            webbrowser.open("http://www.youtube.com")
            speak("Opening YouTube")
            print("Opening YouTube")
        elif "camera" in domain:
            open_camera()
        elif "notepad" in domain:
            os.system("start notepad")
            speak("Opening Notepad")
            print("Opening Notepad")
        elif "calculator" in domain:
            os.system("start calc")
            speak("Opening Calculator")
            print("Opening Calculator")
        elif "command prompt" in query or "cmd" in query:
            os.system("start cmd")
            speak("Opening Command Prompt")
            print("Opening Command Prompt")
        elif "file manager" in domain:
            os.system("start explorer")
            speak("Opening File Manager")
            print("Opening File Manager")
        elif "google maps" in domain:
            speak("Sure, what location do you want to search on Google Maps?")
            print("Sure, what location do you want to search on Google Maps?")
            location = listen()
            open_google_maps(location)
        else:
            speak("I am sorry, I cannot open " + domain)
            print("I am sorry, I cannot open " + domain)
    elif "play a song" in query:
        speak("Sure, which song would you like to play?")
        song_name = listen()
        play_song(song_name)
    elif "play music" in query:
        play_google_music()
    elif "check weather" in query:
        url = "https://www.google.com/search?q=check+weather"
        webbrowser.open(url)
    elif "take a screenshot" in query:
        take_screenshot()
    elif "sleep" in query:
        speak("Putting the computer to sleep")
        sleep_computer()
    elif "shutdown" in query:
        os.system("shutdown /s /t 1")
    elif "restart" in query:
        os.system("shutdown /r /t 1")
    elif "search for" in query:
        search_query = query.replace("search for", "").strip()
        search_url = f"https://www.google.com/search?q={search_query}"
        webbrowser.open(search_url)
        speak("Here are the search results for " + search_query)
    elif "lights on" in query or "lights off" in query or "fan on" in query or "fan off" in query:
        communicate_with_arduino()
    else:
        speak("I didn't understand that. Can you repeat?")
        print("I didn't understand that. Can you repeat?")
    return None

def list_com_ports():
    ports = serial.tools.list_ports.comports()
    for port in ports:
        print(port)

def select_com_port():
    com = 8
    for port in serial.tools.list_ports.comports():
        if port.device.startswith("COM" + str(com)):
            return port.device
    return None

def communicate_with_arduino():
    print("Available COM Ports:")
    list_com_ports()
    com_port = select_com_port()
    if com_port:
        print("Using COM Port:", com_port)
        serial_inst = serial.Serial(com_port, 9600)
        while True:
            print("which room: ")
            speak("which room: ")
            command = listen()
            command = command[7:].upper()
            serial_inst.write(command.encode('utf-8'))
            break
    else:
        print("Selected COM Port not found.")

def notify_task(task):
    speak("Notification: It's time to " + task)
    print("Notification: It's time to " + task)

def add_task(task_description):
    global tasks
    tasks.append(task_description)
    speak("Adding '" + task_description['task'] + "' to your task list. You have " + str(len(tasks)) + " tasks in your list.")
    print("Adding '" + task_description['task'] + "' to your task list. You have " + str(len(tasks)) + " tasks in your list.")
    if 'time' in task_description:
        schedule_notification(task_description)

def schedule_notification(task_description):
    task_time = task_description['time']
    task = task_description['task']
    notification_time = task_time * 60
    Timer(notification_time, notify_task, args=[task]).start()

def sleep_computer():
    ctypes.windll.PowrProf.SetSuspendState(0, 1, 0)

def listen():
    with sr.Microphone() as source:
        recognizer.adjust_for_ambient_noise(source)
        print("Listening for commands...")
        recognizer.pause_threshold = 1
        try:
            audio = recognizer.listen(source, phrase_time_limit=5, timeout=5)
            print("Recognizing...")
            query = recognizer.recognize_google(audio)
            print("User query:", query)
            return query
        except sr.UnknownValueError:
            print("Google Speech Recognition could not understand audio.")
            return "None"
        except sr.RequestError as e:
            print("Error requesting results from Google Speech Recognition service:", e)
            return "None"
        except sr.WaitTimeoutError:
            print("No input detected.")
            return "None"

if __name__ == "__main__":
    nltk.download('punkt')
    print("Welcome! Speak your command.")
    while True:
        user_query = listen()
        if user_query != "None":
            response = gen_response(user_query)
            if response == "exit":
                break
