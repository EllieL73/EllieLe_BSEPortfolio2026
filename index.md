# Time-Zone-Alterable Word Clock
A wall-mountable clock that displays words which correspond with the current time. These words are shown through a laser-cut sheet and LED lighting behind it, with select words being illuminated at a time. It is electronically to a globe, on which one may move a token around the surface to set a time zone for the clock. The token attaches magnetically, with a magnetic follower inside of the globe. Electronics are situated inside of the globe and calculate the position of the token, conveying that data to set the clock's time zone.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Ellie L | Santa Teresa High School | TBD | Incoming Junior

![Headstone Image](logo.svg)
  
# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/-jb-2cFIigA?si=iRqVPUFcU7uKYvGv" title="Final Milestone Youtube Video" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE

# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/N8rfFMciLm0?si=YuZB1L0fa5mSrYMO" title="Milestone 2 Youtube Video" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 

# First Milestone

<iframe width="1160" height="610" src="https://www.youtube.com/embed/PoXtgeYFXZ4?si=KZ8p7ZyIlwunwfTO" title="Milestone 1 Youtube Video" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

My first milestone consists of the entire word clock portion of the project, which took me well over half of the progrma to build. Using hand-sawed acrylic for the backing and sides, as well as a clouded acrylic sheet for the front panel, the majority of the structure and frame are assembled with hot glue and superglue. Inside of the square frame, 201 individual LEDs are daisy chained in series, each with a power and ground wire as well as a data wire that runs from one to the other. The LEDs are cut from a typical individually addressable LED strip, and I soldered each one according to the spacing of letters that I designed on my grid. There is a cardstock grid that isolates each LED from the other, allowing for them to light up individually without the light bleeding over.

In order to make the letter grid, I planned out the most space-efficient option for displaying the time, which included optimizations such as having one constant display of "twenty" for all corresponding minute values, and having numbers such as "fourteen" also double as "four" and "twenty four." Having tihs structure in mind, I wrote out a grid of letters and transferred them over to Canva for more precise measurement and font. I downloaded this design and uploaded it to a Cricut workspace in order to laser cut them out of a 12 inch by 12 inch black cardstock sheet.

As each LED is wired in series, a burnt out or malfunctioned LED along the chain can and will cause all succeeding LEDs to stop responding or dim. To debug the chain as I soldered, I used a multimeter to check for the proper voltage and used a mode on the multimeter to trigger a beeping sound confirming a connection through the data wire. Through this process, I was able to replace LEDs accordingly and light up the entire grid. Also due to the fact that they are wired in series,  power is injected in two additional locations along the chain to prevent a major voltage drop that would cause inconsistency and dimming. In future iterations, I may experiment with parallel wiring for power, even if data must be wired in series.

The display can be powered by plugging a cord into a typical wall outlet, as the specific cord steps the voltage down to 9V, and a linear voltage regulator on the breadboard steps the voltage down further from 9V to 5V, which can power both the Arduino Nano ESP32 and the LED grid. To program the display, I used the FastLED and Time libraries, which have the tools or functions necessary to sync the time over WiFi and allow those to correspond with pin IDs and ranges. Said IDs and ranges are stored in matrices within the code for easy access and to make the code cleaner to read.

# Schematics 
Here's where you'll put images of your schematics. [Tinkercad](https://www.tinkercad.com/blog/official-guide-to-tinkercad-circuits) and [Fritzing](https://fritzing.org/learning/) are both great resoruces to create professional schematic diagrams, though BSE recommends Tinkercad becuase it can be done easily and for free in the browser. 

# Code
Here's where you'll put your code. The syntax below places it into a block of code. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize it to your project needs. 

```c++
void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);
  Serial.println("Hello World!");
}

void loop() {
  // put your main code here, to run repeatedly:

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
