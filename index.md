# Time-Zone-Alterable Word Clock
A wall-mountable clock that displays words which correspond with the current time. These words are shown through a laser-cut sheet and LED lighting behind it, with select words being illuminated at a time. It is electronically to a globe, on which one may move a token around the surface to set a time zone for the clock. The token attaches magnetically, with a magnetic follower inside of the globe. Electronics are situated inside of the globe and calculate the position of the token, conveying that data to set the clock's time zone.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Ellie L | Santa Teresa High School | TBD | Incoming Junior

![Headstone Image](logo.svg)
  
# Final Milestone

<iframe width="870" height="457.5" src="https://www.youtube.com/embed/-jb-2cFIigA?si=iRqVPUFcU7uKYvGv" title="Final Milestone Youtube Video" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE

# Second Milestone

<iframe width="870" height="457.5" src="https://www.youtube.com/embed/N8rfFMciLm0?si=YuZB1L0fa5mSrYMO" title="Milestone 2 Youtube Video" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 

# First Milestone

<iframe width="870" height="457.5" src="https://www.youtube.com/embed/PoXtgeYFXZ4?si=KZ8p7ZyIlwunwfTO" title="Milestone 1 Youtube Video" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

My first milestone consists of the entire word clock portion of the project, which took me well over half of the progrma to build. Using hand-sawed acrylic for the backing and sides, as well as a clouded acrylic sheet for the front panel, the majority of the structure and frame are assembled with hot glue and superglue. Inside of the square frame, 201 individual LEDs are daisy chained in series, each with a power and ground wire as well as a data wire that runs from one to the other. The LEDs are cut from a typical individually addressable LED strip, and I soldered each one according to the spacing of letters that I designed on my grid. There is a cardstock grid that isolates each LED from the other, allowing for them to light up individually without the light bleeding over.

In order to make the letter grid, I planned out the most space-efficient option for displaying the time, which included optimizations such as having one constant display of "twenty" for all corresponding minute values, and having numbers such as "fourteen" also double as "four" and "twenty four." Having tihs structure in mind, I wrote out a grid of letters and transferred them over to Canva for more precise measurement and font. I downloaded this design and uploaded it to a Cricut workspace in order to laser cut them out of a 12 inch by 12 inch black cardstock sheet.

As each LED is wired in series, a burnt out or malfunctioned LED along the chain can and will cause all succeeding LEDs to stop responding or dim. To debug the chain as I soldered, I used a multimeter to check for the proper voltage and used a mode on the multimeter to trigger a beeping sound confirming a connection through the data wire. Through this process, I was able to replace LEDs accordingly and light up the entire grid. Also due to the fact that they are wired in series,  power is injected in two additional locations along the chain to prevent a major voltage drop that would cause inconsistency and dimming. In future iterations, I may experiment with parallel wiring for power, even if data must be wired in series.

The display can be powered by plugging a cord into a typical wall outlet, as the specific cord steps the voltage down to 9V, and a linear voltage regulator on the breadboard steps the voltage down further from 9V to 5V, which can power both the Arduino Nano ESP32 and the LED grid. To program the display, I used the FastLED and Time libraries, which have the tools or functions necessary to sync the time over WiFi and allow those to correspond with pin IDs and ranges. Said IDs and ranges are stored in matrices within the code for easy access and to make the code cleaner to read.

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code

The entirety of the project's code, which is commented to show the purpose and functionality of individual lines/sections:

## Main C++ Code

