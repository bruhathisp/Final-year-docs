## **Understanding Absolute Sum from Scratch**

### **1. What is Absolute Sum?**
The **absolute sum** is a way of measuring the total magnitude of numbers, **ignoring their sign** (positive or negative). It is the sum of the absolute values of a set of numbers.

### **2. Absolute Value Basics**
- The **absolute value** of a number is its distance from zero on the number line.
- It is always **positive** (or zero).
- The symbol for absolute value is **|x|**.

#### **Examples:**
\[
|5| = 5
\]
\[
|-3| = 3
\]
\[
|0| = 0
\]

---

### **3. Definition of Absolute Sum**
If we have a list of numbers:

\[
x_1, x_2, x_3, ..., x_n
\]

The **absolute sum** is:

\[
\sum_{i=1}^{n} |x_i|
\]

This means:
1. Take the absolute value of each number.
2. Add them together.

#### **Example Calculation**
Let's say we have the numbers:
\[
-3, 4, -2, 5, -1
\]

1. Convert to absolute values:
   \[
   | -3 | = 3, \quad | 4 | = 4, \quad | -2 | = 2, \quad | 5 | = 5, \quad | -1 | = 1
   \]

2. Sum them up:
   \[
   3 + 4 + 2 + 5 + 1 = 15
   \]

So, the **absolute sum** is **15**.

---

### **4. Why Do We Use Absolute Sum in STA/LTA?**
In **seismic signal processing**, we often deal with **accelerometer data** that includes **positive and negative values** (representing vibrations in different directions). If we just summed the raw values, **positive and negative numbers could cancel out**, making the result misleading.

By using **absolute values**, we ensure that we measure the total strength of the motion, **regardless of direction**.

---

### **5. Applying Absolute Sum in Code (Example)**
#### **C++ Implementation for Absolute Sum**
```cpp
double computeAbsoluteSum(double data[], int size) {
    double sum = 0;
    for (int i = 0; i < size; i++) {
        sum += fabs(data[i]);  // fabs() computes absolute value
    }
    return sum;
}
```
#### **Usage Example**
```cpp
double values[] = {-3, 4, -2, 5, -1};
int size = 5;
double result = computeAbsoluteSum(values, size);
Serial.println(result); // Output: 15
```

---

### **6. How Absolute Sum is Used in STA/LTA**
- When calculating **STA** (short-term average) and **LTA** (long-term average), we sum the absolute values of seismic signals.
- This ensures that the energy of the vibration is correctly measured.

#### **Example in Seismic Processing**
```cpp
double sum = 0;
for (int i = startIndex; i < endIndex; i++) {
    sum += fabs(signal[i]); // Taking absolute sum
}
double STA = sum / STA_Window_Size;
```

---

### **7. Summary**
✅ **Absolute sum** = Sum of **absolute values** (ignores signs).  
✅ **Prevents negative values from canceling out positive values**.  
✅ **Used in STA/LTA to measure signal strength in seismic detection**.  

Would you like more **examples** or **real-world applications**? 🚀



### **STA/LTA Algorithm - Explanation and Noise Baseline Calculation**

The **STA/LTA (Short-Term Average to Long-Term Average) algorithm** is a method used to detect sudden changes in a signal, particularly in **seismic event detection**. It works by comparing recent activity (STA) to the background noise level (LTA). If the ratio of STA to LTA exceeds a threshold, it signals an event.

---

## **1. Explanation of STA, LTA, and STA/LTA Ratio**

### **1.1 Short-Term Average (STA)**
- **Formula**:  
  \[
  STA(t) = \frac{1}{N_s} \sum_{i=t-N_s+1}^{t} |x(i)|
  \]
- **Meaning**:  
  - Computes the **average absolute amplitude** of the signal **over a short time window** (\( N_s \)).
  - Represents the **current seismic activity**.
  - **Goal**: Capture **sudden motion or spikes** that might indicate an earthquake.

- **Example Calculation (Assuming \( N_s = 32 \)):**
  - If the last 32 samples are:  
    \( [0.2, 0.3, 0.5, 0.4, 0.6, ..., 0.7] \)
  - Compute the **absolute sum**, then divide by 32.

---

