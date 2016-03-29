---
title: Fiscal Framework 2.1.0 API Reference

language_tabs:
  - java
  - python

toc_footers:
  - <a href='#'>Datecs Ltd.</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors
  - api/fprint
  - changelog/2-1-0
  - changelog/2-0-0

search: true
---

# Introduction

Datecs Fiscal Framework provides Java 6+ base classes for communication and common APIs for developers and users
to create POS applications or use Datecs Ltd. fiscal devices functionality. Also provides and extensions by country,
peripherals controls and helper methods for building more easily simple or complex fiscal applications. There is included support and interpreter
for old Datecs fiscal DSL called `FPrint`.

Because variety of fiscal devices and they capabilities, SDK is separated in:

1. Framework - which holds are common and base classes for communication, error codes, constants etc.
1. Common fiscal methods in `FiscalProtocol` interface which are implemented by all devices, by default.
2. Controls - that extends `FiscalProtocol` adding support for different peripherals
3. Extensions - that extends `FiscalProtocol` adding specific for given country common used fiscal methods.

<aside class="notice">
Controls and extensions availability can be checked at runtime using device capabilities and properties.
</aside>

Every specific fiscal device implementation lies on `FiscalProtocol` combined
with implementation of all supported controls and extensions. You can check if device supports given control or capability using `device capabilities`.
Also there are `device properties` and `device parameters` we will discuss later.

<aside class="notice">
All device classes extends our fiscal protocol interface and implement it.
</aside>

## Supported devices

| Name      | Localization | Supported from |
|:----------|:-------------|:--------------:|
| FMP-705KL |   Bulgaria   |      2.1.0     |
| WP-500KL  |   Bulgaria   |      2.1.0     |
| DP-15KL   |   Bulgaria   |      2.1.0     |
| DP-150KL  |   Bulgaria   |      2.1.0     |
| FP-60KL   |   Bulgaria   |      2.1.0     |
| FP-2000KL |   Bulgaria   |      2.1.0     |
| FP-700KL  |   Tanzania   |      2.1.0     |
| FMP-10KL  |   Bulgaria   |      2.0.0     |
| FMP-350KL |   Bulgaria   |      2.0.0     |

# Initializing

Before start using APIs you must retrieve instance of `FiscalDevice` class which implements `FiscalProtocol` interface.

For this purpose you must have connected device and two streams - `InputStream` and an `OutputStream` for communication. There's no limitations where from you get these streams.
Can be from Bluetooth, USB, TCP/IP connection.

After this using these streams you must create instance to work with, using one of the following methods:

1. FiscalDetector - auto detecting connected device
2. Explicitly - if you know device model

## Using auto detection

> Suppose we have connected device and streams for communication

```java
InputStream is = ...
OutputStream os = ...

FiscalDevice device = FiscalDetector.create(is, os);
```

<aside class="warning">
Correct auto detection is not 100% guarantee, because lies on internal firmware implementation.
</aside>

### Check connected device

Using auto detect in some cases we need to know at runtime the type or model of device we have connected.
For this purpose SDK expose handy parameters and easy methods to check this for every device.

| Property                            | Example value      |
| :---------------------------------- | :---------------   |
| `FiscalProperties.DEVICE_MODEL`     | `FMP350`           |
| `FiscalProperties.DEVICE_VARIANT`   | `KL`               |
| `FiscalProperties.DEVICE_COUNTRY`   | `Country.BULGARIA` |
| `FiscalProperties.DEVICE_QUALIFIER` | `FMP350KLBGR`      |

> Example of retrieving of device Property

```java
Country deviceLocalization = device.getProperty(FiscalProperties.DEVICE_COUNTRY);
```

## Explicitly

> Suppose we have FMP-350 KL for Bulgaria, connected with streams for communication

```java
InputStream is = ...
OutputStream os = ...

FiscalDevice device = new FMP350KLBGR(in, out);
```

> Or concrete instance

```java
FMP350KLBGR concrete = new FMP350KLBGR(in, out);
```

<aside class="warning">
Use this method to create device instance only if you know the physical device you work with.
</aside>

If you initialize wrong device type of physical one, you can get strange commands behavior.
Most of the commands can failed on execution.

# Controls
Controls are pluggable modules when describing device. Example of pluggable controls are cutter,
support for external display and so on. Imagine controls as protocol add-ons.

Support for each control can be checked runtime by corresponding capability. If capability is `true`, device have support for this control.

> Example device class declaration

```java
public final AwesomeFiscalPrinter extends FiscalDevice implements BarcodeControl, DisplayControl, JournalControl {
  ...
  // Implementations of all `FiscalProtocol` methods
  ...
  // Implementations of all controls methods
}
```

| Control     | Control interface        |
| ----------- | -----------------        |
| Barcode     | BarcodeControl           |
| Display     | DisplayControl           |
| Cutter      | CutterControl            |
| Drawer      | DrawerControl            |
| Journal     | JournalControl           |
| Rotated Rec | RotatedReceiptControl    |

## Check control Capability

After we retrieve instance of our connected device and we want to use drawer support for example.

- First we need to check device capability.
- Then we must cast our device to drawer control interface
- Execute controls methods

> Suppose we have connected device and streams for input and output operations

```java
FiscalDevice device ...
```

> Then we check if device support drawer and if support it, we can use drawer control methods

```java

if (device.getBooleanCapability(DrawerControl.CAP_DRAWER)) {
  // Device support it
  DrawerControl drawer = (DrawerControl) device;
  drawer.openDrawer(100);
} else {
  // Throw exception or handle that device doesn't support drawer control
}

```

