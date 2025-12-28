                                                                                  //-- Source Code --//
/* 
  Password-protected lock with:
   - 4x4 keypad (custom layout you provided)
   - I2C 16x2 LCD
   - Buzzer on D11 (beeps on every keypress)
   - Password stored in EEPROM (persistent)
   - Controls:
       - Digits: 0-9 (entered)
       - C : Backspace
       - # : Submit / Enter
       - * : Password change mode (press * to start change flow)
*/

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Keypad.h>
#include <EEPROM.h>

// ---- CONFIG ----
LiquidCrystal_I2C lcd(0x27, 16, 2); // change to 0x3F if needed
const int buzzerPin = 11;           // buzzer connected to D11
const byte MAX_PASS_LEN = 6;        // maximum password length supported
const byte PASS_ADDR = 0;           // EEPROM start address for password (stores length then bytes)
const unsigned int LOCKOUT_MS = 30000; // Lockout period (ms) after 3 wrong attempts
const byte MAX_ATTEMPTS = 3;        // allowed wrong attempts before lockout

// Keypad layout (your mapping)
const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'D', 'C', 'B', 'A'},
  {'#', '9', '6', '3'},
  {'0', '8', '5', '2'},
  {'*', '7', '4', '1'}
};

// Wiring: rows -> {9,8,7,6}  cols -> {5,4,3,2}
byte rowPins[ROWS] = {9, 8, 7, 6};
byte colPins[COLS] = {5, 4, 3, 2};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

// ---- STATE VARIABLES ----
String inputBuffer = "";         // current typed digits (shown masked)
byte attemptsLeft = MAX_ATTEMPTS;
bool lockedOut = false;
unsigned long lockoutUntil = 0;

// password in RAM (loaded from EEPROM)
String savedPassword = "";       // store actual password string in RAM

// helpers
void beep(unsigned int freq = 1000, unsigned int dur = 80) {
  // single short beep for keypress
  tone(buzzerPin, freq, dur);
  delay(10); // tiny gap, tone() is non-blocking but this helps spacing
}

// EEPROM helpers: store and load password
void savePasswordToEEPROM(const String &p) {
  // Format: [len][byte0][byte1]...
  byte len = p.length();
  if (len > MAX_PASS_LEN) return;
  EEPROM.update(PASS_ADDR, len);
  for (byte i = 0; i < len; ++i) {
    EEPROM.update(PASS_ADDR + 1 + i, (uint8_t)p[i]);
  }
}

String loadPasswordFromEEPROM() {
  byte len = EEPROM.read(PASS_ADDR);
  if (len == 0xFF || len == 0 || len > MAX_PASS_LEN) {
    // invalid or uninitialized -> fallback default
    return String("1234");
  }
char p[len + 1];
for (byte i = 0; i < len; i++) {
  p[i] = EEPROM.read(PASS_ADDR + 1 + i);
}
p[len] = '\0';

  return p;
}

// UI helpers
void showPrompt(const char* line1, const char* line2 = "") {
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print(line1);
  lcd.setCursor(0,1);
  lcd.print(line2);
}

void showInputMasked() {
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Enter Pass:");
  lcd.setCursor(0,1);
  // mask inputBuffer with '*'
  for (unsigned int i = 0; i < inputBuffer.length(); ++i) lcd.print('*');
}

// Unlock feedback
void unlockedSequence() {
  showPrompt("Access Granted", "");
  // 3 short beeps
  for (int i=0;i<3;i++){
    tone(buzzerPin, 1200, 120);
    delay(160);
  }
  delay(800);
}

// Wrong attempt feedback
void wrongAttemptFeedback() {
  // single longer beep + display
  tone(buzzerPin, 400, 300);
  showPrompt("Wrong Password", ("Try left: " + String(attemptsLeft)).c_str());
  delay(800);
}

// Lockout feedback
void startLockout() {
  lockedOut = true;
  lockoutUntil = millis() + LOCKOUT_MS;
  showPrompt("TOO MANY TRIES","Locked temporarily");
  // continuous beeping for short repeated bursts during start
  for (int i=0;i<6;i++){ tone(buzzerPin, 400, 200); delay(220); }
}

