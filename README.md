# Obstacle-Avoidance-for-Drone-
obstacle detection and Avoidance for drone using ultrasonic sensors


// Pins for Ultrasonic Sensor
#define trigPin 6 // Trigger Pin
#define echoPin 7 // Echo Pin

// Pins for Input Signals
#define THROTTLE_PIN 4
#define PITCH_PIN 3
#define YAW_PIN 5
#define ROLL_PIN 2

// PPM Signal Pin and Parameters
#define PPM_SIGNAL_PIN 9 // Pin to output PPM signal
#define PPM_FRAME_LENGTH 20000 // Total frame length in microseconds (20ms)
#define CHANNEL_COUNT 8 // Number of channels in the PPM frame
#define PPM_MIN 1000 // Minimum pulse width in microseconds (1ms)
#define PPM_MAX 2000 // Maximum pulse width in microseconds (2ms)
#define PPM_SYNC 300 // Sync pulse duration in microseconds (minimum gap between frames)

// Channel values
int channelValues[CHANNEL_COUNT] = {1500, 1500, 1500, 1500, 1500, 1500, 1500, 1500}; // Default values for 8 channels

void setup() {
  // Ultrasonic Sensor Setup
  Serial.begin(115200);      // Initialize Serial Monitor
  pinMode(trigPin, OUTPUT);  // Set TRIG as an output
  pinMode(echoPin, INPUT);   // Set ECHO as an input

  // Input Signals Setup
  pinMode(ROLL_PIN, INPUT);
  pinMode(PITCH_PIN, INPUT);
  pinMode(THROTTLE_PIN, INPUT);
  pinMode(YAW_PIN, INPUT);

  // Setup PPM Signal Pin
  pinMode(PPM_SIGNAL_PIN, OUTPUT);
  digitalWrite(PPM_SIGNAL_PIN, LOW); // Start with LOW signal
}

void loop() {
  // Ultrasonic Sensor Distance Measurement
  int distance = measureDistance();

  // Print Ultrasonic Sensor Distance
  Serial.print("Ultrasonic Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Input Signal Reading
  channelValues[0] = constrain(pulseIn(ROLL_PIN, HIGH), PPM_MIN, PPM_MAX);    // Roll
  channelValues[1] = constrain(pulseIn(PITCH_PIN, HIGH), PPM_MIN, PPM_MAX);   // Pitch
  channelValues[2] = constrain(pulseIn(THROTTLE_PIN, HIGH), PPM_MIN, PPM_MAX); // Throttle
  channelValues[3] = constrain(pulseIn(YAW_PIN, HIGH), PPM_MIN, PPM_MAX);     // Yaw

  // Neutralize pitch, roll, and yaw if obstacle is detected within 30 cm
  if (distance > 0 && distance < 30) {
    channelValues[1] = 1400; // Neutralize pitch value
    channelValues[0] = 1500; // Neutralize roll value
    channelValues[3] = 1500; // Neutralize yaw value
    Serial.println("Obstacle detected! Neutralizing pitch, roll, and yaw.");
  }

  // Print channel values
  Serial.print("Roll: ");
  Serial.println(channelValues[0]);
  Serial.print("Pitch: ");
  Serial.println(channelValues[1]);
  Serial.print("Throttle: ");
  Serial.println(channelValues[2]);
  Serial.print("Yaw: ");
  Serial.println(channelValues[3]);

  // Generate PPM Frame
  generatePPMFrame();

  delay(0); // Delay to match the PPM frame timing
}

// Function to measure distance using ultrasonic sensor
int measureDistance() {
  long duration;
  int distance;

  // Trigger ultrasonic sensor
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // Read echo and calculate distance
  duration = pulseIn(echoPin, HIGH, 30000); // Timeout for long distances
  if (duration == 0) {
    // Timeout or no object detected
    return -1;
  }
  distance = duration * 0.034 / 2; // Convert to cm
  return distance;
}

// Function to generate a PPM frame
void generatePPMFrame() {
  cli(); // Disable interrupts to ensure timing accuracy

  int totalPulseWidth = 0; // Track the total pulse width for sync calculation

  for (int i = 0; i < CHANNEL_COUNT; i++) {
    // Generate pulse for each channel
    int pulseWidth = constrain(channelValues[i], PPM_MIN, PPM_MAX);
    digitalWrite(PPM_SIGNAL_PIN, HIGH);
    delayMicroseconds(pulseWidth);
    digitalWrite(PPM_SIGNAL_PIN, LOW);
    delayMicroseconds(0); // Channel separation
    totalPulseWidth += pulseWidth + 0;
  }

  // Generate sync pulse
  int syncPulseWidth = PPM_FRAME_LENGTH - totalPulseWidth;
  if (syncPulseWidth > PPM_SYNC) {
    digitalWrite(PPM_SIGNAL_PIN, LOW);
    delayMicroseconds(syncPulseWidth);
  }

  sei(); // Re-enable interrupts
}

