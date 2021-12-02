# TEMAT - Security Alarm System

## Opis projektu:
Projekt będzie przedstawiał działane prostego systemu alarmowego. 

Urządzenia wejścia:
- Przycisk

Urządzenia wyjścia:
- Diody 
- Buzzer

Układ będzie realizować proste funkcję centralki alarmowej z jedną czujką czułą (fotorezystorem) na wiązkę laserową. Po podaniu napięcia na układ ziolona dioda powinna kilkukrotnie zaświecić a buzzer podać krótki charakterystyczny dźwięk co będzie oznaką gotowości układu do zazbrojenia. Chcąc zazbroić system należy nacinąć przycisk. Układ odpowie sygnałem akustycznym oraz zapaleniem się czerwonej diody. Po tej czyności układ już jest gotowy. Alarm zostanie wywołany poprzez przecięcie wiązki lasera. W chwili przecięcia wiązki system odpowiada dzwiękiem alarmu oraz migającymi diodami czerowna-pomarańczowa. W celu wyłączenia alarmu należy ponownie nacisnąć przycisk co wywoła powrót do stanu gotowości. 

Elementy potrzebne w celu zrealizowania systemu alarmowego:
- Arduino
- Przwody męsko-męskie
- Przewody męsko-damskie
- Płytka stykowa
- Buzzer
- Laser 5v
- Przycisk
- 2x Rezystor 10k
- 4x Rezystor 220
- Fotorezystor
- 4x Dioda

Układ został podłączony zgodnie z poniższym schematem:
![Laser_Project](https://user-images.githubusercontent.com/92359546/144492375-999c27a2-7f95-49d2-abbf-4a353e71c6eb.png)


# Fragment kodu:

```cpp
const int triggeredLED = 7;
const int triggeredLED2 = 8;
const int RedLED = 4;
const int GreenLED = 5;
const int inputPin = A0;
const int speakerPin = 12;
const int armButton = 6;

boolean isArmed = true;
boolean isTriggered = false;
int buttonVal = 0;
int prev_buttonVal = 0;
int reading = 0;
int threshold = 0;


const int lowrange = 2000;
const int highrange = 4000;

void setup() {

  pinMode(triggeredLED, OUTPUT);
  pinMode(triggeredLED2, OUTPUT);
  pinMode(RedLED, OUTPUT);
  pinMode(GreenLED, OUTPUT);
  pinMode(armButton, INPUT);
  digitalWrite(triggeredLED, HIGH);
  delay(500);
  digitalWrite(triggeredLED, LOW);

  calibrate();
  setArmedState();
}

void loop() {


  reading = analogRead(inputPin);


  int buttonVal = digitalRead(armButton);
  if ((buttonVal == HIGH) && (prev_buttonVal == LOW)) {
    setArmedState();
    delay(500);
  }

  if ((isArmed) && (reading < threshold)) {
    isTriggered = true;
  }

  if (isTriggered) {

    for (int i = lowrange; i <= highrange; i++)
    {
      tone (speakerPin, i, 250);
    }

    for (int i = highrange; i >= lowrange; i--)
    {
      tone (speakerPin, i, 250);
    }


    digitalWrite(triggeredLED, HIGH);
    delay(50);
    digitalWrite(triggeredLED, LOW);
    delay (50);
    digitalWrite(triggeredLED2, HIGH);
    delay (50);
    digitalWrite(triggeredLED2, LOW);
    delay (50);
  }

  delay(20);
}

void setArmedState() {

  if (isArmed) {
    digitalWrite(GreenLED, HIGH);
    digitalWrite(RedLED, LOW);
    isTriggered = false;
    isArmed = false;
  } else {
    digitalWrite(GreenLED, LOW);
    digitalWrite(RedLED, HIGH);
    tone(speakerPin, 220, 125);
    delay(200);
    tone(speakerPin, 196, 250);
    isArmed = true;
  }
}

void calibrate() {

  int sample = 0;
  int baseline = 0;
  const int min_diff = 200;
  const int sensitivity = 50;
  int success_count = 0;

  digitalWrite(RedLED, LOW);
  digitalWrite(GreenLED, LOW);

  for (int i = 0; i < 10; i++) {
    sample += analogRead(inputPin);
    digitalWrite(GreenLED, HIGH);
    delay (50);
    digitalWrite(GreenLED, LOW);
    delay (50);
  }

  do
  {
    sample = analogRead(inputPin);

    if (sample > baseline + min_diff) {
      success_count++;
      threshold += sample;

      digitalWrite(GreenLED, HIGH);
      delay (100);
      digitalWrite(GreenLED, LOW);
      delay (100);
    } else {
      success_count = 0;
      threshold = 0;
    }

  } while (success_count < 3);

  threshold = (threshold / 3) - sensitivity;

  tone(speakerPin, 196, 250);
  delay(200);
  tone(speakerPin, 220, 125);
}
```
