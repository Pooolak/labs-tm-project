# TEMAT - Security Alarm System

## Opis projektu:
Projekt będzie przedstawiał działane prostego systemu alarmowego. 

Urządzenia wejścia:
- Przycisk

Urządzenia wyjścia:
- Diody 
- Buzzer
- Laser

Układ będzie realizować proste funkcję centralki alarmowej z jedną czujką czułą (fotorezystorem) na wiązkę laserową. Po podaniu napięcia na układ ziolona dioda powinna kilkukrotnie zaświecić a buzzer podać krótki charakterystyczny dźwięk co będzie oznaką gotowości układu do zazbrojenia. Chcąc zazbroić system należy nacinąć przycisk. Układ odpowie sygnałem akustycznym oraz zapaleniem się czerwonej diody. Po tej czyności układ już jest gotowy. Alarm zostanie wywołany poprzez przecięcie wiązki lasera. W chwili przecięcia wiązki system odpowiada dzwiękiem alarmu oraz migającymi diodami czerwona-pomarańczowa. W celu wyłączenia alarmu należy ponownie nacisnąć przycisk co wywoła powrót do stanu gotowości. 

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

Schemat układu przy pomocy programu Eagle:
![Laser_Project_Eagle](https://user-images.githubusercontent.com/92359546/148217414-2e566ab5-be0b-45f6-aae4-76bcdbb80a10.PNG)

Schemat połączeniowy układu przy pomocy programu Fritzing:
![Laser_Project](https://user-images.githubusercontent.com/92359546/144492375-999c27a2-7f95-49d2-abbf-4a353e71c6eb.png)

Zdjęcia z realizowanego projektu:
![IMG_20211202_205430](https://user-images.githubusercontent.com/92359546/144495005-29b9f9c1-7e35-4b26-97a8-81a57a0df433.jpg)

![IMG_20211202_205440](https://user-images.githubusercontent.com/92359546/144495042-3aeb320b-23fd-4f68-bd41-92ec28f34e4a.jpg)



# Fragment kodu:

```cpp
//definicja rejestrów
const int triggeredLED = 7;
const int triggeredLED2 = 8;
const int RedLED = 4;
const int GreenLED = 5;
const int inputPin = A0;
const int speakerPin = 12;
const int armButton = 6;

boolean isArmed = true; //(true)jeśli przycisk jest uzbrojony i gotowy do uruchomienia
boolean isTriggered = false;  //(false) jeśli przycisk jest "wzbudzony" i gotowy do uruchomienia
int buttonVal = 0;//przycisk
int prev_buttonVal = 0;
int reading = 0;//zmienna przechowująca odczyt czujnika
int threshold = 0;//średni odczyt z czujnika

//dźwięk alarmu
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

  calibrate();//wywołanie kalibracji
  setArmedState();//wywołanie uzbrojenia alarmu
}

void loop() {


  reading = analogRead(inputPin);//odczyt wartości


  int buttonVal = digitalRead(armButton);
  if ((buttonVal == HIGH) && (prev_buttonVal == LOW)) {//jeśłi przycisk jest wciśnięty i wcześniej nie był wciśnięty
    setArmedState();//uzbrojenie alarmu
    delay(500);
  }

  if ((isArmed) && (reading < threshold)) {//wyzwolenie alarmu
    isTriggered = true;//włączenie alarmu
  }

  if (isTriggered) {//jeśli alarm się włączy

    for (int i = lowrange; i <= highrange; i++)//obsługa głośnika, dźwięk rosnący
    {
      tone (speakerPin, i, 250);
    }

    for (int i = highrange; i >= lowrange; i--)//obsługa głośnika, dźwięk "spadający"
    {
      tone (speakerPin, i, 250);
    }

    //obsługa diod
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

void setArmedState() {// uzbrojenie alarmu

  if (isArmed) {//jeśli uzbrojony
    digitalWrite(GreenLED, HIGH);//zapala się zielona dioda
    digitalWrite(RedLED, LOW);//gaśnie czerwona
    isTriggered = false;//wyłączenie alarmu
    isArmed = false;//rozbrojenie
  } else {//jeśli nie
    digitalWrite(GreenLED, LOW);//gaśnie zielony
    digitalWrite(RedLED, HIGH);//zapala się czerwony
    tone(speakerPin, 220, 125);//zakomunikowanie dźwiękiem
    delay(200);
    tone(speakerPin, 196, 250);
    isArmed = true;//uzbrojenie
  }
}

void calibrate() {

  int sample = 0;//zmienna próbka
  int baseline = 0;
  const int min_diff = 200;//zakres błędu odczytu
  const int sensitivity = 50;//czułość
  int success_count = 0;//sprawdzenie poprawności kalibracji

//dioda
  digitalWrite(RedLED, LOW);
  digitalWrite(GreenLED, LOW);

  for (int i = 0; i < 10; i++) {
    sample += analogRead(inputPin);//próbki
    //obsługa diody
    digitalWrite(GreenLED, HIGH);
    delay (50);
    digitalWrite(GreenLED, LOW);
    delay (50);
  }

  do
  {
    sample = analogRead(inputPin);

    if (sample > baseline + min_diff) {//jeśli próbka jest większa niż baseline + błąd pomiarowy
      success_count++;
      threshold += sample;//dodaj wartość próbki
      //obsługa diody
      digitalWrite(GreenLED, HIGH);
      delay (100);
      digitalWrite(GreenLED, LOW);
      delay (100);
    } else {//jeśli nie
      //zerowanie zmiennych
      success_count = 0;
      threshold = 0;
    }

  } while (success_count < 3);

  threshold = (threshold / 3) - sensitivity;//wyznaczenie średniej wartości odczytu czujnika
  //sygnał dźwiękowy
  tone(speakerPin, 196, 250);
  delay(200);
  tone(speakerPin, 220, 125);
}
```

Prezentacja działania układu:
https://youtu.be/Yt8zGT6XYg8

# Podsumowanie