| Control    | Capability constant                            |
|------------|------------------------------------------------|
|Barcode     | ```BarcodeControl.CAP_BARCODE```               |
|Display     | ```DisplayControl.CAP_EXT_DISPLAY```           |
|Cutter      | ```CutterControl.CAP_CUTTER```                 |
|Drawer      | ```DrawerControl.CAP_DRAWER```                 |
|Journal     | ```JournalControl.CAP_JOURNAL```               |
|Rotated Rec | ```RotatedReceiptControl.CAP_ROT_NON_FISCAL``` |

# Extensions
Extensions are like controls but are country oriented and provides custom common methods for given country.

<aside class="success">
Rembmber - If you have instance of Tanzanian device for sure you can cast your device to Tanzanian extension without checking any properties or capabilities.
</aside>

For example if we have instance of Tanzanian FP-700 KL device class.

We can cast without any additional checks to `TanzaniaExtension` and use it's methods as follows:

> Suppose we have connected device and streams for communication

```java
InputStream input = ...
OutputStream output = ...
FiscalDevice tanzanianPrinter = new FP700KLTNZ(input, output);
```

> We cast without additional checks

```java
TanzaniaExtension ext = (TanzaniaExtension) tanzanianPrinter;
```

| Available extensions   |
| :--------------------: |
| `TanzaniaExtension`    |

# Helpers
Helper classes wraps protocol and provide nice and easy methods for common operations.
There are also classes which use Java core classes advantages to make some operations,
command and logic more easy and clean.

## Usage

Make sure your printer is in normal state before executing single or multiple commands.
If your printer stuck in some not working state, SDK can try to set it in normal state
using `checkAndResolve`.

> Check and resolve device problems

```java
FiscalDevice device = ...;
device.checkAndResolve();

```

### Simple Sales
### Simple display
### PLU item iterator

# Usage

Make sure your printer is in normal state before executing single or multiple commands.
If your printer stuck in some not working state, SDK can try to set it in normal state
using `checkAndResolve`.

> Check and resolve device problems

```java
FiscalDevice device = ...;
device.checkAndResolve();

```

## Sales
To sale something there are some things you might know:

> Open fiscal receipt with operator

```java
final int operatorNum = ...
final String operatorPassword = ...

final FiscalDevice device = ...

try {
  device.openFiscalReceipt(operatorNum, operatorPassword, false);
} catch (Exception e) {
  if (e instanceof NotSupportedException) {
    // Handle command not supported
    // If some of most common methods contains in FiscalDevice is not supported
    // This is most likely a bug and you must report it
  } else if (e instanceof FiscalException) {
    final String msg = ((FiscalException) e).getErrorMessage();
    // Handle this error. Is reported by device
  } else if (e instanceof IOException) {
    // Handle communication problem
  }
}
...
```

> Example sale in Tax group A with quantity of 1 without any adjustments

```java
...

final String item = "Fiscal printer FP-60"

try {
  device.sell(item, 123.42, TaxGroup.A, 1);
  device.subtotal();
  device.total();
  device.closeFiscalReceipt();
} catch (...) {
  ...
}
```

> Subtotal with adjustment (optional)

```java
...
try {
  device.subtotal(AdjustmentType.DISCOUNT_BY_PERCENTAGE, 10);
} catch (...) {
  ...
}
```

1. You must open fiscal receipt using registered operator
2. Make sales single or multiple (with or without adjustment)
<aside class="success">Remember - you can execute as many as you want sales in single fiscal receipt</aside>
3. (Optional) Subtotal (with or without adjustment)
4. You **MUST** execute `total` (single or multiple time, with same of different payment modes) to pay before closing fiscal receipt
<aside class="warning">Overpaying with credit or debit is not allowed </aside>
5. Closing fiscal receipt

<aside class="success">Remember - is allowed to print fiscal text in a opened receipt </aside>

### Tax groups
Every printer supports several tax groups. Mostly they are 6 or 8 but you can retrieve the right
supported number using printer's properties:

`int supportedRates = device.getIntProperty(FiscalProperties.NUM_VAR_RATES);`

<aside class="notice"> Methods requiring tax group uses `TaxGroup` enum. </aside>

### Paid modes
Printers support all common payment modes and some custom user-defined. If you want to know how many
custom payment you can configure, use printer's properties:

`int customPayments = device.getIntProperty(FiscalProperties.NUM_CUSTOM_PAYMENTS);`

<aside class="notice"> Methods requiring payment uses `PaidMode` enum. </aside>

### Adjustment types
Adjustment types can be discounts or surcharge, both by value or by percentage.

<aside class="notice"> Methods requiring adjustment uses `AdjustmentType` enum. </aside>

### Using PLU items

> Sell using PLU item number

```java
...
try {
  int myAwesomeItemPlu = ...

  device.sellItem(myAwesomeItemPlu);
} catch (...) {
  ...
}
```

If there's programmed items in device you can make sales using PLU number instead of text, price and all required for sale data.

Check [Items](#items) section for more information about how you can program items stored in device.

## Reports
All fiscal devices supports X and Z daily reports, simple and detailed reports of fiscal memory by
document number or by date and time range. Reports are printed out, but you can retrieve data directly from
device's electronic journal (if supported). Check [Journal](#journal) section for more details.

## Items
## Journal
## Service
### Non-fiscal receipt
### Date and time
###
