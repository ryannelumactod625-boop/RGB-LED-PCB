# RGB-LED-PCB
### Scope and Project Objectives:  

a.	Designing, prototyping, and assembly of 3-5 PCBA prototypes that support 16 addressable RGBs, and two SMD buttons for cycling through colors/animations.  

b.	Firmware written for the commercial off-the-shelf (COTS) Seeeduino Xiao MG24 to support different light animations and to react to button presses.  

c.	Testing the prototypes to verify functionality, verify the firmware flow chart, and measure current draw. 

## 💡 NeoPixel RGB Stick

![GIF](https://content.arduino.cc/assets/animation.gif)  

## 📦 Installation

App Requirements to install in order to conduct project:
1.  Arduino IDE 2.3.10 -> writing code/software to upload into microcontroller 
2.  KiCad 10.0 -> designing PCB that contains all elements/components within the objectives
3.  Excel -> keep track of data taken for prototypes made (e.g. measuring current draw over different light animations from NeoPixel Sticks)
4.  Microsoft Word -> accessing Project Proposal to view Deliverable List, Design Boundaries, etc.

## 📖 Usage Guides

Follow this guide to installing Arduino IDE & to understanding the microcontroller (Seeeduino XIAO MG24) used in the project:

```bash
https://wiki.seeedstudio.com/xiao_mg24_getting_started/#xiao-mg24-front
```

Follow this YouTube tutorial on learning how to use KiCad 10.0:

```bash
https://youtu.be/3NSjzMN1xyc?si=KGY_VY7iNBNGU1tt
```

## 🛠 Prototypes

### Prototype 1: Seeeduino XIAO MG24 w/ one RGB LED
<img width="819" height="610" alt="image" src="https://github.com/user-attachments/assets/bbe9943c-0615-4fbf-8238-79af4c6e42d5" />

### Prototype 2: ELEGOO UNO R3 w/ 2x Adafruit NeoPixel RGB LED Sticks
<img width="831" height="596" alt="image" src="https://github.com/user-attachments/assets/812ba2ed-9b83-47a2-8cc9-6728ad90fe53" />


## 🧱 Schematic & Layout

### KiCad Schematic (Power System & Microcontroller):
<img width="1127" height="775" alt="image" src="https://github.com/user-attachments/assets/6a8c4ecd-00c0-4671-b2d4-2da442fec9f3" />

### KiCad PCB Layout:
<img width="900" height="682" alt="image" src="https://github.com/user-attachments/assets/79b475b7-cf4e-471c-a05b-bbd45e7c2953" />

### KiCad 3D Model:

<img width="856" height="893" alt="image" src="https://github.com/user-attachments/assets/dfd3e37f-79ea-4e52-a429-77bfd4843265" />


## 💻 Software (hypothetical usage w/ final PCB assembly)

```bash
// Simple demonstration on using an input device to trigger changes on your
// NeoPixels. Wire a momentary push button to connect from ground to a
// digital IO pin. When the button is pressed it will change to a new pixel
// animation. Initial state has all pixels off -- press the button once to
// start the first animation. As written, the button does not interrupt an
// animation in-progress, it works only when idle.

#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
#include <avr/power.h> // Required for 16 MHz Adafruit Trinket
#endif

// Digital IO pin connected to the button. This will be driven with a
// pull-up resistor so the switch pulls the pin to ground momentarily.
// On a high -> low transition the button press logic will execute.
#define BTN_NEXT  D2 //Button to go to next animation

#define BTN_PREV   D3 //Button to go to previous animation

#define LED  D0   // Digital IO pin connected to the NeoPixels.

#define PIXEL_COUNT 8  // Number of NeoPixels

#define DEBOUNCE_MS 200 //Debounce delay in milliseconds 

//----------------------------------------------------------------------------------------------

// Declare our NeoPixel strip object:
Adafruit_NeoPixel strip(PIXEL_COUNT, LED, NEO_GRB + NEO_KHZ800);
// Argument 1 = Number of pixels in NeoPixel strip
// Argument 2 = Arduino pin number (most are valid)
// Argument 3 = Pixel type flags, add together as needed:
//   NEO_KHZ800  800 KHz bitstream (most NeoPixel products w/WS2812 LEDs)

boolean oldState = HIGH;
int mode  = 0;    // Currently-active animation mode, 0-9
unsigned long lastPressNext = 0;
unsigned long lastPressPrev = 0;
const int totalModes = 8;

void setup() {
  pinMode(BTN_NEXT, INPUT_PULLUP);
  pinMode(BTN_PREV, INPUT_PULLUP);
  strip.begin(); // Initialize NeoPixel strip object (REQUIRED)
  strip.show();  // Initialize all pixels to 'off'
}

void loop() {
// ====== BUTTON HANDLING ======
  if (digitalRead(BTN_NEXT) == LOW && millis() - lastPressNext > DEBOUNCE_MS) {
    mode = (mode + 1) % totalModes;
    lastPressNext = millis();
  }
  if (digitalRead(BTN_PREV) == LOW && millis() - lastPressPrev > DEBOUNCE_MS) {
    mode = (mode - 1 + totalModes) % totalModes;
    lastPressPrev = millis();
  }
      
      switch(mode) {                                    // Start the new animation...
        case 0:
          colorWipe(strip.Color(  0,   0,   0), 50);    // Black/off (Idle)
          break;
        case 1:
          colorWipe(strip.Color(50, 50, 50), 50);       // White
          break;
        case 2:
          colorWipe(strip.Color(50, 0, 0), 50);         // Red
          break;
        case 3:
          colorWipe(strip.Color(0, 50, 0), 50);         // Green
          break;
        case 4:
          colorWipe(strip.Color(0, 0, 50), 50);         // Blue
          break;
        case 5:
          fourcorners(strip.Color(0, 50, 0));           //Four Corners Green 
          break;
        case 6:
          powersaver(strip.Color(50, 0, 0), strip.Color(0, 0, 50)); //Flashes Half Red, Half Blue at 5Hz
          break;
        case 7:
          lowbattery(50);                               //Double Blinks Red twice per sec
          break;
      }
    }
//----------------------------------------------------------------------------------------------

// Fill strip pixels one after another with a color. Strip is NOT cleared
// first; anything there will be covered pixel by pixel. Pass in color
// (as a single 'packed' 32-bit value, which you can get by calling
// strip.Color(red, green, blue) as shown in the loop() function above),
// and a delay time (in milliseconds) between pixels.

void colorWipe(uint32_t color, int wait) {
  for(int i=0; i<strip.numPixels(); i++) { // For each pixel in strip...
    strip.setPixelColor(i, color);         // Set pixel's color (in RAM)
    strip.show();                          // Update strip to match
    delay(wait);                           // Pause for a moment
  }
}
//----------------------------------------------------------------------------------------------
void fourcorners(uint32_t color) {
  for(int i=0; i<PIXEL_COUNT; i++){           // For each pixel in strip...
    strip.setPixelColor(i, 0);                // Set pixel's color (none)
    strip.show();                             // Update strip to match
  }
  for(int i=0; i<PIXEL_COUNT; i+=7) {         // Addressing Pixels 0 & 7 on both NeoPixel Sticks
    strip.setPixelColor(i, color);            // Set pixel's color (Green)
    strip.show();                             // Update strip to match
  }
}
//----------------------------------------------------------------------------------------------
// State variables for lowbattery
unsigned long lbStartTime = 0;
unsigned long lbLastStep = 0;
int lbStep = 0;

void lowbattery(uint8_t brightness) {
  unsigned long now = millis();

  if (lbStartTime == 0) {
    lbStartTime = now;
    lbLastStep = now;
    lbStep = 0;
  }

  // Pattern: ON 100ms, OFF 100ms, ON 100ms, OFF 500ms
  if (now - lbLastStep >= (lbStep % 2 == 0 ? 100 : (lbStep == 3 ? 500 : 100))) {
    lbLastStep = now;
    lbStep = (lbStep + 1) % 4;

    if (lbStep % 2 == 0) {
      // ON
      for (int i = 0; i < PIXEL_COUNT; i++) {
        strip.setPixelColor(i, strip.Color(brightness, 0, 0));
      }
    } else {
      // OFF
      for (int i = 0; i < PIXEL_COUNT; i++) {
        strip.setPixelColor(i, 0, 0, 0);
      }
    }
    strip.show();
  }
}

//----------------------------------------------------------------------------------------------
// State variables for powersaver
unsigned long psStartTime = 0;
unsigned long psLastToggle = 0;
bool psState = false;

void powersaver(uint32_t color1, uint32_t color2) {
  int half = PIXEL_COUNT / 2;
  unsigned long now = millis();

  // Start timer when entering mode
  if (psStartTime == 0) {
    psStartTime = now;
    psLastToggle = now;
    psState = false;
  }

  // Stop after 10 seconds
  if (now - psStartTime >= 10000) {
    psStartTime = 0; // Reset for next time
    return;
  }

  // Toggle every 100 ms (5 Hz)
  if (now - psLastToggle >= 100) {
    psLastToggle = now;
    psState = !psState;

    if (psState) {
      // First half: color1
      for (int i = 0; i < half; i++) {
        strip.setPixelColor(i, color1);
      }
      // Second half: color2
      for (int i = half; i < PIXEL_COUNT; i++) {
        strip.setPixelColor(i, color2);
      }
    } else {
      // All off
      for (int i = 0; i < PIXEL_COUNT; i++) {
        strip.setPixelColor(i, 0, 0, 0);
      }
    }
    strip.show();
  }
}
//----------------------------------------------------------------------------------------------
```

## 🤝 Contributions

- Open to any improvements/contributions to the software that would be uploaded to the Seeeduino XIAO MG24
- Open to suggestions to minimizing the budget to align with target specfications
- Open to suggestions to fitting the elements of the design within the design boundaries provided

### Target Budget Specifications:

a.	Target cost: Electrical BoM cost when pricing at 3,000 pcs/yr is not to exceed $25  

b.	Breadboard Prototype Budget: $30, some components will be provided (MG24 Xiao, 16x 5050 Addressable RGB LEDs)  

c.	PCBA Prototyping budget: $120  

### PCBA Design Boundaries: 

#### Footprint Boundary: 

<img width="732" height="668" alt="image" src="https://github.com/user-attachments/assets/7d0020f8-927c-4a63-b720-0ab5135eb787" />

#### Vertical Clearance/Keep-Out Zones:

<img width="975" height="306" alt="image" src="https://github.com/user-attachments/assets/795625c9-d448-4d7e-aadd-96b5671563d6" />

#### Battery Location Relative to PCBA:

<img width="975" height="293" alt="image" src="https://github.com/user-attachments/assets/c25fc3a3-0c46-4aee-a8b5-f7b430e113ae" />


## 📄 PCB Compliance

### PCB Compliance Documentation Requirements:
1. Fully annotated schematic with unique reference designators and assigned footprints to all components.
2. Run ERC (Electrical Rules Check) in Eeschema to catch wiring errors before layout.
3. Generate the BOM and save it in a standard format (CSV, Excel, or PDF) for assembly compliance.
4. Run DRC in the PCB editor to ensure manufacturability and safety compliance.
5. Export manufacturing files (component datasheets) and keep them alongside the BOM and DRC reports.
6. Compile into a compliance package: Combine the BOM, DRC log, netlist, and manufacturing files into a single document or folder for submission to manufacturers or regulatory bodies.

## 🙌 Credits

### My Project Team
Thank you to the following individuals who guided me throughout the project:

- Mechanical/Test Engineer - Michael Allen
- Director/Intern Manager - John Cassidy
- Approval Electrical Engineer - Jesus Duenas
- Approval Quality Engineer - Ruben Moreno 

- And to the Quality Team and to all of Fieldpiece who gave me a space and access to these tools to be able to work efficiently on this project. 
