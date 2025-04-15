# RP2040_MacroKeyPad_DevBoard

<img src="https://github.com/TomatoCube18/RP2040_MacroKeyPad_DevBoard/blob/main/images/Board_TopView.jpg"  width="600" height="auto" />

The **RP2040 Macro-KeyPad DevBoard** is an open-source hardware platform featuring a Raspberry Pi RP2040 MCU Packed with various peripherals i.e., 6 tactile Cherry MX keys with vibrant per-key NeoPixel LEDs, a built-in speaker, power/user LEDs, and a I²C temperature sensor. There's also a Qwiic connector for further I²C expansion. This board is perfect as a desktop Gadget or for diving into embedded dev in Circuit/MicroPython, or Arduino.


<br>

<br>

**Board Features:**

- Main Processor: Raspberry Pi RP2040
- On-Board peripherals: 
  - 1x 64MBit EEPROM Flash (8 MB)
  - 6x Cherry Key Switches (GPIO 0, 1, 2, 3, 4, 5)
  - 6x Level-Shifted NeoPixel LEDs (GPIO 19)
  - 1x Power LED
  - 1x User LED (GPIO13)
  - 1x Reset Button
  - 1x I2C QWiic Connector (SDA-GPIO20, SCL-GPIO21)
  - 1x On-Board Speaker with Amplifier with Low-Power Shutdown (Audio-GPIO16, Shutdown-GPIO14)

<br>

<img src="https://github.com/TomatoCube18/RP2040_MacroKeyPad_DevBoard/blob/main/images/Board_BottomView.jpg"  width="500" height="auto" /> <img src="https://github.com/TomatoCube18/RP2040_MacroKeyPad_DevBoard/blob/main/images/MacroKeyPad02.jpg"  width="500" height="auto" />

<br>
<br>

## CircuitPython Quick Start Guide with Thonny IDE:

### 🧰 What You’ll Need to get started:
- RP2040 Macro-KeyPad DevBoard
- micro-USB cable
- Thonny IDE (Download: https://thonny.org)

- (Optional for Manual Installation- Step1a) CircuitPython firmware for RP2040 (https://circuitpython.org/board/raspberry_pi_pico/)

<br>

### 🚀 Step 1: Install CircuitPython on RP2040 

#### [Option A] Install CircuitPython on RP2040 Manually

1. **Enter Bootloader Mode**
   - Hold the **BOOTSEL** button (Macro-Key SW_F), on your RP2040 Macro-KeyPad DevBoard.
   - Plug in the USB cable to your computer.
   - Release the BOOTSEL button.
   - A new drive named `RPI-RP2` will appear.

2. **Copy CircuitPython UF2**
   - Download the latest `.uf2` file from [circuitpython.org](https://circuitpython.org/board/raspberry_pi_pico/).
   - Drag and drop it into the `RPI-RP2` drive.
   - ✅ The board will reboot and mount as `CIRCUITPY`.



#### [Option B] Install CircuitPython via Thonny

1. Plug your RP2040 Macro-KeyPad DevBoard into your computer via USB.
   - (Hold the **BOOTSEL** button (Macro-Key SW_F), while plugging in only if you're flashing it for the first time.)
2. Open **Thonny IDE**.
3. Go to **Tools > Options > Interpreter**.
4. Set:
   - **Interpreter**: `CircuitPython (generic)`
5. If CircuitPython isn’t already installed, Thonny will prompt you to install it.
6. Click **Install or update CircuitPython(UF2)**, then:
   - Choose your board, Select "variant: Raspberry Pi Pico/Pico-H"
   - Choose the CircuitPython version, "version 9.2.7" as of the writing of this guide
   - Click **Install**

✅ Your board will reboot and appear as a `CIRCUITPY` USB drive.

<br>

### 💡 Step 2: Blink the User LED(Hello-World, CircuitPython!)
Let’s start with something simple, blinking the user LED on GPIO 13.

#### 🔧 Blink Code

```python
import board
import digitalio
import time

led = digitalio.DigitalInOut(board.GP13)
led.direction = digitalio.Direction.OUTPUT

while True:
    led.value = True   # LED ON
    time.sleep(0.5)
    led.value = False  # LED OFF
    time.sleep(0.5)

```
Hit `F5` or the Green Run button on the tool panel to write the script. If Everything works correctly, the user LED connected to GPIO 13 should start blinking every half-second!

<br>

### 💡 Step 3: Blink NeoPixels

#### 📦 Install NeoPixel Library

1. Download the [CircuitPython Library Bundle](https://circuitpython.org/libraries), we will be using the zip file under "Bundle for Version 9.x".
2. From the bundle *`lib/`* folder, copy:
   - `neopixel.mpy` & `adafruit_pixelbuf.mpy` → to `CIRCUITPY/lib/`  
   *(Create a `lib/` folder if it doesn’t exist)*

#### 🔧 Sample Code

```python
import board
import neopixel
import time

pixel_pin = board.GP19
num_pixels = 6
pixels = neopixel.NeoPixel(pixel_pin, num_pixels, brightness=0.5, auto_write=True)

while True:
    pixels.fill((0, 0, 255))  # Blue
    time.sleep(0.5)
    pixels.fill((0, 0, 0))    # Off
    time.sleep(0.5)

```

<br>

### ⌨️ Step 4: Color Organ

Combining all that we know till this step, we are going to try out Cherry Key Button Input and the Sound output.

#### 🔧 Sample Code

```python
import time
import array
import math
import board
import digitalio
import neopixel
from audiocore import RawSample
from audiopwmio import PWMAudioOut as AudioOut

pixel_pin = board.GP19
num_pixels = 6
pixels = neopixel.NeoPixel(pixel_pin, num_pixels, brightness=0.5, auto_write=False)
pixels.fill((0, 0, 0))
pixels.show()

button1 = digitalio.DigitalInOut(board.GP1)
button1.switch_to_input(pull=digitalio.Pull.UP)
button2 = digitalio.DigitalInOut(board.GP2)
button2.switch_to_input(pull=digitalio.Pull.UP)
button3 = digitalio.DigitalInOut(board.GP3)
button3.switch_to_input(pull=digitalio.Pull.UP)
button4 = digitalio.DigitalInOut(board.GP4)
button4.switch_to_input(pull=digitalio.Pull.UP)
button5 = digitalio.DigitalInOut(board.GP5)
button5.switch_to_input(pull=digitalio.Pull.UP)
button6 = digitalio.DigitalInOut(board.GP0)
button6.switch_to_input(pull=digitalio.Pull.UP)


sine_wave_sample = []

def createToneWave(frequency=440): # Set freq to the Hz of the tone you want to generate.
    global sine_wave_sample
    tone_volume = 0.8  # Increase this to increase the volume of the tone.
    length = 8000 // frequency
    sine_wave = array.array("H", [0] * length)
    for i in range(length):
        sine_wave[i] = int((1 + math.sin(math.pi * 2 * i / length)) * tone_volume * (2 ** 15 - 1))
    sine_wave_sample = RawSample(sine_wave)

audio_en = digitalio.DigitalInOut(board.GP14)
audio_en.direction = digitalio.Direction.OUTPUT
audio_en.value = True	# Asserted/Active Low OnBoard Amplifier

audio = AudioOut(board.GP16)


while True:
    if not button1.value:
        pixels.fill((128, 0, 0))	# Red
        pixels.show()
        audio_en.value = False
        createToneWave(262)
        audio.play(sine_wave_sample, loop=True)
        time.sleep(1)
        audio.stop()
        audio_en.value = True
    elif not button2.value:
        pixels.fill((128, 128, 0))	# Yellow
        pixels.show()
        audio_en.value = False
        createToneWave(294)
        audio.play(sine_wave_sample, loop=True)
        time.sleep(1)
        audio.stop()
        audio_en.value = True
    elif not button3.value:
        pixels.fill((0, 128, 0))	# Green
        pixels.show()
        audio_en.value = False
        createToneWave(330)
        audio.play(sine_wave_sample, loop=True)
        time.sleep(1)
        audio.stop()
        audio_en.value = True
    elif not button4.value:
        pixels.fill((0, 0, 128))	# Blue
        pixels.show()
        audio_en.value = False
        createToneWave(349)
        audio.play(sine_wave_sample, loop=True)
        time.sleep(1)
        audio.stop()
        audio_en.value = True
    elif not button5.value:
        pixels.fill((128, 0, 128))	# Magenta
        pixels.show()
        audio_en.value = False
        createToneWave(392)
        audio.play(sine_wave_sample, loop=True)
        time.sleep(1)
        audio.stop()
        audio_en.value = True
    elif not button6.value:
        pixels.fill((128, 128, 128))	# White
        pixels.show()
        audio_en.value = False
        createToneWave(440)
        audio.play(sine_wave_sample, loop=True)
        time.sleep(1)
        audio.stop()
        audio_en.value = True
    else:
        pass

```
<br>

### ⌨️ Step 5: Use as a USB HID Macro Pad

#### 📦 Install HID Library

1. From the same CircuitPython bundle (Step 3), copy:
    - `adafruit_hid/ folder` → to `CIRCUITPY/lib/`

🔧 Sample Code
```python
import time
import board
import digitalio
import usb_hid

from adafruit_hid.keyboard import Keyboard
from adafruit_hid.keycode import Keycode

kbd = Keyboard(usb_hid.devices)

button1 = digitalio.DigitalInOut(board.GP1)
button1.switch_to_input(pull=digitalio.Pull.UP)


while True:
    if not button1.value:
        kbd.send(Keycode.ENTER)
        time.sleep(0.2)

```
<br>

You are now ready to define more keys and assign different keystrokes for a full macro pad setup!

<img src="https://github.com/TomatoCube18/RP2040_MacroKeyPad_DevBoard/blob/main/images/MacroKeyPad01.jpg"  width="600" height="auto" /> 
