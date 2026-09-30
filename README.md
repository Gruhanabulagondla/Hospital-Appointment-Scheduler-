import sqlite3
from datetime import datetime

DATABASE = "hospital_appointment.db"


# -------------------------------
# DATABASE CONNECTION
# -------------------------------
def connect():
    return sqlite3.connect(DATABASE)


# -------------------------------
# CREATE TABLES
# -------------------------------
def create_tables():
    conn = connect()
    cursor = conn.cursor()

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS doctors (
            doctor_id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            specialization TEXT NOT NULL,
            available_time TEXT NOT NULL
        )
    """)

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS patients (
            patient_id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            age INTEGER NOT NULL,
            phone TEXT NOT NULL
        )
    """)

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS appointments (
            appointment_id INTEGER PRIMARY KEY AUTOINCREMENT,
            patient_id INTEGER,
            doctor_id INTEGER,
            appointment_date TEXT NOT NULL,
            appointment_time TEXT NOT NULL,
            status TEXT DEFAULT 'Booked',
            FOREIGN KEY(patient_id) REFERENCES patients(patient_id),
            FOREIGN KEY(doctor_id) REFERENCES doctors(doctor_id)
        )
    """)

    conn.commit()
    conn.close()


# -------------------------------
# INSERT SAMPLE DOCTORS
# -------------------------------
def add_sample_doctors():
    conn = connect()
    cursor = conn.cursor()

    cursor.execute("SELECT COUNT(*) FROM doctors")
    count = cursor.fetchone()[0]

    if count == 0:
        doctors = [
            ("Dr. Ravi Kumar", "Cardiologist", "10:00 AM - 1:00 PM"),
            ("Dr. Priya Sharma", "Dermatologist", "2:00 PM - 5:00 PM"),
            ("Dr. Anil Reddy", "General Physician", "9:00 AM - 12:00 PM"),
            ("Dr. Sneha Rao", "Pediatrician", "11:00 AM - 2:00 PM")
        ]

        cursor.executemany("""
            INSERT INTO doctors
            (name, specialization, available_time)
            VALUES (?, ?, ?)
        """, doctors)

        conn.commit()

    conn.close()


# -------------------------------
# VIEW DOCTORS
# -------------------------------
def view_doctors():
    conn = connect()
    cursor = conn.cursor()

    cursor.execute("SELECT * FROM doctors")
    doctors = cursor.fetchall()

    print("\n--- Available Doctors ---")

    for doctor in doctors:
        print(
            f"ID: {doctor[0]} | "
            f"{doctor[1]} | "
            f"{doctor[2]} | "
            f"Available: {doctor[3]}"
        )

    conn.close()


# -------------------------------
# REGISTER PATIENT
# -------------------------------
def register_patient():
    name = input("Enter patient name: ")
    age = int(input("Enter age: "))
    phone = input("Enter phone number: ")

    conn = connect()
    cursor = conn.cursor()

    cursor.execute("""
        INSERT INTO patients (name, age, phone)
        VALUES (?, ?, ?)
    """, (name, age, phone))

    conn.commit()

    print("Patient registered successfully.")
    print("Patient ID:", cursor.lastrowid)

    conn.close()


# -------------------------------
# BOOK APPOINTMENT
# -------------------------------
def book_appointment():

    view_doctors()

    doctor_id = int(input("\nEnter Doctor ID: "))

    conn = connect()
    cursor = conn.cursor()

    cursor.execute(
        "SELECT * FROM doctors WHERE doctor_id = ?",
        (doctor_id,)
    )

    doctor = cursor.fetchone()

    if not doctor:
        print("Invalid Doctor ID.")
        conn.close()
        return

    patient_id = int(input("Enter Patient ID: "))

    cursor.execute(
        "SELECT * FROM patients WHERE patient_id = ?",
        (patient_id,)
    )

    patient = cursor.fetchone()

    if not patient:
        print("Invalid Patient ID.")
        conn.close()
        return

    appointment_date = input("Enter appointment date (DD-MM-YYYY): ")
    appointment_time = input("Enter appointment time: ")

    cursor.execute("""
        SELECT * FROM appointments
        WHERE doctor_id = ?
        AND appointment_date = ?
        AND appointment_time = ?
        AND status = 'Booked'
    """, (doctor_id, appointment_date, appointment_time))

    existing = cursor.fetchone()

    if existing:
        print("This time slot is already booked.")
        conn.close()
        return

    cursor.execute("""
        INSERT INTO appointments
        (patient_id, doctor_id, appointment_date, appointment_time)
        VALUES (?, ?, ?, ?)
    """, (
        patient_id,
        doctor_id,
        appointment_date,
        appointment_time
    ))

    conn.commit()

    print("\nAppointment booked successfully!")
    print("Appointment ID:", cursor.lastrowid)

    conn.close()


# -------------------------------
# VIEW APPOINTMENTS
# -------------------------------
def view_appointments():

    conn = connect()
    cursor = conn.cursor()

    cursor.execute("""
        SELECT
            appointments.appointment_id,
            patients.name,
            doctors.name,
            doctors.specialization,
            appointments.appointment_date,
            appointments.appointment_time,
            appointments.status
        FROM appointments
        JOIN patients
        ON appointments.patient_id = patients.patient_id
        JOIN doctors
        ON appointments.doctor_id = doctors.doctor_id
    """)

    appointments = cursor.fetchall()

    print("\n--- Appointments ---")

    if not appointments:
        print("No appointments found.")
    else:
        for appointment in appointments:
            print(
                f"ID: {appointment[0]} | "
                f"Patient: {appointment[1]} | "
                f"Doctor: {appointment[2]} | "
                f"Specialization: {appointment[3]} | "
                f"Date: {appointment[4]} | "
                f"Time: {appointment[5]} | "
                f"Status: {appointment[6]}"
            )

    conn.close()


# -------------------------------
# SEARCH APPOINTMENT
# -------------------------------
def search_appointment():

    appointment_id = int(input("Enter Appointment ID: "))

    conn = connect()
    cursor = conn.cursor()

    cursor.execute("""
        SELECT
            appointments.appointment_id,
            patients.name,
            doctors.name,
            doctors.specialization,
            appointments.appointment_date,
            appointments.appointment_time,
            appointments.status
        FROM appointments
        JOIN patients
        ON appointments.patient_id = patients.patient_id
        JOIN doctors
        ON appointments.doctor_id = doctors.doctor_id
        WHERE appointments.appointment_id = ?
    """, (appointment_id,))

    appointment = cursor.fetchone()

    if appointment:
        print("\nAppointment Found")
        print("Appointment ID:", appointment[0])
        print("Patient:", appointment[1])
        print("Doctor:", appointment[2])
        print("Specialization:", appointment[3])
        print("Date:", appointment[4])
        print("Time:", appointment[5])
        print("Status:", appointment[6])
    else:
        print("Appointment not found.")

    conn.close()


# -------------------------------
# CANCEL APPOINTMENT
# -------------------------------
def cancel_appointment():

    appointment_id = int(input("Enter Appointment ID: "))

    conn = connect()
    cursor = conn.cursor()

    cursor.execute("""
        UPDATE appointments
        SET status = 'Cancelled'
        WHERE appointment_id = ?
    """, (appointment_id,))

    conn.commit()

    if cursor.rowcount > 0:
        print("Appointment cancelled successfully.")
    else:
        print("Appointment not found.")

    conn.close()


# -------------------------------
# MAIN MENU
# -------------------------------
def main():

    create_tables()
    add_sample_doctors()

    while True:

        print("\n================================")
        print("  HOSPITAL APPOINTMENT SCHEDULER")
        print("================================")
        print("1. View Doctors")
        print("2. Register Patient")
        print("3. Book Appointment")
        print("4. View Appointments")
        print("5. Search Appointment")
        print("6. Cancel Appointment")
        print("7. Exit")

        choice = input("Enter your choice: ")

        if choice == "1":
            view_doctors()

        elif choice == "2":
            register_patient()

        elif choice == "3":
            book_appointment()

        elif choice == "4":
            view_appointments()

        elif choice == "5":
            search_appointment()

        elif choice == "6":
            cancel_appointment()

        elif choice == "7":
            print("Thank you for using Hospital Appointment Scheduler!")
            break

        else:
            print("Invalid choice. Please try again.")


main()
