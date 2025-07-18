# IR-blaster-M5StickC-PLUS-ESP32
IR remote control in Arduino with M5StickC PLUS ESP32-PICO Mini


## What you need:

1. M5StickC Plus

<img src="https://github.com/enricocid/IR-blaster-M5StickC-PLUS-ESP32/blob/main/m5.png" width=20% height=20%>

2. [Ky-005](https://www.az-delivery.de/en/products/ir-sende-modul) IR infrared transmitter transceiver module
<img src="https://www.az-delivery.de/cdn/shop/products/ky-005-ir-infrarot-sender-transceiver-modul-906607.jpg" width=20% height=20%>


3. Arduino IDE
   - [Download it here](https://www.arduino.cc/en/software/)
   - Follow the official setup guide: [Arduino setup for M5StickC Plus](https://docs.m5stack.com/en/arduino/m5stickc_plus/program)
   - Make sure to install **IRremoteESP8266**


```
#include <M5StickCPlus.h>
#include <IRremoteESP8266.h>
#include <IRsend.h>


// Default delay to use in most cases
const uint32_t kDelayMs = 100;

// IR emitter Pin
const uint16_t kSendPin = 26;

// IRemote class for sending
IRsend irsend(kSendPin);

// Code to send

// code for LG Power: 0x0020DF10EF

// Code for Xtreamer
// power: 0x0000FF20DF, Play/pause: 0x0000FF32CD

// Variables to control delay running
// run sending nec signal for 3000ms (3s)
bool delayRunning = false;
unsigned long delayStart = 0;

// Delay for printing
const uint32_t pDelayMs = 3000;

// Function to print coloured screen messages
void print_screen(uint32_t color = BLACK, String msg = " Sending...\n 00FF20DF\n :=]", uint32_t text_color = WHITE) {
  M5.Lcd.setTextColor(text_color, color);
  delay(250);
  M5.Lcd.fillScreen(color);
  M5.Lcd.setCursor(0, 50);
  M5.Lcd.setTextSize(2);
  M5.Lcd.println(msg);
}

void setup() {
  // Initialize with LCD and Power, but no Serial
  M5.begin(true, true, false);
  
  // Show a msg when the device is ready
  print_screen(GREEN, " Ready =]", BLACK);
  delay(2000);

  // Turn off display
  M5.Axp.SetLDO2(false);
  M5.Lcd.fillScreen(false);
  // Initialize the IR sender
  irsend.begin();
}

// Function to send IR signal (loop for 3s)
void send_nec(uint64_t ir_code, bool timer = true) {

  if (timer) {
    delayStart = millis(); // start delay
    delayRunning = true;   // not finished yet
    // check if delay has timed out after 5s == 5000ms
    while (delayRunning && ((millis() - delayStart) <= pDelayMs)) {
      irsend.sendNEC(ir_code);
      delay(1000);  // Delay must be greater than 5 ms, otherwise the receiver sees it as a long signal
    }
    // finished
    delayRunning = false;
  } else {
    irsend.sendNEC(ir_code);
  }
}

void loop() {
  // Update the buttons state
  M5.update();

  // if button A is pressed for 3s, shut off the M5StickC!
  if (M5.BtnA.pressedFor(3000)) {
    M5.Axp.SetLDO2(true);
    print_screen(DARKGREY, " OFF...", BLACK);
    delay(pDelayMs);
    M5.Axp.PowerOff();
  } else {
    // If "A" was pressed, send the code to the device via IR transmitter
    if (M5.BtnA.wasReleased()) {
      print_screen();
      M5.Axp.SetLDO2(true);
      send_nec(0x0000FF20DF);
      M5.Axp.SetLDO2(false);
      M5.Lcd.fillScreen(BLACK);
    }
  }

  // Use button B to trigger different commands
  if (M5.BtnB.wasReleased()) {
    M5.Axp.SetLDO2(true);
    print_screen(ORANGE, " Play/Pause\n :)", BLACK);
    send_nec(0x0000FF32CD, false);
    delay(pDelayMs);
    M5.Axp.SetLDO2(false);
    M5.Lcd.fillScreen(BLACK);
  }

  delay(kDelayMs);
}
```