```c++
//necessary libararies
#include<FastLED.h>
#include<Arduino.h>
#include<TimeLib.h>
#include<WiFi.h>
//needs to be included because C++ doesn't automatically include all tools, but to make things faster only includes them if you explicitly say you need them
#include<unordered_map>
#include<string>

//these are the network credentials, replace with WiFi SSID and password
const char* SSID = "WiFi SSID Here";
const char* password = "WiFi Password Here";

//uses pool.ntp.org which is a public bunch of global timeservers
const char* ntpServer = "pool.ntp.org";

//both of these are in seconds
//default timezone offset for San Jose
long gmtOffset = -8*3600;
//and then offset for daylight savings time
long daylightOffset = 3600;

//assorted variables
int hours;
int minutes;
int i = 0;
int reedSwitch = 0;
int reedSwitchLast = 0;

int startBrightness = 0;

int rotateInt1 = 8;
int rotateInt2 = 7;
int rotateInt3 = 6;

int colorNum = 0;
int colorOrderNum = 1;

int id1;
int id2;
int id3;

//configurations for the hours start IDs and ranges
int minConfigs[31][2] = {
  {0, 0},
  {66, 3}, {81, 3}, {93, 5}, {84, 4}, {77, 4},
  {137, 3}, {102, 5}, {73, 5}, {113, 4}, {14, 3},
  {17, 6}, {22, 6}, {47, 8}, {84, 8}, {34, 7},
  {133, 7}, {98, 9}, {70, 8}, {113, 8}, {56, 6},
  {66, 3}, {81, 3}, {93, 5}, {84, 4}, {77, 4},
  {137, 3}, {102, 5}, {73, 5}, {113, 4}, {56, 6}
 };

int countryColors[9][9] = {
  {179, 25, 66, 10, 49, 97, 255, 255, 255},
  {200, 16, 46, 218, 37, 29, 255, 205, 0},
  {179, 25, 66, 10, 49, 97, 255, 255, 255},
  {30, 181, 58, 252, 209, 22, 0, 163, 221},
  {0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 0, 0, 0, 0, 0, 0, 0, 0},
  {0, 0, 0, 0, 0, 0, 0, 0, 0}};

/*
locations:
A3 --> San Jose, CA, 1 top
A4 --> Saigon, Vietnam, 3 bottom
A5 --> New York, NY, 3 top
A6 --> Arusha, Tanzania, 2 bottom
A7 --> Sydney, Australia, 2 top
D2 --> Alesund, Norway, 4 bottom
D3 --> Istanbul, Turkiye, none
D4 --> Cusco, Peru, 1 bottom
D5 --> Tokyo, Japan
*/
//first value of element is pin that is being read, second is gmtOffset
std::unordered_map<uint8_t, int> timezones = 
  {{A3, -8*3600}, {A4, 3600*7}, {A5, -3600*5}, {A6, 3600*3}, {A7, 3600*11}, 
  {D2, 3600}, {D3, 3600*3}, {D4, -3600*5}, {D5, -3600*9}};

//configurations for the hours start IDs and ranges
int hourConfigs[13][2] = {
  {148, 6},
  //^^added this since I forgot midnight is technically 0
  {176, 3}, {154, 3}, {163, 5}, {143, 4}, {192, 4},
  {179, 3}, {168, 5}, {159, 5}, {172, 4}, {157, 3},
  {187, 6}, {148, 6} 
 };


//lists for the initialization sequences
int rowVals[14] =  
  {14*0, 14*1, 14*2, 14*3, 14*4,
  14*5, 14*6, 14*7, 14*8, 14*9,
  14*10, 14*11, 14*12, 14*13};

int rotateVals[54] = 
  {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13,
  14, 41, 42, 69, 70, 97, 98, 125, 126, 153, 154, 181, 182,
  183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195,
  168, 167, 140, 139, 112, 111, 84, 83, 56, 55, 28, 27};

//constants for the LEDs
#define NUM_LEDS 201
#define DATA_PIN 2
#define LED_TYPE WS2812B
#define COLOR_ORDER GRB

//sets number of LEDs
CRGB leds[NUM_LEDS];

//code here runs once at the very start
void setup() {
  //FastLED initialization
  FastLED.addLeds<LED_TYPE, DATA_PIN, COLOR_ORDER>(leds, NUM_LEDS);

  Serial.begin(115200);

  //connect to the wifi with the credentials
  WiFi.begin(SSID, password);

  //stalls while the connection to WiFi is being made
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.print("connected sucessfully");

  //configurate pins for the reed switches
  pinMode(A2, OUTPUT);
  for (auto& [pin, offset] : timezones) {
    pinMode(pin, INPUT_PULLUP);
  }

  //initialization sequence part where the white LEDs join in rows from the top and bottom to the middle
  //cycles through brightness and rows
  FastLED.setBrightness(0);
  for (int i = 0; i < 7; ++i) {
    fill_solid(&leds[rowVals[i]], 14, CRGB::White);
    fill_solid(&leds[rowVals[13-i]], 14, CRGB::White);
    for (int b = 1; b < 3; ++b) {
      startBrightness += b;
      FastLED.setBrightness(startBrightness);
      FastLED.show();
      delay(20);
    }
  }

  //red rotation sequence for initialization, basically just lights up in a circle
  //there's a dedicated unordered map for the IDs correspnding to this sequence
  for (int i = 0; i < 54; ++i) {
    leds[rotateVals[rotateInt1]] = CRGB::Red;
    leds[rotateVals[rotateInt2]] = CRGB(181, 0, 0);
    leds[rotateVals[rotateInt3]] = CRGB(102, 0, 0);
    if (rotateInt1 == 53) {
      rotateInt1 = 0;
    } else {
      rotateInt1 += 1;
    }
    if (rotateInt2 == 53) {
      rotateInt2 = 0;
    } else {
      rotateInt2 += 1;
    }
    if (rotateInt3 == 53) {
      rotateInt3 = 0;
    } else {
      rotateInt3 += 1;
    }
    FastLED.show();
    delay(20);
  }

  //makes the initialization LED sequence fade out slowly
  for (int b = startBrightness; b > 0; b -= 2) {
    FastLED.setBrightness(b);
    FastLED.show();
    delay(20);
  }

  //set default brightness
  FastLED.setBrightness(150);
}

//defines function setTime which takes parameters to set the RGB values of the time
void setTime(int red, int green, int blue) {

  //lights up all constants
  //"Hello!" "it" "is" "made by Ellie Le =D"
  fill_solid(&leds[0], 6, CRGB(red, green, blue));
  fill_solid(&leds[7], 2, CRGB(red, green, blue));
  fill_solid(&leds[10], 2, CRGB(red, green, blue));
  fill_solid(&leds[196], 5, CRGB(red, green, blue));

  //conditionals to check if we're in the first or second half of the hour
  if (minutes <= 30 && minutes != 0) {
    //illuminate "past"
    fill_solid(&leds[127], 4, CRGB(red, green, blue));
  } else if (minutes > 30) {
    //illuminate "to"
    minutes = 60 - minutes;
    fill_solid(&leds[140], 2, CRGB(red, green, blue));
    //adds to hours because of the way it's read (i.e. it is 2:54 is 6 to 3, not 6 to 2)
    hours += 1;
  }

  
  //light up am and pm based on which half of the day it is (i.e. if the hours are greater than 12)
  if (hours >= 12) {
    hours = (12 - hours)*-1;
    fill_solid(&leds[182], 2, CRGB(red, green, blue));
  } else if (hours < 12 || hours == 0) {
    fill_solid(&leds[184], 2, CRGB(red, green, blue));
  }


  if (minutes >= 20 && minutes != 30) {
    //lights up "twenty" if the minute is any value in the twenties
    fill_solid(&leds[56], 6, CRGB(red, green, blue));
  }

  if (minutes == 15 || minutes == 30) {
    // light up "a" when the minute is 15 or 30 for "a half" and "a quarter"
    leds[13] = CRGB(red, green, blue);
  }

  //access values in the matrices which store LED IDs and ranges that correspond to certain numerical valus
  //display the ones that correspond to the time at that moment
  //for example, if the minute is 26, it will go to the 26th element in the matrix, set the first value of that element as the start ID, and set the 2nd value as the range
  //for that example specifically, this part only illuminates "six" and "twenty" is lit up separately
  fill_solid(&leds[minConfigs[minutes][0]], minConfigs[minutes][1], CRGB(red, green, blue));
  fill_solid(&leds[hourConfigs[hours][0]], hourConfigs[hours][1], CRGB(red, green, blue));

  //transfers all the settings (including the reset all black later in the code) over to the LEDs to be phsyically shown
  FastLED.show();
}

//main code goes here and runs repeatedly
void loop() {
  //configuring all the time, which would typically go in setup() but here goes in loop() because the gmtOffset variable will update according to the token's location
  //also sets the offset for daylight savings time, as well as the server (we're using a typical online server that is used for many of these projects)
  configTime(gmtOffset, daylightOffset, ntpServer);

  //makes a group of 9 variables, that follow a structure predefined by tm
  //tm accomodates for the getLocalTime() function to populate it with the values for the current time when called
  struct tm timeInfo;

  //the ampersand allows getLocalTime to directly modify timeInfo's 9 variables by giving it the memory address for the timeInfo containers
  //if getting the time doesn't fail (which is what the conditional is meant to detect), getLocalTime populates timeInfo, which is kind of like 9 individual variables structured in the tm format
  if(!getLocalTime(&timeInfo)){
    Serial.println("Failed to obtain time");
    return;
  }

  //set the following variables to the current hour and time respectively, using functions from the time library and specific formats from the tm struct
  hours = timeInfo.tm_hour;
  minutes = timeInfo.tm_min;

  //reset: turns all LEDs off to set up for the new time if the time changes, otherwise the new time is just written over again
  //the reason this doesn't create a flashing effect is because the code for turning all the LEDs off is stored but not shown yet; it's shown after the code for the LEDs is also written
  fill_solid(&leds[0], NUM_LEDS, CRGB::Black);

  //sequence for if the time zone changes
  //setTime() is a function defined earlier, so you just have ot pass in parameters for a red color and have that fade in and out
  //reedSwitchLast stores the data for which reed switch was last set off, and reedSwitch stores the data for the new reed switch set off
  //iterate through a for loop that allows brightness to be increased and then another that allows it to be decreased
  if (reedSwitchLast != reedSwitch) {

    //extra code for another feature that wasn't finished
    /*
    for (auto& [pin, offset] : timezones) {
      if (digitalRead(pin) != 0) {
        colorNum += 1;
      }
    }
    for (int i = 0; i < 13; ++i) {
      if (colorOrderNum == 1) {
        id1 = 0;
        id2 = 1;
        id3 = 2;
      }
      if (colorOrderNum == 2) {
        id1 = 3;
        id2 = 4;
        id3 = 5;
      }
      if (colorOrderNum == 3) {
        id1 = 6;
        id2 = 7;
        id3 = 8;
      }
      fill_solid(&leds[rowVals[i]], 14, CRGB(countryColors[colorNum][id1], countryColors[colorNum][id2], countryColors[colorNum][id3]));
      for (int b = 1; b < 3; ++b) {
        startBrightness += b;
        FastLED.setBrightness(startBrightness);
        FastLED.show();
        delay(20);
      }
      if (colorOrderNum == 3) {
        colorOrderNum = 1;
      } else {
        colorOrderNum += 1;
      }
    }
    
    for (int b = startBrightness; b > 0; b -= 2) {
      FastLED.setBrightness(b);
      FastLED.show();
      delay(20);
    }
    */

    for (int i = 0; i < 2; ++i) {
      for (int i = 0; i < 100; i += 10) {
        FastLED.setBrightness(i);
        setTime(255, 0, 0);
        delay(10);
      }
      for (int i = 100; i > 0; i -= 10) {
        FastLED.setBrightness(i);
        setTime(255, 0, 0);
        delay(10);
      }
      fill_solid(&leds[0], NUM_LEDS, CRGB::Black);
    }
    //update reedSwitchLast so that the cycle doesn't go on forever
    reedSwitchLast = reedSwitch;
  }

  //A2 send a LOW signal to all of the reed switches, which is checked for on multiple other pins to see which reed siwtches are closed
  //iterates through a matrix that contains the pin IDs for each pin on the nano that is wired to a reed switch and checks if the switch is closed
  //then, if the switch is set off, it takes the 2nd value in that matrix element and sets the GMT offset to that value, which is updated at the beginning of the loop
  digitalWrite(A2, LOW);
  for (auto& [pin, offset] : timezones) {
    if (digitalRead(pin) == 0) {
      gmtOffset = offset;
      reedSwitch = pin;
    }
  }

  //set the default brightness for the display and show the time with default white color
  FastLED.setBrightness(150);
  setTime(255, 255, 255);

  //debug code which would allow you to light up the entire grid and see where a break is in the LED chain
  //fill_solid(&leds[0], NUM_LEDS, CRGB::White);

  delay(400);
}
```

