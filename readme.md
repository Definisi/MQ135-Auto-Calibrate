# Dokumentasi Sensor MQ135 🚗🌫️

## Overview 🛠️

Proyek ini menggunakan sensor gas MQ135 untuk mengukur konsentrasi CO2 dan CO (Karbon Monoksida) di udara. Sensor ini memberikan pembacaan analog yang diproses untuk menghitung konsentrasi gas dalam satuan parts per million (PPM). Kode di bawah ini mencakup fungsionalitas untuk kalibrasi dan pengukuran.

## Penjelasan Kode 📜

### Library 📚
```cpp
#include <AverageValue.h>
```

Library `AverageValue` digunakan untuk menghitung rata-rata konsentrasi dari beberapa pembacaan.

### Konstanta dan Variabel 🧮
```cpp
AverageValue<float> avgCO2(3);  // Untuk rata-rata konsentrasi CO2
AverageValue<float> avgCO(3);   // Untuk rata-rata konsentrasi CO
const int analogPin = A0;       // Pin analog untuk input sensor

float R0 = 35183.33;            // Nilai R0 default untuk kalibrasi
const float A_CO2 = 116.602;    // Konstanta kalibrasi untuk CO2
const float B_CO2 = -2.769;     // Konstanta kalibrasi untuk CO2
const float A_CO = 40.0;        // Konstanta kalibrasi untuk CO (perbarui sesuai kebutuhan)
const float B_CO = -1.5;        // Konstanta kalibrasi untuk CO (perbarui sesuai kebutuhan)
```

### Fungsi Kalibrasi 🧪
```cpp
void calibrateSensor(float currentPPM, float &R0, const float A, const float B) {
  int numReadings = 100;
  float totalResistance = 0;

  for (int i = 0; i < numReadings; i++) {
    int sensorValue = analogRead(analogPin);
    float voltage = sensorValue * (5.0 / 1023.0);
    float R_S = (5.0 - voltage) * 10000.0 / voltage;
    totalResistance += R_S;
    delay(100);
  }

  float averageResistance = totalResistance / numReadings;
  R0 = averageResistance / pow(currentPPM / A, 1.0 / B);
  Serial.print("R0 yang dihitung: ");
  Serial.println(R0);
}
```

Fungsi ini mengkalibrasi sensor dengan menghitung rata-rata resistansi dan memperbarui `R0` berdasarkan nilai PPM yang diberikan.

### Fungsi Setup 🔧
```cpp
void setup() {
  Serial.begin(9600);
  calibrateSensor(425, R0, A_CO2, B_CO2); // Kalibrasi untuk CO2
}
```

Inisialisasi komunikasi serial dan melakukan kalibrasi.

### Fungsi Loop 🔄
```cpp
void loop() {
  int sensorValue = analogRead(analogPin);
  float voltage = sensorValue * (5.0 / 1023.0);
  float R_S = (5.0 - voltage) * 10000.0 / voltage;

  float PPM_CO2 = A_CO2 * pow(R_S / R0, B_CO2);
  avgCO2.push(PPM_CO2);

  float PPM_CO = A_CO * pow(R_S / R0, B_CO);
  avgCO.push(PPM_CO);

  Serial.print("Konsentrasi CO2 (PPM): ");
  Serial.println(avgCO2.average());
  
  Serial.print("Konsentrasi CO (PPM): ");
  Serial.println(avgCO.average());

  delay(500);
}
```

## Rumus 🧮

### Menghitung Konsentrasi CO2
Konsentrasi CO2 (PPM) dihitung dengan rumus:

    PPM_CO2 = A_CO2 * (R_S / R0) ^ B_CO2

- `A_CO2` dan `B_CO2` adalah konstanta kalibrasi untuk CO2.
- `R_S` adalah resistansi sensor saat ini.
- `R0` adalah resistansi sensor pada konsentrasi CO2 standar.

### Menghitung Konsentrasi CO
Konsentrasi CO (PPM) dihitung dengan rumus:

    PPM_CO = A_CO * (R_S / R0) ^ B_CO

- `A_CO` dan `B_CO` adalah konstanta kalibrasi untuk CO.
- `R_S` adalah resistansi sensor saat ini.
- `R0` adalah resistansi sensor pada konsentrasi CO standar.


Membaca data sensor, menghitung PPM untuk CO2 dan CO, serta mencetak konsentrasi rata-rata ke Serial Monitor.

## Catatan Kalibrasi 📝
- Pastikan sensor telah stabil sebelum melakukan kalibrasi.
- Perbarui `A_CO` dan `B_CO` dengan nilai dari datasheet untuk pengukuran CO yang akurat.

## Pemecahan Masalah 🛠️
- **Jika pembacaan tidak akurat:** Periksa kabel dan koneksi sensor.
- **Masalah kalibrasi:** Verifikasi konstanta kalibrasi dan pastikan sensor berada di lingkungan bersih saat kalibrasi.
