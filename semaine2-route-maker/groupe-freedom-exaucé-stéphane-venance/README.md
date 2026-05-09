// Project 2: Automated mini-monitoring station
#include <Servo.h>
// We import Servo.h because controlling a servo requires PWM signals with
// specific timing that analogWrite() cannot handle — the library manages this automatically.

///// Pins definition
const int redPin = 9;         // Red channel of RGB LED
const int greenPin = 10;      // Green channel of RGB LED
const int bluePin = 11;       // Blue channel of RGB LED (defined but kept OFF in this project)
const int buzzer = 8;         // Piezoelectric buzzer
const int ldrPin = A0;        // LDR light sensor
const int servo_motorPin = 6; // Servo motor signal pin
const int potent = A1;        // Potentiometer for threshold adjustment

Servo myServo; // We create a Servo object to use the library's controlled write methods

void setup() {
  // We set these pins as OUTPUT because they power components (LED, buzzer)
  // rather than reading from them
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);   // Must be declared even if unused, to avoid floating state
  pinMode(buzzer, OUTPUT);

  // Serial communication lets us monitor sensor values in real time for debugging
  Serial.begin(9600);

  // We attach the servo to its pin so the library knows which pin to send PWM signals to
  myServo.attach(servo_motorPin);
}

void loop() {
  // We use analogRead() here because both the LDR and potentiometer output a
  // variable voltage (0–5V), not just HIGH or LOW — analogRead converts this to 0–1023
  int lightValue = analogRead(ldrPin);  // Current light intensity from LDR
  int limit = analogRead(potent);       // User-defined alert threshold via potentiometer

  // println() is used (not print) so each reading appears on its own line in Serial Monitor
  Serial.print("Light: ");
  Serial.print(lightValue);
  Serial.print(" | Limit: ");
  Serial.println(limit);

  // =====================
  // Alert condition
  // We compare lightValue to limit: if light drops below the threshold, it signals
  // an anomaly (e.g. something blocking the sensor), triggering all alert outputs
  if (lightValue < limit) {

    // Red = alert state. We turn green OFF to avoid color mixing on the RGB LED
    digitalWrite(redPin, HIGH);
    digitalWrite(greenPin, LOW);

    // tone() generates a square wave at 1000Hz — we use it instead of digitalWrite
    // because the buzzer needs an oscillating signal to produce sound, not just voltage
    tone(buzzer, 1000);

    // We write 90° because the servo's mechanical range is 0–180°,
    // and 90° gives a clear, visible physical signal of the alert state
    myServo.write(90);

  } else {

    // Green = normal/standby state
    digitalWrite(redPin, LOW);
    digitalWrite(greenPin, HIGH);

    // noTone() stops the PWM signal to the buzzer — without it, the buzzer keeps ringing
    noTone(buzzer);

    // Return servo to rest position when system resets to normal
    myServo.write(0);
  }

  // A short delay stabilizes analogRead() values and avoids flooding Serial Monitor
  delay(200);
}
