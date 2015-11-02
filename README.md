# Android Infra Red

Open source project for transmitting signals via IR-blasters on Android devices

Created in Android Studio

Supports Samsung, HTC, some LG devices with IR

Contains:
- SampleApplication
- InfraRed library

Email: one.button.developer@gmail.com


## Menu

- **[Basis](#basis)**
  - [Basic theory](#basic-theory)
  - [Sequence types](#sequence-types)
  - [Converting the patterns](#converting-the-patterns)
- **[How to use](#how-to-use)**
  - [0. Include the InfraRed library to your project](#0-include-the-infrared-library-to-your-project)
  - [1. Add the following permissions to your AndroidManifest.xml](#1-add-the-following-permissions-to-your-androidmanifestxml)
  - [2. Initialize logging](#2-initialize-logging)
  - [3. Initialize the InfraRed class](#3-initialize-the-infrared-class)
  - [4. Initialize the transmitter](#4-initialize-the-transmitter)
  - [5. Initialize the raw patterns](#5-initialize-the-raw-patterns)
  - [6. Adapt the patterns for the device that is used to transmit the patterns](#6-adapt-the-patterns-for-the-device-that-is-used-to-transmit-the-patterns)
  - [7. Implement the start and stop methods](#7-implement-the-start-and-stop-methods)
  - [8. Transmit the adapted patterns via IR](#8-transmit-the-adapted-patterns-via-ir)
  - [In addition](#in-addition)
- **Details**

## Basis

### Basic theory

In order to remotely control a device via IR, you require a transmitter (mobile phone with IR blaster) and a control signal sequence that you want to send. This sequence is the key to the function that the device is supposed to perform (e.g. the function of a TV to turns itself on). The built-in IR receiver of the controlled device analyzes all incoming signals. When they match a pattern of a particular function, the devices calls that function.

The control signal sequence consists of two parameters: The frequency (in _Hertz_) and the pattern, which is a time sequence of _ON_/_OFF_ signals on the specified carrier frequency. Such patterns can be found on the Internet.

### Sequence types

The pattern is pictured by a sequence of numbers either of the hexadecimal (_HEX_) or the decimal (_DEC_) numeral system.

Here's an example:

```
Frequency:
  33000 Hertz (33KHz)
Patterns:
  0) 0x01f4 0x1c84 0x01f4 0x00c8 or (01f4 1c84 01f4 00c8)
  1) 500 7300 500 200
  2) 17 241 17 7
```

The main difference between the patterns in this example is their notation.
Each number in the pattern describes the duration of a single _ON_/_OFF_ signal, only that patterns 0 and 1 describe it in microseconds, while patterns 2 describes it in cycles.

One cycle always consists of an an _ON_ and an _OFF_ signal (the image below shows cycles with a frequency of 40000 _Hz_).

![Difference between sequences](http://s8.postimg.org/s9s4dxumd/91598ad1_1766_469e_8731_99f70f606864.jpg)

### Converting the patterns

The length of a single period _T_ in microseconds equals 1 000 000 (1 second in microseconds) divided by the frequency _f_ (in _Hertz_) 

> _T_ = 1 000 000 _ms_ / _f_

For example

> _T_ = 1 000 000 _ms_/ 33000 _Hz_ ≈ 30.3 _ms_

To covert a microsecond pattern to a cycle pattern, divide each value by _T_

To convert a cycle pattern to a microsecond pattern, simply multiply each value by _T_

## How to use

### 0. Include the InfraRed library to your project

- Download the [AndroidInfraRed](https://github.com/OneButtonDeveloper/AndroidInfraRed) project
- Run the project on a device or an emulator 
- Find the **infrared-release.aar** file in the project folder _(infrared/build/outputs/aar/infrared-release.aar)_
- Import this file into your own project in Android Studio
  _(File > New > New Module... > Import .JAR/.AAR package)_

**Tip!** Your can download the **infrared-release.aar** file from [releases](https://github.com/OneButtonDeveloper/AndroidInfraRed/releases/download/v3.0/infrared-release.aar)

**Known issue!** One of the project files contains broken characters. Just rename it again.

### 1. Add the following permissions to your AndroidManifest.xml

```xml
	<uses-permission android:name="android.permission.TRANSMIT_IR" android:required="false"/>
	<uses-feature android:name="android.hardware.consumerir" android:required="false"/>
```
 
### 2. Initialize logging

You can choose between printing your logs to an _EditText_ view ([_LogToEditText_](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/log/LogToEditText.java)), the _LogCat_ console ([_LogToConsole_](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/log/LogToConsole.java)) and not logging at all ([_LogToAir_](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/log/LogToAir.java))

```java
	// print log messages to EditText
	EditText console = (EditText) this.findViewById(R.id.console);
	log = new LogToEditText(console, TAG);

	// print Log messages with Log.d(), Log.w(), Log.e() (LogCat)
	// LogToConsole log = new LogToConsole(TAG);

	// Turn off logs
	// LogToAir log = new LogToAir(TAG);
```
[_LogToEditText_](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/log/LogToEditText.java) is the default option

### 3. Initialize the [**InfraRed**](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/InfraRed.java) class

Create a new [_InfraRed_](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/InfraRed.java) object with the parameters _Context_ and [_Logger_](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/log/Logger.java)

```java
	infraRed = new InfraRed(this, log);
```

### 4. Initialize the transmitter

```java
	TransmitterType transmitterType = infraRed.detect();

	// initialize transmitter by type
	infraRed.createTransmitter(transmitterType);
```

The "detect()" method returns the [_TransmitterType_](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/transmit/TransmitterType.java). Possible values:  

* Actual - The signals will be transmitted by the ConsumerIrManager  
* Obsolete - The signals will be transmitted by a Samsung IR service
* HTC - The signals will be transmitted by the HTC IR SDK
* LG, LG_WithOutDevice - The signals will be transmitted by the QRemote SDK
* Undefined - the "empty" implementation will be used to transmitt the signals

### 5. Initialize the raw patterns

```java
	// initialize raw patterns
	List<PatternConverter> rawPatterns = new ArrayList<>();
	// Canon
	// rawPatterns.add(new PatternConverter(PatternType.Intervals, 33000, 500, 7300, 500, 200));
	// Nikon D7100 v.1
	rawPatterns.add(new PatternConverter(PatternType.Cycles, 38400, 1, 105, 5, 1, 75, 1095, 20, 60, 20, 140, 15, 2500, 80, 1));
	// Nikon D7100 v.2
	rawPatterns.add(new PatternConverter(PatternType.Cycles, 38400, 77, 1069, 16, 61, 16, 137, 16, 2427, 77, 1069, 16, 61, 16, 137, 16));
	// Nikon D7100 v.3
	rawPatterns.add(new PatternConverter(PatternType.Intervals, 38000, 2000, 27800, 400, 1600, 400, 3600, 400, 200));
	// Nikon D7100 v.3 fromString
	rawPatterns.add(PatternConverterUtils.fromString(PatternType.Intervals, 38000, "2000, 27800, 400, 1600, 400, 3600, 400, 200"));
	// Nikon D7100 v.3 fromHexString
	rawPatterns.add(PatternConverterUtils.fromHexString(PatternType.Intervals, 38000, "0x7d0 0x6c98 0x190 0x640 0x190 0xe10 0x190 0xc8"));
	// Nikon D7100 v.3 fromHexString without 0x
	rawPatterns.add(PatternConverterUtils.fromHexString(PatternType.Intervals, 38000, "7d0 6c98 190 640 190 e10 190 c8"));
```

The [_PatternConverter_](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/patterns/PatternConverter.java) can be used to easily define and convert IR patterns between the different patterns types (Cycles and Intervals in ms)

### 6. Adapt the patterns for the device that is used to transmit the patterns

```java
	PatternAdapter patternAdapter = new PatternAdapter(log);
	
	// initialize TransmitInfoArray
	TransmitInfo[] transmitInfoArray = new TransmitInfo[rawPatterns.size()];
	for (int i = 0; i < transmitInfoArray.length; i++) {
	    transmitInfoArray[i] = patternAdapter.createTransmitInfo(rawPatterns.get(i));
	}
	this.patterns = transmitInfoArray;
```

[_PatternAdapter_](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/patterns/PatternAdapter.java) automatically detects the device's manufacturer (_LG_/_HTC_/_Samsung_) and uses that information to determine how the raw patterns have to be converted in order to be compatible with the transmitter of the used device.

### 7. Implement the start and stop methods

```java
    @Override
    protected void onResume() {
        super.onResume();
        infraRed.start();
    }
   
    @Override
    protected void onDestroy() {
        super.onDestroy();

        infraRed.stop();

        if (log != null) {
            log.destroy();
        }
    }
```

Always call the _start()_ method before transmit IR signals and call the _stop()_ method when you are done.

It is a **good idea** to call them in the _onCreate_/_onDestroy_ or _onResume_/_onPause_ methods.


### 8. Transmit the adapted patterns via IR

```java
    private int currentPattern = 0;

    @Override
    public void onClick(View v) {
        TransmitInfo transmitInfo = patterns[currentPattern++];
        if (currentPattern >= patterns.length) currentPattern = 0;
        infraRed.transmit(transmitInfo);
    }
```

Now we can send IR signals ([_TransmitInfo_](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/java/com/obd/infrared/transmit/TransmitInfo.java)) to the device we want to control

### In addition

Examples for the individual steps can be found in this project

- Step 1 in [AndroidManifest.xml](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/infrared/src/main/AndroidManifest.xml)
- Steps 2..8 in [MainActivity.java](https://github.com/OneButtonDeveloper/AndroidInfraRed/blob/master/app/src/main/java/com/obd/infrared/sample/MainActivity.java)

