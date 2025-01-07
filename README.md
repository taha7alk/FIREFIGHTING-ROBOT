# FIREFIGHTING-ROBOT

#include <Servo.h>  // include servo.h library

Servo myservo;

int pos = 0;
boolean fire = false;  // Flag to indicate fire detection and extinguishing mode

#define Left 12      // left sensor
#define Right 11    // right sensor
#define Forward 10   // front sensor

#define LM1 2       // left motor forward (PWM pin)
#define LM2 3       // left motor backward
#define RM1 4       // right motor forward (PWM pin)
#define RM2 5       // right motor backward

#define pump 8      // water pump

int motorSpeed = 90;  // Adjust this value between 0-255 for motor speed

void setup() {
  pinMode(Left, INPUT);
  pinMode(Right, INPUT);
  pinMode(Forward, INPUT);

  pinMode(LM1, OUTPUT);
  pinMode(LM2, OUTPUT);
  pinMode(RM1, OUTPUT);
  pinMode(RM2, OUTPUT);

  pinMode(pump, OUTPUT);

  myservo.attach(9);
  myservo.write(90);  // Initialize the servo to 90 degrees (neutral position)
}

void stop_motors() {
  // Function to stop all motors
  analogWrite(LM1, 0);  // Stop the left motor
  analogWrite(LM2, 0);
  analogWrite(RM1, 0);  // Stop the right motor
  analogWrite(RM2, 0);
}

void move_forward() {
  // Move the robot forward at reduced speed
  analogWrite(LM1, motorSpeed);  // Use PWM to set motor speed
  digitalWrite(LM2, LOW);
  analogWrite(RM1, motorSpeed);
  digitalWrite(RM2, LOW);
}

void turn_left() {
  // Turn the robot left at reduced speed
  analogWrite(LM1, motorSpeed);  // Left motor moves forward at reduced speed
  digitalWrite(LM2, LOW);
  digitalWrite(RM1, LOW);   // Right motor is off
  analogWrite(RM2, motorSpeed);  // Right motor moves backward
}

void turn_right() {
  // Turn the robot right at reduced speed
  analogWrite(LM1, motorSpeed);  // Left motor moves forward at reduced speed
  analogWrite(LM2, motorSpeed);  // Left motor moves backward
  digitalWrite(RM1, LOW);        // Right motor is off
  analogWrite(RM2, LOW);  // Right motor moves backward
}

void put_off_fire() {
  // Stop the robot before extinguishing the fire
  stop_motors();
  delay(500);

  // Activate the water pump
  digitalWrite(pump, HIGH);
  
  // Sweep the servo to simulate fire extinguishing
  for (pos = 40; pos <= 140; pos += 1) {
    myservo.write(pos); 
    delay(10);  
  }
  for (pos = 140; pos >= 40; pos -= 1) {
    myservo.write(pos); 
    delay(10);
  }

  // Turn off the water pump after the fire is extinguished
  digitalWrite(pump, LOW);
  myservo.write(90);  // Reset servo to neutral position

  fire = false;  // Fire has been extinguished, robot can resume movement
}

void loop() {
  if (!fire) { // Only move or turn if no fire is detected
    // If no fire is detected by any sensor, stop the robot
    if (digitalRead(Left) == 1 && digitalRead(Right) == 1 && digitalRead(Forward) == 1) {   
      stop_motors();
    }
    // If fire is detected straight ahead, move forward until close to the fire
    else if (digitalRead(Forward) == 0) {
      move_forward();  // Keep moving forward until close to the fire
      delay(500);      // Adjust this delay based on how close the robot should get
      stop_motors();   // Stop the motors when close to the fire
      fire = true;     // Set the fire flag to true and lock movement
    }   
    // If fire is detected to the left, turn left
    else if (digitalRead(Left) == 0) {
      turn_left();
    }
    // If fire is detected to the right, turn right
    else if (digitalRead(Right) == 0) {
      turn_right();
    }
  }

  // When fire is detected, extinguish it and disable further movement
  if (fire) {
    put_off_fire();  // Start the extinguishing process
    fire = false;    // Fire has been extinguished, allow movement in the next loop cycle
  }

  delay(300);  // Slow down the robot's movement and response
}
