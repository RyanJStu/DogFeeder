/* Written by Ryan Jones for the DIY Auto Dog Feeder
   It utilizes an Arduino Uno, a generic cereal dispenser,
   a stepper motor, Adafruit Motorshield V2, LCD, and 1307RTC
*/
#include <SPI.h>
#include <Wire.h>
#include <Adafruit_MotorShield.h>
#include "utility/Adafruit_MS_PWMServoDriver.h"
#include "RTClib.h"
#include <Bounce2.h>

RTC_DS1307 rtc;

Adafruit_MotorShield AFMS = Adafruit_MotorShield();
Adafruit_StepperMotor *myMotor = AFMS.getStepper(200, 1);//200 steps, port 1

//Interface stuff
int button = 2; //Button attached to pin 2 for LEDs
int buttonState = 0; //initial state of switch, low
int prevSwitchState = 0; //stores current button state
static int hits = 0; //stores presses
Bounce debouncer1 = Bounce(); //use Bounce2 library for button

int amLED = 13; //morning feeder status LED
int pmLED = 4; //evening feeder status LED

bool amFeeder = false;
bool pmFeeder = false;

//how we keep track of the weeks, not that it matters here
char daysOfTheWeek[7][12] = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"};

void setup() { //Set things up

  //Inputs
  pinMode(button, INPUT_PULLUP); //utilize internal pullup
  debouncer1.attach(button); //use button for debounce
  debouncer1.interval(20); //20ms debounce

  //Outputs
  pinMode(amLED, OUTPUT);
  pinMode(pmLED, OUTPUT);

  //Motor Stuff
  Serial.begin(57600);
  AFMS.begin();
  myMotor->setSpeed(30);  //speed = 30 rpm

  //RTC STUFF
  while (!Serial) ; // wait for Arduino Serial Monitor
  if (! rtc.begin()) {
    Serial.println("Couldn't find RTC");
    while (1);
  }

  if (! rtc.isrunning()) {
    Serial.println("RTC is NOT running!");
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__))); //stores date and time of upload to RTC
    // rtc.adjust(DateTime(2014, 1, 21, 3, 0, 0));
    // uncomment above line for manual date/time entry
  }
}

void loop() {

  //Give us the time and let's look at it
  //in the serial monitor to help
  DateTime now = rtc.now();
  Serial.print(now.hour(), DEC);
  Serial.print(':');
  Serial.print(now.minute(), DEC);
  Serial.print(':');
  Serial.print(now.second(), DEC);
  Serial.println();

  //Get button ready
  buttonState = digitalRead(button); //let's read the button state
  debouncer1.update(); //let debounce begin!
  int press = debouncer1.read(); //read buttonState, name it "press"


  //Trigger the dog feeder at specific times, daily
  if (now.hour() == 8 && now.minute() == 0 && now.second() == 0) { //if 8:0:0 am
    DogFeederAM(); //jump to am feeder loop
  }

  if (now.hour() == 18 && now.minute() == 0 && now.second() == 0) { //if 6:0:0 pm
    DogFeederPM(); //jump to evening feeder loop
  }

  if (now.hour() == 23 && now.minute() == 59 && now.second() == 59) { //if about midnight
    Reset(); //jump to reset
  }

  //Manual status LED button control
  if (press == LOW) { //if button pressed...
    if (amFeeder == false && pmFeeder == false) { //...and if autofeed hasn't happened
      digitalWrite(amLED, HIGH); //turn on morning LED
      amFeeder = true;
    } else if (amFeeder && pmFeeder == false) { //...and if only morning feed happened
      digitalWrite(pmLED, HIGH); //turn on evening LED
      pmFeeder = true;
    } else if (amFeeder && pmFeeder) { //...and if both autofeeds have occured
      digitalWrite(amLED, LOW); //turn off morning LED
      digitalWrite(pmLED, LOW); //turn off evening LED
      amFeeder = false; 
      pmFeeder = false;
      hits = 0; //reset hits count
    }
  }

  /* //Button stuff in case of manual feed
    if (buttonState != prevSwitchState) {
     if (press == LOW) { //if button pressed
       hits = hits + 1; //count one more press
     }
     prevSwitchState = buttonState; //store the state of hits
    }


    if (hits == 1) { //first button press
     Serial.println("morning LED");
     digitalWrite(amLED, HIGH); //
     amFeeder = true;
    }

    if (hits == 2) { //second button press
     Serial.println("evening LED");
     digitalWrite(pmLED, HIGH);
     pmFeeder = true;
    }

    if (hits == 3) { //third press resets LEDs and hit count
     Serial.println("turn LEDs off");
     digitalWrite(amLED, LOW);
     digitalWrite(pmLED, LOW);
     hits = 0; //reset the hit count
    } */
    
} //end loop


//Morning Dog Feed
void DogFeederAM() {
  Serial.println("dog feed AM");
  digitalWrite(amLED, HIGH);
  myMotor->step(500, FORWARD, DOUBLE); //move stepper 500 steps forward
  amFeeder = true;
}

//Evening Dog Feed
void DogFeederPM() {
  Serial.println("dog feed PM");
  digitalWrite(amLED, HIGH);
  myMotor->step(500, FORWARD, DOUBLE); //move stepper 500 steps forward
  pmFeeder = true;
}

//Reset at midnight
void Reset() { //reset everything to default state
  digitalWrite(amLED, LOW);
  digitalWrite(pmLED, LOW);
  amFeeder = false;
  pmFeeder = false;
  hits = 0;
}