// Password change flow
void passwordChangeFlow() {
  // Step 1: ask current password
  inputBuffer = "";
  showPrompt("Change Pass:", "Enter current");
  while (true) {
    char k = keypad.getKey();
    if (k) {
      beep();
      if (k == 'C') { // backspace
        if (inputBuffer.length()) inputBuffer.remove(inputBuffer.length()-1);
      } else if (k == '#') { // submit
        if (inputBuffer == savedPassword) {
          // proceed to new password entry
          break;
        } else {
          tone(buzzerPin, 400, 250);
          showPrompt("Wrong current","password");
          delay(1000);
          return; // abort change
        }
      } else if (k == '*') {
        // cancel change
        showPrompt("Change canceled","");
        delay(700);
        return;
      } else {
        if (isDigit(k) && inputBuffer.length() < MAX_PASS_LEN) inputBuffer += k;
      }
      // show masked and short hint
      lcd.clear();
      lcd.setCursor(0,0); lcd.print("Change Pass:"); lcd.setCursor(0,1);
      for (unsigned int i = 0; i < inputBuffer.length(); ++i) lcd.print('*');
    }
  }

  // Step 2: enter new password
  String newPass = "";
  String confirmPass = "";
  // get new
  showPrompt("Enter new pass","");
  inputBuffer = "";
  while (true) {
    char k = keypad.getKey();
    if (k) {
      beep();
      if (k == 'C') {
        if (inputBuffer.length()) inputBuffer.remove(inputBuffer.length()-1);
      } else if (k == '#') {
        if (inputBuffer.length() >= 1) { newPass = inputBuffer; break; }
      } else if (k == '*') { showPrompt("Change canceled",""); delay(700); return; }
      else {
        if (isDigit(k) && inputBuffer.length() < MAX_PASS_LEN) inputBuffer += k;
      }
      lcd.clear(); lcd.setCursor(0,0); lcd.print("Enter new pass"); lcd.setCursor(0,1);
      for (unsigned int i = 0; i < inputBuffer.length(); ++i) lcd.print('*');
    }
  }

  // confirm
  showPrompt("Confirm new pass","");
  inputBuffer = "";
  while (true) {
    char k = keypad.getKey();
    if (k) {
      beep();
      if (k == 'C') {
        if (inputBuffer.length()) inputBuffer.remove(inputBuffer.length()-1);
      } else if (k == '#') {
        if (inputBuffer.length() >= 1) { confirmPass = inputBuffer; break; }
      } else if (k == '*') { showPrompt("Change canceled",""); delay(700); return; }
      else {
        if (isDigit(k) && inputBuffer.length() < MAX_PASS_LEN) inputBuffer += k;
      }
      lcd.clear(); lcd.setCursor(0,0); lcd.print("Confirm pass"); lcd.setCursor(0,1);
      for (unsigned int i = 0; i < inputBuffer.length(); ++i) lcd.print('*');
    }
  }

  if (newPass == confirmPass) {
    savedPassword = newPass;
    savePasswordToEEPROM(savedPassword);
    showPrompt("Password Saved","");
    for (int i=0;i<3;i++){ tone(buzzerPin, 1200, 120); delay(150); }
    delay(800);
  } else {
    showPrompt("Mismatch!","Try again");
    tone(buzzerPin, 400, 300);
    delay(900);
  }
}

void setup() {
  pinMode(buzzerPin, OUTPUT);
  digitalWrite(buzzerPin, LOW);

  Serial.begin(9600);
  lcd.init();
  lcd.backlight();

  // load password, fallback default "1234" if EEPROM uninitialized
  savedPassword = loadPasswordFromEEPROM();

  // If EEPROM was uninitialized (we used default 1234), store it for persistence
  // (Only do this if what was loaded is "1234" and EEPROM didn't previously store)
  // We'll detect uninitialized by reading PASS_ADDR and seeing 0xFF
  byte raw = EEPROM.read(PASS_ADDR);
  if (raw == 0xFF) { // uninitialized EEPROM
    savedPassword = "1234";
    savePasswordToEEPROM(savedPassword);
  }

  attemptsLeft = MAX_ATTEMPTS;
  inputBuffer = "";
  lockedOut = false;
  showPrompt("Enter Pass:","# to submit");
  lcd.setCursor(0,1);
}

void loop() {
  // handle lockout timeout
  // =============================================
// PERMANENT LOCKOUT – only BT can unlock
// =============================================
if (lockedOut)
{
    // ----- 1. Check Bluetooth for UNLOCK command -----
    if (Serial.available()) {
        String msg = Serial.readStringUntil('\n');
        msg.trim();

        if (msg == "UNLOCK") {
            lockedOut = false;
            attemptsLeft = MAX_ATTEMPTS;

            showPrompt("BT Unlocked", "");
            delay(1000);

            inputBuffer = "";
            showPrompt("Enter Pass:", "# to submit");
            return;
        }
    }

    // ----- 2. Always display LOCKED message -----
    lcd.setCursor(0,0);
    lcd.print("SYSTEM LOCKED   ");     // 16 chars
    lcd.setCursor(0,1);
    lcd.print("Wait BT:UNLOCK  ");     // keeps refreshing

    delay(200);
    return;  // stop all keypad function
}

  char k = keypad.getKey();
  if (!k) return;

  // beep on every keypress
  beep();

  // Controls:
  // '#' -> submit / enter
  // '*' -> start password change flow
  // 'C' -> backspace
  // digits append

  if (k == 'C') {
    if (inputBuffer.length() > 0) inputBuffer.remove(inputBuffer.length() - 1);
    showInputMasked();
    return;
  }

  if (k == '*') {
    // start password change flow
    passwordChangeFlow();
    // after change flow, return to main prompt
    inputBuffer = "";
    showPrompt("Enter Pass:","# to submit");
    return;
  }

  if (k == '#') {
    // submit current typed password
    if (inputBuffer.length() == 0) {
      showPrompt("No input","Enter digits");
      delay(700);
      showInputMasked();
      return;
    }

    if (inputBuffer == savedPassword) {
      unlockedSequence();
      // After unlocking, reset attempts and buffer
      attemptsLeft = MAX_ATTEMPTS;
      inputBuffer = "";
      showPrompt("Enter Pass:","# to submit");
      return;
    } else {
      attemptsLeft--;
      if (attemptsLeft == 0) {
        startLockout();
        inputBuffer = "";
        return;
      } else {
        wrongAttemptFeedback();
        inputBuffer = "";
        showPrompt("Enter Pass:","# to submit");
        return;
      }
    }
  }

  // If key is a digit (0-9) append (ignore letters A,B,D etc)
  if (isDigit(k)) {
    if (inputBuffer.length() < MAX_PASS_LEN) {
      inputBuffer += k;
    } else {
      // optional: indicate max length reached
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("Max length");
      delay(500);
    }
    showInputMasked();
    return;
  }

  // Ignore A,B,D (or you may map them if you want)
  // Provide hint for available controls if pressing other keys:
  if (k == 'A' || k == 'B' || k == 'D') {
    // show small hint line
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Use digits, C,#,*");
    lcd.setCursor(0,1);
    lcd.print("C=back #=enter *=chg");
    delay(700);
    showInputMasked();
    return;
  }


}
