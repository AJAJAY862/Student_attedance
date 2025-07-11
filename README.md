# Student_attedance
# Daily class attendences of students &amp; save the attendence time in data base ( FireBase studio )
 
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button
from kivy.clock import Clock
from kivy.core.window import Window
import requests
from datetime import datetime
import sys

# Set window size (optional for testing)
Window.size = (360, 640)

# Firebase Realtime Database URL
FIREBASE_URL = 'https://student-attendence-a9ddb-default-rtdb.asia-southeast1.firebasedatabase.app/'

# Predefined login credentials
LOGIN_ID = "BDS_2023-27"
LOGIN_PASSWORD = "bds123456@"

# Predefined student list
STUDENTS = [
    "AJAY THAKUR",
    "ANIRUDDHA DEY",
    "ARINDAM DAS",
    "KUSHAL DEY",
    "NISHA DAS",
    "SATYAKI CHAKRABORTY",
    "SNEHADHREE BANDYOPADHYAY",
    "SOUMYADEEP GHOSH",
    "SUBHADIP MITRA"
]

class AttendanceApp(App):
    def build(self):
        self.logged_in = False
        self.layout = BoxLayout(orientation='vertical', padding=20, spacing=20)

        self.label = Label(text="Login", font_size=24)
        self.layout.add_widget(self.label)

        self.id_input = TextInput(hint_text="Enter ID", multiline=False)
        self.layout.add_widget(self.id_input)

        self.password_input = TextInput(hint_text="Enter Password", password=True, multiline=False)
        self.layout.add_widget(self.password_input)

        self.login_button = Button(text="Login", on_press=self.check_login)
        self.layout.add_widget(self.login_button)

        return self.layout

    def check_login(self, instance):
        entered_id = self.id_input.text.strip()
        entered_password = self.password_input.text.strip()

        current_hour = datetime.now().hour
        if entered_id == LOGIN_ID and entered_password == LOGIN_PASSWORD:
            if 9 <= current_hour < 17:
                self.label.text = "Logged in. Submitting Attendance..."
                self.submit_attendance()
            else:
                self.label.text = "Access only allowed between 9 AM and 5 PM."
        else:
            self.label.text = "Invalid Credentials."

    def submit_attendance(self):
        now = datetime.now()
        attendance_data = {
            "teacher": LOGIN_ID,
            "timestamp": now.strftime("%Y-%m-%d %H:%M:%S"),
            "students": STUDENTS,
            "number_of_classes": len(STUDENTS),
        }
        try:
            response = requests.post(FIREBASE_URL, json=attendance_data)
            if response.status_code == 200:
                self.label.text = "Attendance Submitted!"
            else:
                self.label.text = "Submission Failed!"
        except Exception as e:
            self.label.text = f"Error: {str(e)}"

        Clock.schedule_once(self.close_app, 3)

    def close_app(self, dt):
        App.get_running_app().stop()
        sys.exit()

if __name__ == "__main__":
    AttendanceApp().run()
