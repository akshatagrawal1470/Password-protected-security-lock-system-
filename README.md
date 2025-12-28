Hello everyone,
This is our **Password Protected Security Lock System**, developed as part of the **MPMC course**, by **Akshat Agrawal** and my teammate **Ritesh Sarawgi**.

In this project, we have designed a secure digital lock system using **Arduino Nano**, **4×4 keypad**, **I2C 16×2 LCD**, **buzzer**, and an **HC-05 Bluetooth module**. The system allows users to enter a password through the keypad, which is masked on the LCD for security. The password is stored permanently in **EEPROM**, ensuring data retention even after power loss.

The lock provides audio feedback using a buzzer, allows a limited number of attempts, and activates a lockout mode after multiple wrong entries. A Bluetooth-enabled admin mode is implemented using the HC-05 module, allowing the system to be unlocked remotely. Users can also change the password securely after verifying the current password.

Although the project looks simple at first, implementing features like **EEPROM handling, password masking, keypad interfacing, buzzer feedback, and Bluetooth control** helped us strengthen our embedded systems concepts. The **Embedded Systems Workshop** played an important role in applying these concepts practically.

This project demonstrates a real-world application of a **microcontroller-based security system** and integrates both hardware and software concepts learned during the course.

🔐 How the System Works (Step-by-Step)

* The system starts and loads the saved password from EEPROM.
* User enters digits using the keypad.
* Entered digits are stored in `inputBuffer`.
* The LCD shows `*` instead of actual digits for security.
* Pressing `C` removes the last entered digit (backspace).
* Pressing `#` submits the entered password.
* If the password matches:

  * Access is granted.
  * Buzzer beeps for confirmation.
* If the password is wrong:

  * Buzzer gives error sound.
  * Attempts are reduced.
* After multiple wrong attempts:

  * System enters lockout mode.
  * Keypad is disabled.
* Admin can unlock the system using Bluetooth (HC-05).
* User can press `*` to enter password change mode.
* Old password is verified before allowing a new password.
* New password is saved in EEPROM permanently.


🧩 Code Structure and Functions Used

📌 Important Functions in the Code

* `setup()`
  Initializes LCD, keypad, buzzer, Bluetooth, and loads password from EEPROM.

* `loop()`
  Continuously checks keypad input, Bluetooth commands, and system state.

* `savePasswordToEEPROM()`
  Stores the password length and characters in EEPROM.

* `loadPasswordFromEEPROM()`
  Reads the stored password from EEPROM and returns default password if EEPROM is empty.

* `passwordChangeFlow()`
  Handles secure password change after verifying the old password.

* `unlockedSequence()`
  Displays access granted message and plays success buzzer sound.

* `wrongAttemptFeedback()`
  Shows wrong password message and plays error sound.

* `startLockout()`
  Activates lockout mode after maximum wrong attempts.

* `showPrompt()`
  Displays messages on the LCD.

* `showInputMasked()`
  Displays masked password (`*`) on LCD.

* `beep()`
  Generates short buzzer sound on keypress.

🔁 Explanation of `setup()` Function

* Initializes buzzer pin and LCD display.
* Starts serial communication for Bluetooth.
* Loads password from EEPROM into `savedPassword`.
* Checks if EEPROM is uninitialized and stores default password.
* Resets attempts and prepares the system for user input.
* Displays initial message on LCD.


🔄 Explanation of `loop()` Function

* Continuously checks if the system is in lockout mode.
* Listens for Bluetooth `UNLOCK` command during lockout.
* Reads keypad input using `keypad.getKey()`.
* Handles:

  * Digit entry
  * Backspace (`C`)
  * Submit (`#`)
  * Password change (`*`)
* Compares entered password with stored password.
* Grants access or triggers lockout accordingly.
* Updates LCD and buzzer feedback in real time.

THANK YOU
