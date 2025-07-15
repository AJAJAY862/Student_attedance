# Student_attedance
# Daily class attendences of students &amp; save the attendence time in data base ( FireBase studio )
 
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.button import Button
from kivy.uix.checkbox import CheckBox
from kivy.uix.textinput import TextInput
from kivy.uix.popup import Popup
from datetime import datetime
import socket
import threading
import requests

import firebase_admin
from firebase_admin import credentials, db

# Firebase Initialization
cred = credentials.Certificate("student-attendence-a9ddb-firebase-adminsdk-fbsvc-a6535b3235.json")
firebase_admin.initialize_app(cred, {
    'databaseURL': 'https://student-attendence-a9ddb-default-rtdb.asia-southeast1.firebasedatabase.app/'
})

# Constants
LOGIN_ID = "BDS_2023-27"
PASSWORD = "bds123456@"
students = [
    "AJAY THAKUR", "ANIRUDDHA DEY", "ARINDAM DAS", "KUSHAL DEY",
    "NISHA DAS", "SATYAKI CHAKRABORTY", "SNEHADHREE BANDYOPADHYAY",
    "SOUMYADEEP GHOSH", "SUBHADIP MITRA"
]

def is_connected():
    try:
        socket.create_connection(("8.8.8.8", 53), timeout=3)
        return True
    except OSError:
        return False

class LoginScreen(BoxLayout):
    def __init__(self, switch_to_attendance, **kwargs):
        super().__init__(orientation='vertical', **kwargs)
        self.switch_to_attendance = switch_to_attendance

        self.add_widget(Label(text="Login ID"))
        self.login_input = TextInput(multiline=False)
        self.add_widget(self.login_input)

        self.add_widget(Label(text="Password"))
        self.password_input = TextInput(password=True, multiline=False)
        self.add_widget(self.password_input)

        login_btn = Button(text="Login")
        login_btn.bind(on_press=self.validate_login)
        self.add_widget(login_btn)

    def validate_login(self, instance):
        now = datetime.now()
        if not is_connected():
            self.show_popup("No Internet", "Please connect to the internet.")
            return
        if now.hour < 9 or now.hour >= 17:
            self.show_popup("Access Denied", "Allowed only between 9 AM and 5 PM.")
            return
        if self.login_input.text == LOGIN_ID and self.password_input.text == PASSWORD:
            self.switch_to_attendance()
        else:
            self.show_popup("Login Failed", "Invalid ID or Password.")

    def show_popup(self, title, message):
        popup = Popup(title=title, content=Label(text=message), size_hint=(0.8, 0.4))
        popup.open()

class AttendanceScreen(BoxLayout):
    def __init__(self, **kwargs):
        super().__init__(orientation='vertical', **kwargs)
        self.checkboxes = {}

        self.teacher_input = TextInput(hint_text="Enter Teacher Name", multiline=False)
        self.class_input = TextInput(hint_text="Enter Number of Classes", multiline=False, input_filter='int')

        self.add_widget(Label(text="Teacher Name"))
        self.add_widget(self.teacher_input)

        self.add_widget(Label(text="Number of Classes"))
        self.add_widget(self.class_input)

        for student in students:
            h_layout = BoxLayout(size_hint_y=None, height=40)
            label = Label(text=student)
            checkbox = CheckBox()
            h_layout.add_widget(label)
            h_layout.add_widget(checkbox)
            self.add_widget(h_layout)
            self.checkboxes[student] = checkbox

        submit_btn = Button(text="Submit Attendance")
        submit_btn.bind(on_press=self.submit_attendance)
        self.add_widget(submit_btn)

    def submit_attendance(self, instance):
        teacher = self.teacher_input.text.strip()
        classes = self.class_input.text.strip()

        if not teacher or not classes:
            self.show_popup("Missing Info", "Please fill in all fields.")
            return

        now = datetime.now()
        date_str = now.strftime("%Y-%m-%d")
        time_str = now.strftime("%H:%M:%S")

        data = {
            "timestamp": f"{date_str} {time_str}",
            "teacher": teacher,
            "number_of_classes": classes,
            "students": {
                student: "Present" if cb.active else "Absent"
                for student, cb in self.checkboxes.items()
            }
        }

        try:
            # Firebase write
            ref = db.reference('attendance')
            ref.push(data)

            # Optional: REST API POST (correct Firebase REST path)
            try:
                requests.post(
                    "https://student-attendence-a9ddb-default-rtdb.asia-southeast1.firebasedatabase.app/attendance.json",
                    json=data
                )
            except Exception as req_err:
                print("HTTP Post Error:", req_err)

            self.show_popup("Success", "Attendance Submitted!")
        except Exception as e:
            self.show_popup("Error", str(e))
            return

        threading.Timer(2, self.close_app).start()

    def show_popup(self, title, message):
        popup = Popup(title=title, content=Label(text=message), size_hint=(0.8, 0.4))
        popup.open()

    def close_app(self):
        App.get_running_app().stop()

class AttendanceApp(App):
    def build(self):
        self.root_layout = BoxLayout(orientation='vertical')
        self.show_login()
        return self.root_layout

    def show_login(self):
        self.root_layout.clear_widgets()
        self.login_screen = LoginScreen(self.show_attendance)
        self.root_layout.add_widget(self.login_screen)

    def show_attendance(self):
        self.root_layout.clear_widgets()
        self.attendance_screen = AttendanceScreen()
        self.root_layout.add_widget(self.attendance_screen)

if __name__ == '__main__':
    AttendanceApp().run()

