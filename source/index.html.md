---
title: Fiscal Framework 2.2.0 API Reference

language_tabs:
  - java

toc_footers:
  - <a href='#'>Datecs Ltd.</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors
  - api/fprint
  - changelog/2-2-0
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

| Name      | Country      | Supported from |
|:----------|:-------------|:--------------:|
| FMP-350D  |   Bulgaria   |      2.2.0     |
| DP-05BKL  |   Bulgaria   |      2.2.0     |
| DP-05KL   |   Bulgaria   |      2.2.0     |
| DP-45KL   |   Bulgaria   |      2.2.0     |
| DP-35KL   |   Bulgaria   |      2.2.0     |
| FP-700CRX |   Moldova    |      2.2.0     |
| FP-700CRX |   Federation Bosnia | 2.1.0   |
| FP-705KL  |   Bulgaria   |      2.1.0     |
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

> Example of retrieving of device info property

```java
Country deviceLocalization = device.getInfo().getDeviceCountry();
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

# Device information

```java
final FiscalDevice device...
final FiscalDeviceInfo information = device.getInfo();

final boolean supportsCutter = info.capCutter();
final int maxPrintTextLen = info.getPrintTextLength();

```

Every `FiscalDevice` instance contains `FiscalDevceInfo` object which can be retrieved by `getInfo()` method. This class return capabilities and properties
of currently connected device. Methods starting with `cap` return capabilities. Starting with `get` returns device properties.

For example `device.getInfo().capPrintDetailedReportByDate()` will return `boolean` indicating whenever currently connected device supports print of detailed report by date.

<aside class="warning">Fiscal device information must be used instead of obsoleted properties and capabilities.</aside>

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

> Checking if device support drawer and if support it, we can use drawer control methods

```java
FiscalDevice device ...

...

if (device.getInfo().getCapDrawer()) {
  // Device support it
  DrawerControl drawer = (DrawerControl) device;
  drawer.openDrawer(100);
} else {
  // Throw exception or handle that device doesn't support drawer control
}

```

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
| ---------------------- |
| `TanzaniaExtension`    |
| `BosniaExtension`      |

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

> Create instance of SimpleReceipt helper class

```java
SimpleReceipt receipt = SimpleReceipt.create(device);
```

All helper classes are created using `FiscalDevice` instance with the static method `create()`
on the given helper class, for example:

`SimpleDisplay easy = SimpleDisplay.create(device);`

### Simple receipt

`SimpleReceipt` helps executing widely used both fiscal or non-fiscal commands.

For example with `SimpleReceipt` you can:

- Batch print of non-fiscal text lines
- Make sales using builders

Builders can be saved and changed to create easy batch of sales, subtotals and totals.

### Simple display

`SimpleDisplay` extends `DisplayControl` functionality and add:

- Floating text support
- Multiline text support

### PLU item iterator

> Example usage of PLU item iterator

```java
// By default: forward direction, starting from first, iterating all programmed items
PluItemIterator iterator = PluItemIterator.create(device);

// or

iterator = PluItemIterator.create(device, Direction.FORWARD, Type.ITER_PROGRAMMED, 1);

...

// Now iterate

while (iterator.hasNext()) {
  PluItem nextItem = iterator.next();

  // PluItem object provides getters for all supported fields

  nextItem.getName();
  nextItem.getPrice();

  ...
}