## Configurations in platformio.ini

This project was programmed on VS Code with the Platform IO extension, which configures an environment on the IDE for programming microcontrollers such as the Arduino Nano ESP32. The following are specifications required for Platform IO, including libraries used:

```ini
; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter
;   Upload options: custom upload port, speed and extra flags
;   Library options: dependencies, extra library storages
;   Advanced options: extra scripting
;
; Please visit documentation for the other options and examples
; https://docs.platformio.org/page/projectconf.html

[env:arduino_nano_esp32]
platform = espressif32
board = arduino_nano_esp32
framework = arduino

lib_deps = 
    fastled/FastLED@^3.10.3
    paulstoffregen/Time@^1.6.1
```

# Bill of Materials
The essential costs for the project and cost of each item, as well as their function in the overall project:

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Arduino Nano ESP32 | the project's microcontroller or the "brain" of the project which stores and processes code; the central operator that controls other components | $18.30
| <a href="https://store-usa.arduino.cc/products/nano-esp32?srsltid=AfmBOoqTU1gbJDuiphj2Ymhx0KF3ZIAIQRkZAZkR9OxiSuW1m9UaDbtA"> Link </a> |
| WS2812B LED Strip (order 2) | LEDs to light up words corresponding to the time; can be cut and soldered as I did for individual LEDs for each letter or just soldered in rows | $8.99 | <a href="https://www.amazon.com/dp/B0BNN1JZS8?lv=shuf&s=hi&crid=1MIIJ7W08LDIH&keywords=Individually%2BAddressable%2BLED%2BStrips&sprefix=individually%2Baddress%2Ctools%2C1257&th=1&dib_tag=se&dib=eyJ2IjoiMSJ9.Tp2GRBfcB5XRN3_DGOSt1ZGfn5k9iGgHFvvbHYD-GgMURYFRwUykXxwbqmRMhgalucGUZm7h3lgfdazdAW3-NpAZqw-P1D9ATs66BOnBdlnYFTtra7sbA4fEuuO8vIpu1k1ToETi1NfMPYSow_NJU5j2rnrWYHboLaY8OZ7RUP0dna7RP9fAp8m_0RbiYHi9d6lDQkNH8evD-3r0_W0c6r8Cu-90ambDua5m1sk58idhdg6vESW81b1A0FuhqFN3siInXR1BqqutyePDX-o6rngmTkMQSa49ZFLG3MAilzw.RGP3Xz1e7bbUWb3l9zeoxxbqYC8rrlK7zzyWVsW5Xv0&qid=1782841976&sr=1-4&channelId=500&ref_=sr_1_4&plpRedirect=mhFallback"> Link </a> |
| Frosted Acrylic Sheet (12”x12”) | prevents harsh light from coming through the word cutouts; allows for a more consistent lighting of each word | $15.00 | <a href="https://www.amazon.com/dp/B07R9YRNZM?lv=shuf&keywords=frosted+clear+acrylic+sheet&dib_tag=se&dib=eyJ2IjoiMSJ9.ZXWzAMLgzG14HoXIRglW82kV4MpLMkj9CnuQ5ww8LcRqSaqE1yxzHaNUGgsojT1NH10UseoCBpvj1lP1um7u6O2sv2gObw-uJNCSiUyVScRjfVTDczL-L7R5BKYJXCtKKG-eeInJQvVgJ7IQLlBTVzSOzmGVGsUdAPSc37TGCP_XqvoTwANJmsMqxis-TaYYe_KMqfLPruXqdjnsp40mgTmK-aYaGoCeCpmlGxvQZWo.4ANrX46Xz-eQ_KilWISUtOqpXoioPctk_WkogflSTPs&qid=1782672692&sr=8-20&channelId=500&ref_=sr_1_20&plpRedirect=mhFallback"> Link </a> |
| Clear Acrylic Sheet (12"x12") Pack of 2 | composes the backing for the clock (surface for the LEDs) and the sides to make it stable/boxy | $18.30
| <a href="https://www.amazon.com/Clear-Acrylic-Sheet-Plexiglass-2-Pack/dp/B081R3VCLL/ref=sr_1_1_sspa?dib=eyJ2IjoiMSJ9.TrlIoOO_kkx2582e7270mRkNdeMMHa8pUPem-V-MX_veX401GG0fRin737QNuYv-jKEeyOMX5hN5nEDK3lKpXieoTOX6xN8Vmi4PWjNFfySOKwRTvwBW--Uvl_QWtSdybc5r-eUmpqrfMx2czdkz86ov_7c8NyrziS31C6O_r4XobS_j3OlRO5XZscw3hl6nz3d3AygVNYndtP05JQYHL9u6ddpCwXKY-jlJV6PNPOs.JQlgB40Nwf7LJtBr5d4CdZjp5yYld3pb9UHzitF8AOk&dib_tag=se&hvadid=695066734069&hvdev=c&hvexpln=67&hvlocphy=9032183&hvnetw=g&hvocijid=16295209345137170117--&hvqmt=e&hvrand=16295209345137170117&hvtargid=kwd-295763709733&hydadcr=24656_13611721&keywords=12x12%2Bacrylic%2Bsheet&mcid=c34901558a0e3fb5a51f48364bc353cd&qid=1785515643&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |
| Assorted Cardstock (multiple colors; 12"x12") | what I used to make the isolating grid to stop light from bleeding into other letters; I also used a laser cutter at home to make the front panel with the letter grid that I designed | $18.30
| <a href="https://www.amazon.com/Neenah-Collection-Specialty-Assortment-46408-02/dp/B003A2I4TO/ref=sr_1_7?channelId=4&clpRedir=Y&dib=eyJ2IjoiMSJ9.7R90mO8YkLbG001URG4Nk8yONgDd4kJzji550MQdw6GG0ErNUyoLmqZpcgnKsD0wpw8GnnifPsWLTaxkJVcL150VnJovDBJhguyujxQ_U3Z0Pgd4Qm7G0FDBvx3pBntRf32-QwAsUjFRuQg3bCxhzFxYlyrRfFbCxqVwAlLN4wgSvUaZWIzqDTUOEa0RvQNasq1GLzTG39g6mglCe4ueBSp07Eu6jH3kY0RxMuC1I14Lyi52qbh_3qRgic3E9WvA1J5bIC7KgRMIse9TmtqcptWHZMJ0RmFg2bLC-B0UO4Q.w8CRTM03-uMncwA69D9g1E65j6e2TVc5RrurODSGzwg&dib_tag=se&hvadid=693936209231&hvdev=c&hvexpln=67&hvlocphy=9032183&hvnetw=g&hvocijid=13896404654331395278--&hvqmt=e&hvrand=13896404654331395278&hvtargid=kwd-298652188787&hydadcr=4863_13229487&keywords=12x12%2Bcardstock&mcid=8993af7c102837c48df34d6c7ce94265&plpRedirect=mhFallback&qid=1785515854&sr=8-7&th=1"> Link </a> |
| Assorted Capacitors and Resistors (other smaller kits available) | used 2 capacitors to manage voltage drop and 1 resistor to keep the data wire for the LEDs safely operating | $25.99
| <a href="https://www.amazon.com/MOGAOPI-Electronic-Electrolytic-Capacitors-Transistor/dp/B09237GYCD/ref=sr_1_1_sspa?dib=eyJ2IjoiMSJ9.Wpac7S1V8txZVlLd6kpC7rWmIeKkONuelRYp53Lh5qgilaqV0Gh2gpH-Zlww6QPw1pgR461A-gZySPxPsJAn2UZrTW6xH8w9A8HTNHhGiCp0aRB0_jd3cVXYSex8nfaZxe4ZstdixTp-3nNHy2FIHquZO1oMxhO5sUQFA2LkLMxMs_dgdWf5IoMfA0miYx9zVPsZAbH2mZot2pZwYdcAR2vUsOhF0sEaLPg4AQrIhX6oIosawEoF9vJRAzUL1xnEPN0PitvoPv6mle9eZNwaNQ_ya5l5EV0GCYSMN011lNE.nD-3OWnnqSOGJyQ98nucrWuU6MwmXWwQIrbzGo34oSA&dib_tag=se&hvadid=792754737803&hvdev=c&hvexpln=67&hvlocphy=9032183&hvnetw=g&hvocijid=7073605126640458081--&hvqmt=e&hvrand=7073605126640458081&hvtargid=kwd-331922686271&hydadcr=543_1015366242&keywords=resistor+and+capacitor+kit&mcid=eafdfa97d07b3ce4becc9b8baca614b7&qid=1785516543&s=industrial&sr=1-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Perfboards | a basis for the smaller components involved in the project, such as the microcontroller, capacitors, and resistors | $9.99
| <a href="https://www.amazon.com/ELEGOO-Prototype-Soldering-Compatible-Arduino/dp/B072Z7Y19F/ref=sr_1_1_sspa?crid=M81G3XGD2M6Z&dib=eyJ2IjoiMSJ9.FPDVUs4HOgNXdui7sk7F5_Lb6eBhfnp0Lfap_6Wi-bsD1glkmTTecqq9g7BYok0PWlGb6XIKVPdCX_7FyXqQElwKc11lev2Tx9UOu_b1yovkB-ZjCfs4L0Cwn-_mSBvIobyITrgM6TVfTgE2_B2j0mDlmxJHJ8h264Kbe-vou9uFJffRgLdHn8LodEjEEWySwL1xkUd40DQkyLTPN01v700-6W3N38WFvh06NVq-nMI.LiMxyCcAxx-dolTm8CqMAt6aotHp8bdigClHL7KuW8I&dib_tag=se&keywords=individual+perfboard&qid=1785516776&sprefix=individualperfboard%2Caps%2C215&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Assorted Wire Spools | connects all electrical components and carries power or data | $15.99
| <a href="https://www.amazon.com/sspa/click?ie=UTF8&spc=MTo4Mzk2Mzk3NDQ1MTMzNjc3OjE3ODU1MTcwMDY6c3BfYXRmOjMwMDE5MDAxNjUxMTUwMjo6MDo6&url=%2FTUOFENG-Solid-Colors-Tinned-Copper%2Fdp%2FB07TX6BX47%2Fref%3Dsr_1_1_sspa%3Fdib%3DeyJ2IjoiMSJ9.QwMX3CTcLF7NPouCJnhBWWTVqMXBkRL5yKXS-uGMkMEXgVqo0GHkVgBrzL0tJ5NnArguOjZ4bI2dJEZcOrZDotfHpgw-2NWXrDBUsOvM-bFy3GjSlbtQWDGBwhDjveLd4ec9Ddl8R-_bMoLFKfHSorBeomFTDSZEY8neBhdez8DS_OaRKojovpvZ3zhiEsxT0hRTM46W3hROEFyFI_ybJ4NgggcS57DVBykFDQuk_hXfRjK6U9MYxDotepnVVYITzsqGPhokrnZnXqUeyn89Gp9LstCn3Bz854xrgR8cFQs.l8sfy6G2kM7FbD-MmLGLcmpRm1q1RhDX5GhMou1z7lQ%26dib_tag%3Dse%26hvadid%3D777805363327%26hvdev%3Dc%26hvexpln%3D67%26hvlocphy%3D9032183%26hvnetw%3Dg%26hvocijid%3D16185805269056756440--%26hvqmt%3De%26hvrand%3D16185805269056756440%26hvtargid%3Dkwd-21758096%26hydadcr%3D8426_13831415%26keywords%3Dwire%2Bspool%26mcid%3Da7224b9707bb34fba3d04cdf22f990d5%26qid%3D1785517006%26sr%3D8-1-spons%26sp_csd%3Dd2lkZ2V0TmFtZT1zcF9hdGY%26psc%3D1"> Link </a> |
| 9V DC Adapter | provides power that can be handled by the linear voltage regulator from a typical power outlet | $9.35
| <a href="https://www.amazon.com/sspa/click?ie=UTF8&spc=MTozMTE3MzA0OTYxNDA0MDI4OjE3ODU1MTcxODQ6c3BfYXRmOjMwMDIwMjg4OTE0MzkwMjo6MDo6&url=%2FAdapter-Guitar-Supply-PSA-120S-ME-50B%2Fdp%2FB087JJ1JHX%2Fref%3Dsr_1_1_sspa%3Fdib%3DeyJ2IjoiMSJ9.6_WALrpeCtl4wUAC3g0gvDgY1sGYWe3G5R8p-Yd567_eFNMenZSpDkOeyCpHtMGUjfDImdHAaa2CRwB6uOwn4GnGWVMl-zRWf8EW8FX5F271dSs4ShHYJSTOMVdRouUN5spi7KK_mMrlTx85Pm4s9VXnP7MMxXrZcRmT3FnSAMzZDQbIWgGisrRz-0kIK9dt3IaCAdetbMx1iwJMejoz-yly7zrgC3Q5tBkXX6HDfxU.xCQuE9mj7-0lSuWFqq0TDXw9raN0W5oMOJmvXyz3lmo%26dib_tag%3Dse%26hvadid%3D693349412817%26hvdev%3Dc%26hvexpln%3D67%26hvlocphy%3D9032183%26hvnetw%3Dg%26hvocijid%3D8489475260472379682--%26hvqmt%3Db%26hvrand%3D8489475260472379682%26hvtargid%3Dkwd-65527940%26hydadcr%3D19103_13454503%26keywords%3D9v%2Bdc%2Badapter%26mcid%3De7c67687e2863697a591e5fcced55e0c%26qid%3D1785517184%26sr%3D8-1-spons%26sp_csd%3Dd2lkZ2V0TmFtZT1zcF9hdGY%26psc%3D1"> Link </a> |
| Linear Voltage Regulator (9V to 5V) | steps down the voltage from the DC converter to 5V, which can both power the LED grid and the Arduino Nano ESP32 | $0.42
| <a href="https://www.digikey.com/en/products/detail/umw/L7809CV/24889965?gclsrc=aw.ds&gad_source=4&gad_campaignid=21136823955&gbraid=0AAAAADrbLlgax7XkhkBZAYZXBTjxmM_Ox&gclid=CjwKCAjwj7HTBhBiEiwA8s35OmWbbnDTIcJ-OTrKWcdzf3ZUveNLpL-t31LZuEuk38GTTEdtdjzcWhoCITkQAvD_BwE"> Link </a> |
| DC Barrel Jack Adapter | allows for wiring the DC adapter to the perfboard | $1.90 | <a href="https://www.dfrobot.com/product-508.html?gad_source=1&gad_campaignid=23441887437&gbraid=0AAAAADucPlAcY6rVjhUWVJH9oTBNrIIQq&gclid=CjwKCAjwj7HTBhBiEiwA8s35OliOnHRyuTPeaf5M04IkpQPCyezKpIN4ZCPyI52AzZwlPUcb7T2r2hoCDJQQAvD_BwE"> Link </a> |
| 8" World Globe | the main stage of the globe portion; self explanatory | $19.00 | <a href="https://www.amazon.com/Exerz-Educational-World-Swivel-Rotating/dp/B014TL44FM/ref=sr_1_5?crid=3OD9W9YSLBZM4&dib=eyJ2IjoiMSJ9.ErRL3Ht18bMKpWTAOOwUcXb9CAXMhCYV6zbHbi-MDMo2exSBzbXvAJa2ih7XH861ItlAtEhBuOb82UDpx3F0dWyaG3oTFrKhPVnHqLbJPGdzOmXnUsjxfTrXwl_YIP3pLk_bm-RW0lYq_YUlWMNCYxfvlIzBot9DLuQJF08JAjbWN1-fIQ-MPGH-J06VWMqJ28ANLD35da6fbjgb3ad-ElXOx4yhy4O9xQFMEeAMp2IBOYtiKGo_SqA33RkPM15D20S3uk6_R9_kkz1xpKPIAqmq5ohHxSKDSBnC9oHfBL8.OKfqnW3o0ba9UT0oePj-tOXR2MVBoavAb_vEDpdSO5k&dib_tag=se&keywords=globe&qid=1782675875&refinements=p_36%3A1100-2700&rnid=386479011&s=office-products&sprefix=globe%2Coffice-products%2C222&sr=1-5"> Link </a> |
| Glass Reed Switches (10 pcs) | detects when the magnet token is nearby and closes a circuit, which is read by the microcontroller | $9.99 | <a href="https://www.amazon.com/Switch-Magnetic-Normally-Closed-Conversion/dp/B0CJFJ2316/ref=sr_1_1_sspa?dib=eyJ2IjoiMSJ9.ngVg8JW-tehsy1O_AeDyiX0X_6vfdpNKDPVfByZ1GjD1uCEBZHwqjlCRmaF6jn5sexex5NGPtlc7d8iHF3-O3NSwsK8WMjudNRjU0WTjKCURoHj_Becheajn7yPOd7JxhHFdPGR9UNakV74TkDuxOdn4sfVhYVTJMSSce7dVS9VGVQ6D-WJp91c3ktFz4GSToLPQrpc3zr7BKx40FmduwJ4JE5czr2wwcY27Gzj-cSI.6fAjNP-sRsHwXl41x3Vxs4aII-3uhRmDT-MPwZ-PrXQ&dib_tag=se&hvadid=693978383251&hvdev=c&hvexpln=67&hvlocphy=9032183&hvnetw=g&hvocijid=3393158275765664050--&hvqmt=e&hvrand=3393158275765664050&hvtargid=kwd-303061515838&hydadcr=27084_14699464&keywords=reed%2Bswitch%2Bglass&mcid=8f905b1948a931808f7bc60d1bd5a9a7&qid=1785519034&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&th=1"> Link </a> |
| Ceramic Disc Magnets (20 pcs) | used for the airplane token and also for inside the globe to keep the token suspended | $4.99 | <a href="https://www.amazon.com/Magnets-Adhesive-Backing-Refrigerator-Projects/dp/B0BKQ4J2DB/ref=sr_1_11?dib=eyJ2IjoiMSJ9.7Lvwznq0O0Ud0JGycxrlGdO1YvG28zJ5uZkupRt-pDU1rV8lpsvQ51SeeAEQxNEREPVCLjjlrS5CFWbPv7cJx6DWA7cnF_JmPt_oNTyeHvmz0gs8IXuTgV5BxDyZlexZCLgFbwumYvMIqsaPvylBy0Kz9iz2ox5nt5HPTrS5XvVzKBx7jkbkUk-YtoBOy4P9x4S9SeKM0-BIpLj4-WJHKeyV5r-LUmDdfEew7N8f8F4.1ExiDg8AXCZQ5GTTdV43SvYW5djObgsLDBAG1X4Vli4&dib_tag=se&hvadid=792737725364&hvdev=c&hvexpln=67&hvlocphy=9032183&hvnetw=g&hvocijid=16384528335500780283--&hvqmt=e&hvrand=16384528335500780283&hvtargid=kwd-296253639522&hydadcr=7518_13871070&keywords=ceramic%2Bdisc%2Bmagnets&mcid=32981390dc83362f953f3b898250f88a&qid=1785519234&sr=8-11&th=1"> </a> |

# Other Resources/Examples 
 - 
 - 
 - 

# Other Bluestamp Portfolios
Example portfolios that can be used for reference and insight into the program:
- [ASL Robotic Arm](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Ball Tracking Robot](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Gesture-Controlled Robot and Custom Computer Game](https://arneshkumar.github.io/arneshbluestamp/)

To Angeline, Kin, Ishaan, and Justin --> thank you!
