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

# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | What the item is used for | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components.
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

To watch the BSE tutorial on how to create a portfolio, click here.