```

`PluItemIterator` provide Java style iteration of programmed items. Supports both
forward and backward direction of iteration and also can iterate over programmed,
these with sales and non programmed items. You can specify it by creating iterator
with chosen type. Iteration will stop when there are no more items which not meet
the requirements.

Also wraps `FiscalResponse` in a more usable and handy object called `PluItem`.

### Text formatter

`TextFormatter` helps aligning and formatting text which can span on whole line.
This is useful when creating or using receipt text templates on different devices
or different paper size. `TextFormatter` will get current connected device available
text length to span the text. This will work on all devices.

# APIs Usage

> Check and resolve device problems

```java
FiscalDevice device = ...;
device.checkAndResolve();
```

Make sure your printer is in normal state before executing single or multiple commands.
If your printer stuck in some not working state, SDK can try to set it in normal state
using `checkAndResolve`.


Take a look on [customCommand](#custom-command) if there's missing methods on the API,
but still supported by printer, but be careful. Because of variety of devices and
countries this API define subset of all commands and it's a superset of all common
and widely used commands. Before writing your own logic using `customCommand` please
check if the desired command exists in device country extension or some of
the peripheral controls.

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

1. You must open fiscal receipt using registered operator using:

    `openFiscalReceipt(int operator, String password, boolean invoice);`

2. Make sales single or multiple (with or without adjustment) using on of following:

    `sell(String item, double price, TaxGroup tax, double quantity);`

    `sell(String item, double price, TaxGroup tax, double quantity, AdjustmentType type, double value);`

    <aside class="success">Remember - you can execute as many as you want sales in single fiscal receipt</aside>
3. (Optional) Subtotal (with or without adjustment) using one of:

    `subtotal();`

    `subtotal(AdjustmentType type, double value);`

4. You **MUST** execute `total` (single or multiple time, regardless payment mode) to pay before closing receipt

      All in cash: `total();`

      All in desired paid mode: `total(PaidMode mode);`

      Custom amount in desired paid mode: `total(PaidMode mode, double amount);`

      <aside class="warning">Overpaying with credit or debit is not allowed </aside>
5. Closing fiscal receipt

    `closeFiscalReceipt()`

<aside class="success">Remember - is allowed to print fiscal text in a opened receipt </aside>

```java
device.sell(...);
...
device.sell(...);
...
device.subtotal(...);

// Print text
device.printFiscalText("Promo code: 42424242");

device.total();

for (String line : lines) {
  device.printFiscalText(line);
}

```

### Tax groups
Every printer supports several tax groups. Mostly they are 6 or 8 but you can retrieve the right
supported number using printer's properties:

`int supportedRates = device.getInfo().getVatRates();`

<aside class="notice"> Methods requiring tax group uses `TaxGroup` enum. </aside>

### Paid modes
Printers support all common payment modes and some custom user-defined. If you want to know how many
custom payment you can configure, use printer's properties:

`int customPayments = device.getInfo().getCustomPayments();`

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

> Don't forget to close fiscal receipt when you're done with the sales

```java
device.closeFiscalReceipt();
```

## Reports
All fiscal devices supports X and Z daily reports, simple and detailed reports of fiscal memory by
document number or by date and time range. Reports are printed out, but you can retrieve data directly from
device's electronic journal (if supported). Check [Journal](#journal) section for more details.

To print X or Z report use respectively `device.printZReport()` or `device.printXReport()`;

<aside class="notice">Date and time format is: dd-MM-yy hh:mm:ss</aside>

For report by date or number use:

- `printReportByDate()`
- `printReportByNumber()`
- `printDetailedReportByDate()`
- `printDetailedReportByNumber()`

## Items

### Available methods

| Method                    | Description |
| ------------------------- | ----------- |
| `defineItem()`            | Program item |
| `changeItemQuantity()`    | Change item availability
| `getItemsInfo()`          | Get info about items
| `getItem()`               | Get single item by plu number
| `getFirstItem()`          | Get first programmed item
| `getLastItem()`           | Get last programmed item
| `getNextItem()`           | Get next programmed item
| `getFirstItemWithSales()` | Get first item with sales
| `getLastItemWithSales()`  | Get last item with sales
| `getNextItemWithSales()`  | Get next item with sales
| `getFirstFreeItem()`      | Get first free item to be programmed
| `getLastFreeItem()`       | Get last free item to be programmed
| `deleteItem()`            | Delete item by plu number
| `deleteItems()`           | Delete all items
| `deleteItems(int, int)`   | Delete all items in a range

### Work with items

Most of Datecs fiscal devices supports programmable items. There are APIs making
item sales using plu number.

Iterating over items can be achieved like using C++/Java style iterator. First you
point the iterator to position, for example `getFirstItem()` or `getFirstItemWithSales()`
and increment the iterator's position every time you want to get the next element.
Incrementing in this context is executing `getNextItem()` or `getNextItemWithSales()`.
Iteration must stop when command return error, so you have to check your self the result.

Getting item will return `FiscalResponse` object which in case of items will contains all or most of fields in `FiscalPluItemParams`:

| Field             |
| ----------------- |
| ITEM_NAME         |
| ITEM_PLU          |
| ITEM_PRICE        |
| ITEM_TOTAL        |
| ITEM_SOLD         |
| ITEM_STOCK        |
| ITEM_STOCK_GROUP  |

Check [helpers section](#helpers) about PLU item iterator which makes iterating items more simple and easy.
Also wraps `FiscalResponse` in a more friendly `PluItem` object.

## Journal

<aside class="warning">Before start working with journal make sure your device supports it.</aside>

[Here](#check-control-capability) you find how to check if device support some peripheral control

Most of the devices supports EJ (aka. Electronic Journal). EJ holds all both
fiscal and non-fiscal data in energy independent memory. EJ has it's own capacity.
In some situations you must have read access to old documents. Datecs Fiscal Framework
provides simple way to retrieve documents from the journal, by name or by some date time range.

### Get journal documents



### Get last document from the journal

> Get last document from journal

```java
JournalControl ctrl = ...;

