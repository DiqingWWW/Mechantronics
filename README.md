# Mechantronics
latest code:
// This is a demonstration on how to use an input device to trigger changes on your neo pixels.
// You should wire a momentary push button to connect from ground to a digital IO pin.  When you
// press the button it will change to a new pixel animation.  Note that you need to press the
// button once to start the first animation!

#include <Adafruit_NeoPixel.h>

#define MOTION_PIN   2    // Digital IO pin connected to the button.  This will be
// driven with a pull-up resistor so the switch should
// pull the pin to ground momentarily.  On a high -> low
// transition the button press logic will execute.
#define PRESSURE_PIN1   A0
#define PIXEL_PIN    6    // Digital IO pin connected to the NeoPixels.

#define PIXEL_COUNT 60

// Parameter 1 = number of pixels in strip,  neopixel stick has 8
// Parameter 2 = pin number (most are valid)
// Parameter 3 = pixel type flags, add together as needed:
//   NEO_RGB     Pixels are wired for RGB bitstream
//   NEO_GRB     Pixels are wired for GRB bitstream, correct for neopixel stick
//   NEO_KHZ400  400 KHz bitstream (e.g. FLORA pixels)
//   NEO_KHZ800  800 KHz bitstream (e.g. High Density LED strip), correct for neopixel stick
Adafruit_NeoPixel strip = Adafruit_NeoPixel(PIXEL_COUNT, PIXEL_PIN, NEO_GRB + NEO_KHZ800);

//bool oldState = HIGH;
//int showType = 0;
unsigned long previousMillis = 0;  
const long interval = 1000;           // interval at which to blink (milliseconds)
  
int pressureValue1 = 0;

void setup() {
  Serial.begin(9600);
  pinMode(MOTION_PIN, INPUT_PULLUP);
  strip.begin();
  strip.show(); // Initialize all pixels to 'off'
}

void loop() {
  // Get current button state.
  //  bool newState = digitalRead(MOTION_PIN);
  //
  //  // Check if state changed from high to low (button press).
  //  if (newState == LOW && oldState == HIGH) {
  //    // Short delay to debounce button.
  //    delay(20);
  //    // Check if button is still low after debounce.
  //    newState = digitalRead(BUTTON_PIN);
  //    if (newState == LOW) {
  //      showType++;
  //      if (showType > 9)
  //        showType=0;
  //      startShow(showType);
  //    }
  //  }
 unsigned long currentMillis = millis();
  boolean PIRvalue = digitalRead(MOTION_PIN);
  if (PIRvalue == HIGH) {
    Serial.println("motion detected");
    rainbowCycle(1000);
    if (PIRvalue == LOW) {
      for (int i = 0; i < PIXEL_COUNT; i++) {
      strip.setPixelColor(i, 0);
      strip.show();
      delay(20);
      }
  } else {
    Serial.println("scanning");
    for (int i = 0; i < PIXEL_COUNT; i++) {
      strip.setPixelColor(i, 0);
      strip.show();
      delay(20);
    }
  }
  

  pressureValue1 = analogRead(PRESSURE_PIN1);
  Serial.print("pressuresensor = ");
  Serial.println(pressureValue1);
   for (int i = 0; i < PIXEL_COUNT; i++) {
      if (pressureValue1 < = 200) {
      strip.setPixelColor(i, 0);
      strip.show();
      delay(20);
          } else if (200 < pressureValue1 < 400){
          strip.setPixelColor(i, strip.Color(124,115,93));
      //strip.setPixelColor(i, strip.Color(249,249,222));
      strip.show();
      delay(20);
          }
       else if (400 < pressureValue1 < 600) {
      strip.setPixelColor(i, strip.Color(229,104,19));
      strip.show();
      delay(20);
        } else if (600 < pressureValue1 < 800) {
      strip.setPixelColor(i, strip.Color(252,202,76));
      strip.show();
      delay(20);
        } else if (pressureValue1 > 800) {
      strip.setPixelColor(i, strip.Color(249,249,222));
      strip.show();
      delay(20);
          }
        } 
      //delay(20);
   }  
  }





// Fill the dots one after the other with a color
void colorWipe(uint32_t c, uint8_t wait) {
  for (uint16_t i = 0; i < strip.numPixels(); i++) {
    strip.setPixelColor(i, c);
    strip.show();
    delay(wait);
  }
}

void rainbow(uint8_t wait) {
  uint16_t i, j;

  for (j = 0; j < 256; j++) {
    for (i = 0; i < strip.numPixels(); i++) {
      strip.setPixelColor(i, Wheel((i + j) & 255));
    }
    strip.show();
    delay(wait);
  }
}

// Slightly different, this makes the rainbow equally distributed throughout
void rainbowCycle(uint8_t wait) {
  uint16_t i, j;
  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= interval) {
    // save the last time you blinked the LED
    previousMillis = currentMillis;
     
     for (j = 0; j < 256 * 5; j++) { // 5 cycles of all colors on wheel
       for (i = 0; i < strip.numPixels(); i++) {
         strip.setPixelColor(i, Wheel(((i * 256 / strip.numPixels()) + j) & 255));
    }
    strip.show();

  }
 }
}

//Theatre-style crawling lights.
void theaterChase(uint32_t c, uint8_t wait) {
  for (int j = 0; j < 10; j++) { //do 10 cycles of chasing
    for (int q = 0; q < 3; q++) {
      for (int i = 0; i < strip.numPixels(); i = i + 3) {
        strip.setPixelColor(i + q, c);  //turn every third pixel on
      }
      strip.show();

      delay(wait);

      for (int i = 0; i < strip.numPixels(); i = i + 3) {
        strip.setPixelColor(i + q, 0);      //turn every third pixel off
      }
    }
  }
}

//Theatre-style crawling lights with rainbow effect
void theaterChaseRainbow(uint8_t wait) {
  for (int j = 0; j < 256; j++) {   // cycle all 256 colors in the wheel
    for (int q = 0; q < 3; q++) {
      for (int i = 0; i < strip.numPixels(); i = i + 3) {
        strip.setPixelColor(i + q, Wheel( (i + j) % 255)); //turn every third pixel on
      }
      strip.show();

      delay(wait);

      for (int i = 0; i < strip.numPixels(); i = i + 3) {
        strip.setPixelColor(i + q, 0);      //turn every third pixel off
      }
    }
  }
}

// Input a value 0 to 255 to get a color value.
// The colours are a transition r - g - b - back to r.
uint32_t Wheel(byte WheelPos) {
  WheelPos = 255 - WheelPos;
  if (WheelPos < 85) {
    return strip.Color(255 - WheelPos * 3, 0, WheelPos * 3);
  }
  if (WheelPos < 170) {
    WheelPos -= 85;
    return strip.Color(0, WheelPos * 3, 255 - WheelPos * 3);
  }
  WheelPos -= 170;
  return strip.Color(WheelPos * 3, 255 - WheelPos * 3, 0);
}