### **1.2 Long-Term Average (LTA)**
- **Formula**:  
  \[
  LTA(t) = \frac{1}{N_l} \sum_{i=t-N_l+1}^{t} |x(i)|
  \]
- **Meaning**:  
  - Computes the **average absolute amplitude** over a **longer window** (\( N_l \)), usually **10× longer than STA**.
  - Represents the **background noise level**.
  - **Goal**: Provide a stable **baseline** to compare with STA.

- **Example Calculation (Assuming \( N_l = 320 \)):**
  - If the last 320 samples are:
    \( [0.1, 0.2, 0.15, 0.05, 0.3, ..., 0.25] \)
  - Compute the **absolute sum**, then divide by 320.

---

### **1.3 STA/LTA Ratio for Event Detection**
- **Formula**:  
  \[
  R(t) = \frac{STA(t)}{LTA(t)}
  \]
- **Event Detection Condition**:
  \[
  R(t) > T
  \]
  - If \( R(t) \) exceeds **threshold \( T \)** (e.g., **3 to 5**), a **seismic event is detected**.

- **Why is this useful?**
  - If **STA is much higher than LTA**, it means there is a **sudden increase in signal energy**, indicating **potential earthquake motion**.
  - If **STA ≈ LTA**, it means **normal background vibrations**.

---

## **2. Noise Baseline Calculation**
To **dynamically adjust the threshold**, we use a **noise baseline**. This helps adapt the detection system to different environments.

### **2.1 How is Noise Baseline Calculated?**
- The **noise baseline** is computed as the **average absolute value of past signal samples**:
  \[
  \text{NoiseBaseline}(t) = \frac{1}{M} \sum_{i=t-M+1}^{t} |x(i)|
  \]
  - \( M \) = Total number of samples used to calculate baseline.
  - This smooths out **random fluctuations**.

- **Implementation in Code**:
  ```cpp
  void updateNoiseBaseline() {
      AccelReading sample;
      double sum[3] = {0, 0, 0};

      for (int i = 0; i < QUEUE_LEN; i++) {
          if (StaLtaQueue.peekIdx(&sample, i)) {
              sum[0] += fabs(sample.x);
              sum[1] += fabs(sample.y);
              sum[2] += fabs(sample.z);
          }
      }

      for (int i = 0; i < 3; i++) {
          noiseBaseline[i] = sum[i] / QUEUE_LEN;
          dynamicThreshold[i] = noiseBaseline[i] * THRESHOLD_SCALING_FACTOR;
      }
  }
  ```
  - **Step 1**: Read all stored samples in the FIFO queue.
  - **Step 2**: Compute their absolute sum.
  - **Step 3**: Divide by the queue length to get an average.
  - **Step 4**: Scale it using a factor (e.g., **1.2**) to set **dynamic thresholds**.

---

### **2.2 Why Do We Use a Noise Baseline?**
✅ **Adapts to different environments** (quiet vs. noisy).  
✅ **Prevents false alarms** from normal vibrations.  
✅ **Automatically updates detection sensitivity**.

---

## **3. Threshold Calculation and Dynamic Adjustment**
### **3.1 Fixed Threshold vs. Dynamic Threshold**
- **Fixed Threshold**: A constant \( T \) (e.g., **4.0**) may work **only in some environments**.
- **Dynamic Threshold**:
  \[
  T(t) = \text{NoiseBaseline}(t) \times k
  \]
  - **\( k = 1.2 \)** (Scaling Factor)
  - Adjusts automatically based on recent background noise.

- **Code Implementation**:
  ```cpp
  dynamicThreshold[i] = noiseBaseline[i] * THRESHOLD_SCALING_FACTOR;
  ```
  - **Higher noise** → Higher threshold (**avoids false positives**).
  - **Lower noise** → Lower threshold (**increases sensitivity**).

---

## **4. Example STA/LTA Computation**
### **Given:**
- **STA Window** = 32 samples
- **LTA Window** = 320 samples
- **Threshold \( T \) = 4.0**
- **Seismic signal values**:
  ```text
  Sampled acceleration data:
  [0.2, 0.3, 0.5, 0.4, 0.6, ..., 0.7]
  ```

### **Step-by-Step Calculation**
1. **Compute STA at \( t = 100 \)**:
   \[
   STA(100) = \frac{1}{32} \sum_{i=69}^{
