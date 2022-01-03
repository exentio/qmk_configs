# Hakase
⚠️ WORK IN PROGRESS ⚠️  

Hakase is an 8-key macro pad with a display and a rotary encoder, designed to also work as a remote for your smart home.  
The idea is to output serial commands towards a connected module like an ESP32 or a CC2652.  

## Current status

### Done
+ Base keyboard layer
+ Layer with serial output

### To Do
+ Display
+ Encoder
+ ESPhome code
+ CC2652 code
+ Get the hardware out of prototype stage and make a PCB

## Serial output
The baud rate can be configured in `config.h`, the default is `#define SERIAL_BAUD 9600`.  
As an example, the format of the data for a single tap of the first key is:

| **Byte**   | 1st | 2nd | 3rd | 4th |
|------------|-----|-----|-----|-----|
| **Output** | 0   | 0   | T   | \n  |

The serial layer outputs (with the default 8-key UART layer) 4 bytes when the key is tapped, held, double-tapped, or held after a tap, and after their release. It handles invalid tap dances (triple tap, triple hold, etc) with an invalid code, and in that case it doesn't output a release code.  
The first 2 bytes are the number of the key (by default, 00 to 07). The fourth is always a newline. The third byte contains the type of press:

| **Code** | **Description**                      |
|----------|--------------------------------------|
| T        | Tap                                  |
| H        | Hold                                 |
| D        | Double-tap                           |
| B        | Both (double tap ending with a hold) |
| R        | Release                              |
| X        | Invalid                              |

## Adding more serial output keys
1. Add new codes in `uart_tap_dance.h` inside the `dance_kcodes` enum.  
2. In `uart_tap_dance.c` add elements in the `tap_dance_actions` array, one for each of the new codes. They contain the `ACTION_TAP_DANCE_FN_ADVANCED` macro that accepts three functions as parameters (first tap of the key, end of tap dance, and release).  
    + Those functions can't have parameters: basically, it means you'll have to repeat a lot of code with differently named functions. Solving this would require changes to the QMK code, so **this is out of my and your control** (unless you feel like making a pull request to QMK). With that said, since the use case allowed it, I tried to minimize the issue, check the functions I wrote.  
3. Add the keys to the keymap using `TD(keycode)`.  

## Adding more press types
1. Add new codes in `uart_tap_dance.h` inside the `td_state_t` enum.
2. In `uart_tap_dance.c` adjust the switch-cases in both the `curr_dance()`, `dance_reg_switching()`, and `uart_data_code()` functions.