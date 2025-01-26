The **STA/LTA (Short-Term Average / Long-Term Average)** algorithm implemented in the code is used to detect seismic anomalies, such as earthquakes, by analyzing changes in accelerometer data. Here's a detailed explanation of this part of the code:

---

### **How the STA/LTA Algorithm Works**
1. **Short-Term Average (STA)**:
   - Averages the absolute acceleration values over a short, recent time window.
   - Captures sudden, high-energy signals like tremors or shocks.

2. **Long-Term Average (LTA)**:
   - Averages the absolute acceleration values over a longer time window.
   - Represents the baseline or steady-state energy level of the system.

3. **STA/LTA Ratio**:
   - Calculated as \( \text{STA} / \text{LTA} \).
   - A high STA/LTA ratio indicates a significant deviation from the baseline (potential seismic activity).

4. **Thresholding**:
   - If the STA/LTA ratio exceeds a predefined threshold (\( \text{thresh} \)), the system classifies the event as a potential earthquake.

---

### **Implementation in Code**

#### 1. **Data Storage**
The code uses a **FIFO queue** to store accelerometer data. The queue holds enough samples to calculate both STA and LTA.

```cpp
cppQueue StaLtaQue(sizeof(AccelReading), 352, FIFO); // 11 seconds of Accelerometer data
```

- **Queue Length (`QUE_len`)**: Total number of samples for analysis.
- **STA Length (`STA_len`)**: Number of samples for the short-term window.
- **LTA Length (`LTA_len`)**: Number of samples for the long-term window.

---

#### 2. **Offset Calculation**
The algorithm first calculates an **offset** (mean of the entire queue). This removes any constant bias in the accelerometer readings.

```cpp
for (int idx = 0; idx < queCount; idx++) {
    if (StaLtaQue.peekIdx(&AccelRecord, idx)) {
        sample[0] = AccelRecord.x;
        sample[1] = AccelRecord.y;
        sample[2] = AccelRecord.z;
        for (int j = 0; j < 3; j++) {
            sampleSUM[j] += sample[j];
        }
    }
}
for (int j = 0; j < 3; j++) {
    offset[j] = sampleSUM[j] / (QUE_len);
}
```

---

#### 3. **Calculate LTA**
The **LTA** is calculated by averaging the absolute differences from the offset over a long-term window.

```cpp
for (int idx = 0; idx < LTA_len; idx++) {
    if (StaLtaQue.peekIdx(&AccelRecord, idx)) {
        sampleABS[0] = abs(AccelRecord.x - offset[0]);
        sampleABS[1] = abs(AccelRecord.y - offset[1]);
        sampleABS[2] = abs(AccelRecord.z - offset[2]);
        for (int j = 0; j < 3; j++) {
            sampleSUM[j] += sampleABS[j];
        }
    }
}
for (int j = 0; j < 3; j++) {
    ltav[j] = sampleSUM[j] / (LTA_len);
}
```

---

#### 4. **Calculate STA**
The **STA** is similarly calculated over a shorter, recent window within the queue.

```cpp
for (int idx = LTA_len - STA_len; idx < LTA_len; idx++) {
    if (StaLtaQue.peekIdx(&AccelRecord, idx)) {
        sampleABS[0] = abs(AccelRecord.x - offset[0]);
        sampleABS[1] = abs(AccelRecord.y - offset[1]);
        sampleABS[2] = abs(AccelRecord.z - offset[2]);
        for (int j = 0; j < 3; j++) {
            sampleSUM[j] += sampleABS[j];
        }
    }
}
for (int j = 0; j < 3; j++) {
    stav[j] = sampleSUM[j] / STA_len;
    stalta[j] = stav[j] / ltav[j]; // STA/LTA ratio
}
```

---

#### 5. **Threshold Comparison**
The STA/LTA ratio is compared to the threshold (\( \text{thresh} \)) for each axis. If it exceeds the threshold, it indicates potential seismic activity.

```cpp
if (bPossibleEarthQuake == false) {
    if (stalta[j] >= thresh) {
        Serial.printf("STA/LTA = %f = %f / %f (%i)\n", stalta[j], stav[j], ltav[j], j);
        bPossibleEarthQuake = true; // Mark as potential earthquake
    }
}
```

---

#### 6. **Post Detection Actions**
- If an earthquake is detected:
  - Sends a **10-second history** of accelerometer data to the cloud.
  - Activates the **buzzer** and **LED alerts**.

```cpp
if (bPossibleEarthQuake) {
    Serial.println("Start sending 5 minutes of live accelerometer data");
    numSecsOfAccelReadings = 300; // 5 minutes
    Send10Seconds2Cloud(); // Send the historical data
    bPossibleEarthQuake = false;
}
```

---

### **Summary of STA/LTA Steps**
1. **Collect Data**: Store accelerometer samples in a queue.
2. **Calculate Offset**: Compute a baseline offset.
3. **Compute Averages**:
   - Compute LTA over a long-term window.
   - Compute STA over a short-term window.
4. **Calculate STA/LTA Ratio**:
   - Divide STA by LTA for each axis.
5. **Classify Event**:
   - Compare STA/LTA ratio with a threshold.
   - If exceeded, classify as potential seismic activity.

This method is computationally efficient and works well for real-time earthquake detection on resource-constrained devices like the ESP32.
