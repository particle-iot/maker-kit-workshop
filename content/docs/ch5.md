# Chapter 5: Controlling your devices from the cloud

| **Project Goal**            | Process sensor data in the cloud, and publish events to control an RGB LED on your Photon.                 |
| --------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **What you’ll learn**       | How to control an RGB LED from a Photon; Subscribing to Device Cloud events; Publishing events from Azure. |
| **Tools you’ll need**       | build.particle.io, A Particle Photon and Maker Kit; Microsoft Azure Account                                |
| **Time needed to complete** | 30 minutes                                                                                                 |

In this session, we're going to tie everything together by adding an RGB LED to our project that can be controlled from the cloud. Our Azure instance will monitor the incoming temperature readings, and with the help of an Azure Function, we can fire events that will tell our RGB LED which colors to use based on the temp we pass along.

If you get stuck at any point, [click here](https://go.particle.io/shared_apps/5b85708299b6c19f4f00008e) for the completed source.

## Wire up the status RGB LED to your Photon

![](./images/05/rgb-circuit.jpg)

To build this circuit, you'll need the following items:

- Photon in breadboard (this is how it comes in the Maker Kit)
- 1x Clear cap RGB LED. The RGB LED has four leads: the left lead is for the blue light, second for green and the right lead for the red light. The longest lead is the common anode, which we'll connect to our power source.

![](./images/05/rgbled.jpg)

- 2x red, green, and blue jumper wires.
- 3x 330 Ohm Resistors. The maker kit includes four different types of resistors (220, 1k, 4.7k and 10k ohm). The one you need is blue, with red, red, black, black and brown bands, as depicted below.

![](./images/05/resistors.jpg)

1. The RGB LED has four leads. One common anode lead, and one each for the red, green and blue diodes inside the LED. Start by plugging the four legs into the first four rows of column J on your breadboard. Make sure the LED is oriented where the longest leg, the common anode, is the second from the right before you plug it in.

![](./images/05/ledpluggedin.jpg)

2. For each LED led, we'll need to wire a resistor in series to keep the LED safe. Grab the first resistor and plug one leg into the first row of column G, then plug the other led into the first row of column D.

![](./images/05/redresistor.jpg)

3. The lead at A1 is the red LED, so take a red jumper wire and place one end into the first row of column A. Place the other end into the `D2` pin on the Photon.

![](./images/05/redwire.jpg)

4. The second lead is the common anode lead, which we'll plug into our power source. Take the red jumper wire and plug it into row two of column H. Plug the other end into the `3V3` pin of the Photon.

![](./images/05/blackwire.jpg)

5. Take the second resistor and plug one leg into the third row of column G, then plug the other led into the third row of column D.

![](./images/05/greenresistor.jpg)

6. The third lead is the green LED, so take a green jumper wire and place one end into the third row of column A. Plug the other end into the `D1` pin on the Photon.

![](./images/05/greenwire.jpg)

7. Take the third resistor and plug one leg into the fourth row of column G, then plug the other led into the fourth row of column D.

![](./images/05/blueresistor.jpg)

8. Finally, The fourth lead is the blue LED, so take a blue jumper wire and place one end into the fourth row of column A. Plug the other end into the `D1` pin on the Photon.

![](./images/05/bluewire.jpg)

With our RGB LED circuit all wired up, let's write the firmware code for lighting it up when we receive an event from the cloud.

## Add an event subscription to capture events from Azure

1. First, let's create some `#define` statements to manage the pins of our RGB LED. Add these three lines after the `#define` we created in part two of this workshop.

```cpp
#define RED_PIN D2
#define GREEN_PIN D1
#define BLUE_PIN D0
```

2. Now let's set the `pinMode` of each pin on the RGB LED to be an output by adding the following lies to the beginning of our `setup` function.

```cpp
pinMode(RED_PIN, OUTPUT);
pinMode(GREEN_PIN, OUTPUT);
pinMode(BLUE_PIN, OUTPUT);
```

3. Next, let's add the `subscribe` primitive. `subscribe` tells the device cloud to watch for events that match a name and, like a `Particle.function` defines a handler to call when that event is received. Add the following line to `setup`.

```cpp
Particle.subscribe("setLED", setLEDColor, MY_DEVICES);
```

4. At the end of `setup` we'll also initialize the RGB LED to be totally off. Since the RGB LED in the maker kit is a common anode LED, all three leads receive power by default. To turn them off, we pass in the max analog value or 255. Add the following to the end of the `setup` function.

```cpp
analogWrite(RED_PIN, 255);
analogWrite(BLUE_PIN, 255);
analogWrite(GREEN_PIN, 255);
```

::: tip
`analogWrite` is a standard Arduino-style function for setting analog values on a GPIO pin. Unlike `digitalWrite`, which sets a pin either `HIGH` (digital 0 or 0 volts) or `LOW` (digital 1 or 3.3 volts on the Photon), `analogWrite` takes an integer value between 0 and 255 and converts that to a voltage in the operating range of the device (0 to 3.3 volts in the case of the Photon). This allows us to do some pretty cool stuff like turning a servo or motor, or controlling the brightness of an LED!
:::

5. Finally, let's add the `setLEDColor` function for our `subscribe` handler. Like `Particle.function` a subscribe handler needs to have a precise signature. It returns void and takes two char arrays for the event name and any data provided by the event. Copy the following into your program.

```cpp{3}
void setLEDColor(const char *event, const char *data) {
  int red, green, blue;
  int result = sscanf(data, "%02x%02x%02x", &red, &green, &blue);

  if (result == 3) {
    analogWrite(RED_PIN, 255 - red);
    analogWrite(BLUE_PIN, 255 - blue);
    analogWrite(GREEN_PIN, 255 - green);
  }
}
```

The key piece here is the `sscanf` function in the third line. In short, [this function](http://www.cplusplus.com/reference/cstdio/sscanf/) allows us to provide a C-style string (a char array) and a format string to use to extract multiple values from the input string into the variables provided by the remaining parameters (red, green, and blue, in this case). The format string (`%02x%02x%02x`) tells the function to read each pair of input values (`%02`) as a hexadecimal value (`x`) and place the result in the red, green, and blue variables, respectively.

Once we have those values extracted, we'll perform an `analogWrite` on each pin to set each color. Again, because this is a common anode RGB LED, we subtract the hex value from 255 in order to get the correct value for the LED.

6. Flash the firmware to your device.

7. Navigate to your device page in the Particle Console. Under Event Logs, there's a `Publish Event` button. Click on it to open the event publishing UI.

![](./images/05/publishUI.png)

8. In the event name textbox, enter "setLED" and a six character hex string for a color, like `FF0000`, in the Event data textbox.

![](./images/05/setLED.png)

9. Click the "Publish" button. Your RGB LED should light up with the color you provided.

![](./images/05/redLED.gif)

10. Try changing the color string to other hex values like `00FF00`, `0000FF`, `808080`

::: tip
To turn the light back off, you can publish an event with `000000` in the Event data field.
:::

Now that we have everything wired up on our device, lets publish events from the cloud!

## Processing sensor data in Azure and publishing events

- [Process the incoming event and fire an event back to the device]

## Bringing it all together!

- [Test it all out; LED should be blue(?) at first; Dunk the sensor in a glass of hot or cold water and watch it change]
