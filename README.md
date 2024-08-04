import tkinter as tk
from tkinter import simpledialog, messagebox
from datetime import datetime, timedelta
import threading
import random

class Phone:
    def __init__(self, name, email, phone_number, wifi_network):
        self.battery_percentage = 100
        self.contacts = {}
        self.conversations = {}
        self.call_receipts = []
        self.current_time = datetime.now()
        self.user_name = name
        self.user_email = email
        self.user_phone_number = phone_number
        self.wifi_network = wifi_network
        self.wifi_connected = True

    def make_call(self, contact_or_number):
        self.call_receipts.append(f"Called {contact_or_number} at {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
        self.discharge_battery(5)

    def send_message(self, contact_or_number, message):
        if contact_or_number not in self.conversations:
            self.conversations[contact_or_number] = []
        self.conversations[contact_or_number].append((self.user_name, message))
        self.receive_message(contact_or_number, self.simple_response(message))
        self.discharge_battery(1)

    def receive_message(self, contact_or_number, message):
        if contact_or_number not in self.conversations:
            self.conversations[contact_or_number] = []
        self.conversations[contact_or_number].append((contact_or_number, message))

    def simple_response(self, message):
        responses = [
            "Got it!",
            "Thanks for the update.",
            "I'll get back to you.",
            "Understood.",
            "Okay."
        ]
        return random.choice(responses)

    def set_timer(self, hours=0, minutes=0, seconds=0):
        total_seconds = hours * 3600 + minutes * 60 + seconds
        threading.Timer(total_seconds, self.timer_finished).start()
        self.discharge_battery(1)

    def timer_finished(self):
        messagebox.showinfo("Timer", "Timer finished!")

    def add_contact(self, name, number):
        self.contacts[name] = number

    def discharge_battery(self, amount):
        self.battery_percentage = max(0, self.battery_percentage - amount)

    def about(self):
        return (f"Phone Simulator by Your Name\n"
                f"User Name: {self.user_name}\n"
                f"Email: {self.user_email}\n"
                f"Phone Number: {self.user_phone_number}")

    def show_conversation(self, contact_or_number):
        if contact_or_number in self.conversations:
            return self.conversations[contact_or_number]
        else:
            return "No conversation found"

    def view_call_receipts(self):
        return self.call_receipts


class PhoneApp:
    def __init__(self, root, phone):
        self.phone = phone
        self.root = root
        self.root.title("Phone Simulator")
        self.home_screen()

    def home_screen(self):
        self.clear_screen()

        self.home_label = tk.Label(self.root, text="Home Screen", font=("Helvetica", 24))
        self.home_label.pack(pady=20)

        self.clock_label = tk.Label(self.root, text="", font=("Helvetica", 18))
        self.clock_label.pack(pady=10)
        self.update_clock()

        self.call_button = tk.Button(self.root, text="Make a Call", command=self.make_call_ui)
        self.call_button.pack(pady=5)

        self.message_button = tk.Button(self.root, text="Send a Message", command=self.send_message_ui)
        self.message_button.pack(pady=5)

        self.timer_button = tk.Button(self.root, text="Set a Timer", command=self.set_timer_ui)
        self.timer_button.pack(pady=5)

        self.receipts_button = tk.Button(self.root, text="View Call Receipts", command=self.view_call_receipts_ui)
        self.receipts_button.pack(pady=5)

        self.about_button = tk.Button(self.root, text="About", command=self.about_ui)
        self.about_button.pack(pady=5)

    def update_clock(self):
        now = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
        self.clock_label.config(text=now)
        self.root.after(1000, self.update_clock)

    def make_call_ui(self):
        number = simpledialog.askstring("Make a Call", "Enter the phone number or contact name:")
        if number:
            self.phone.make_call(number)
            messagebox.showinfo("Call", f"Calling {number}...")

    def send_message_ui(self):
        contact = simpledialog.askstring("Send a Message", "Enter the phone number or contact name:")
        if contact:
            message = simpledialog.askstring("Send a Message", "Enter your message:")
            if message:
                self.phone.send_message(contact, message)
                messagebox.showinfo("Message", f"Message sent to {contact}: {message}")

    def set_timer_ui(self):
        hours = simpledialog.askinteger("Set Timer", "Enter hours:", minvalue=0, initialvalue=0)
        minutes = simpledialog.askinteger("Set Timer", "Enter minutes:", minvalue=0, initialvalue=0)
        seconds = simpledialog.askinteger("Set Timer", "Enter seconds:", minvalue=0, initialvalue=0)
        if hours is not None and minutes is not None and seconds is not None:
            self.phone.set_timer(hours, minutes, seconds)
            messagebox.showinfo("Timer", f"Timer set for {hours} hours, {minutes} minutes, and {seconds} seconds.")

    def view_call_receipts_ui(self):
        receipts = self.phone.view_call_receipts()
        if receipts:
            messagebox.showinfo("Call Receipts", "\n".join(receipts))
        else:
            messagebox.showinfo("Call Receipts", "No call receipts found.")

    def about_ui(self):
        about_info = self.phone.about()
        messagebox.showinfo("About", about_info)

    def clear_screen(self):
        for widget in self.root.winfo_children():
            widget.destroy()


def main():
    root = tk.Tk()
    name = "Alice"
    email = "alice@example.com"
    phone_number = "0987654321"
    wifi_network = "HomeNetwork"

    phone = Phone(name, email, phone_number, wifi_network)
    app = PhoneApp(root, phone)

    root.mainloop()

if __name__ == "__main__":
    main()

