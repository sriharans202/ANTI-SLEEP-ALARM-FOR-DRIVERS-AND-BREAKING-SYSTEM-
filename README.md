# ANTI-SLEEP-ALARM-FOR-DRIVERS-AND-BREAKING-SYSTEM-
import cv2
import dlib
from scipy.spatial import distance

# Function to compute the eye aspect ratio (EAR)
def calculate_EAR(eye):
    A = distance.euclidean(eye[1], eye[5])
    B = distance.euclidean(eye[2], eye[4])
    C = distance.euclidean(eye[0], eye[3])
    ear = (A + B) / (2.0 * C)
    return ear

# Thresholds
EYE_AR_THRESH = 0.25  # Threshold for EAR
EYE_AR_CONSEC_FRAMES = 20  # Number of consecutive frames the eye must be below the threshold to sound alarm

# Load the face detector and landmark predictor
detector = dlib.get_frontal_face_detector()
predictor = dlib.shape_predictor('shape_predictor_68_face_landmarks.dat')

# Indexes for the left and right eye landmarks
(lStart, lEnd) = (42, 48)
(rStart, rEnd) = (36, 42)

cap = cv2.VideoCapture(0)
frame_counter = 0

while True:
    ret, frame = cap.read()
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    rects = detector(gray, 0)

    for rect in rects:
        shape = predictor(gray, rect)
        shape = shape_to_np(shape)

        leftEye = shape[lStart:lEnd]
        rightEye = shape[rStart:rEnd]

        leftEAR = calculate_EAR(leftEye)
        rightEAR = calculate_EAR(rightEye)

        avg_EAR = (leftEAR + rightEAR) / 2.0

        if avg_EAR < EYE_AR_THRESH:
            frame_counter += 1
            if frame_counter >= EYE_AR_CONSEC_FRAMES:
                cv2.putText(frame, "DROWSINESS ALERT!", (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 0, 255), 2)
                # Trigger alarm here (buzzer or speaker)
        else:
            frame_counter = 0

    cv2.imshow("Frame", frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
#define trigPin 9
#define echoPin 10
#define motorPin 3  // Motor connected to pin 3

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(motorPin, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  long duration, distance;
  
  // Send the ultrasonic pulse
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  // Measure the pulse duration
  duration = pulseIn(echoPin, HIGH);
  
  // Calculate the distance
  distance = (duration / 2) / 29.1;
  
  // Display the distance
  Serial.print("Distance: ");
  Serial.println(distance);
  
  // Brake when an obstacle is within a certain distance
  if (distance <= 20) {
    analogWrite(motorPin, 0);  // Stop the motor (apply brake)
    Serial.println("Braking...");
  } else {
    analogWrite(motorPin, 255);  // Normal operation (motor running)
  }
  
  delay(100);
}