final FiscalResponse rspn = ctrl.getJournalDocumentRange();

// Check last document number

int last = rspn.getInt(JournalControl.RESP_LAST_DOC);

if (last != -1) {
  ctrl.printJournalDocument(last);

  // or if we want to get it

  String document = ctrl.getJournalDocument(last).getString(JournalControl.RESP_PARAM_TEXT);

  // Do whatever you want with the documents content
} else {
  // Something goes wrong and command doesn't return last number.
}

```

This is little tricky. To retrieve last document from journal first you must get
last document number. Once you have it you can print it or get it.


### Print journal document

> Print document from journal

```java
...
JournalControl control = (JournalControl) fiscalDevice;

control.printJournalDocument(last);
```

Printing journal document is very simple if you know document number you want to print.

<aside class="warning">On some devices print journal document isn't supported, but you can check it using device capabilities.</aside>


## Service

| Command | Description
| ------- | -----------
| `serviceCashIn()`               | Service cash in
| `serviceCashOut()`              | Service cash out
| `feedPaper()`                   | Feed paper given lines
| `beep()`                        | Make beep sound signal
| `printDiagnosticInfo()`         | Prints out diagnostic information about printer
| `getLastTransactionStatus()`    | Check if there's open fiscal receipt and get some details on it, like amount not paid
| `getLastFiscalReceiptDateTime()`| Get last fiscal transaction date and time
| `setOperatorName()`             | Set new operator's name
| `setOperatorPassword()`         | Change operator's password
| `getRemainingFreeFiscalEntries()`| Check how many free entries are in the fiscal memory
| `checkAndResolve()`             | Reset device to normal working state (if possible)

### Date and time

Date and time can be set and get via `setDateTime()` and `getDateTime()`

<aside class="warning">Date and time format is: dd-MM-yy hh:mm:ss</aside>

## Custom command

For all commands that aren't in SDK but supported by printer, there is `customCommand()`.

`customCommand()` gets two arguments: `number` of command and command `parameters` as string, if any.
Empty string is allowed if there's no parameters. Custom command return String as result. The result
holds all response data in some format. This format is different or have small
differences for all devices and commands.

<aside class="warning">Before using custom command make sure you have understand format of input and output parameters for selected command on device you work with</aside>

###
