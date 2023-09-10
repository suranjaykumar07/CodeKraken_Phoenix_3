# CodeKraken_Phoenix_3
import cv2
import face_recognition
import sqlite3
from PIL import Image


# Connect to the SQLite database or create one if it doesn't exist
conn = sqlite3.connect('people.db')
cursor = conn.cursor()

# Create a table to store people's information (if not already created)
cursor.execute('''CREATE TABLE employee
                  (name TEXT, dob TEXT, encoding BLOB)''')

# Capture a photo of the person
def capture_photo():
    # Open the camera
    cap = cv2.VideoCapture(0)

    while True:
        ret, frame = cap.read()
        cv2.imshow('Capture Photo', frame)

        # Capture the photo when the 'c' key is pressed
        if cv2.waitKey(1) & 0xFF == ord('c'):
            cv2.imwrite('captured_photo.jpg', frame)
            break

    cap.release()
    cv2.destroyAllWindows()

# Capture a photo of the person
capture_photo()

# Load the captured photo for face recognition
captured_image = face_recognition.load_image_file('captured_photo.jpg')
captured_face_encoding = face_recognition.face_encodings(captured_image)[0]

# Search for the face in the database
cursor.execute("SELECT name, dob FROM people WHERE encoding=?", (captured_face_encoding,))
result = cursor.fetchone()

if result:
    name, dob = result
    print(f"Person found: Name: {name}, DOB: {dob}")
else:
    print("Person not found in the database.")
    insert_new = input("Do you want to insert this person into the database? (yes/no): ")

    if insert_new.lower() == 'yes':
        name = input("Enter the person's name: ")
        dob = input("Enter the person's date of birth: ")

        # Insert the new person's information into the database
        cursor.execute("INSERT INTO people (name, dob, encoding) VALUES (?, ?, ?)",
                       (name, dob, captured_face_encoding))
        conn.commit()
        print("Person added to the database.")

# Close the connection
conn.close()
