---
title: Fiscal Processor Guide

language_tabs:
  - java

toc_footers:
  - <a href='http://datecs.bg/'>Datecs Ltd.</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - changelog

search: true
---

# Introduction

Datecs Fiscal devices works on custom internal synchronous command-response protocol.
With variety of devices and their functionality, where are minor protocol differences
on all of the devices. To rapid adding devices to `Fiscal Framework` we decide to use
`Java annotations` and `Java annotations processor` to describe devices and generate
implementations to work with the fiscal protocol.

This guide shows how newly device can be describe and add support to the framework.

# Overview

Before annotating devices, we must know about framework structure. Datecs fiscal framework
is modular. Each new device extends base `FiscalDevice` and implements all common fiscal
methods from `FiscalProtocol`. In additional there are peripheral controls, device capabilities
and properties and also extensions.

## FiscalDevice

`FiscalDevice` is base class which implements all `FiscalProtocol` methods. These
methods throws `NotSupportedException` by default. This class also provides constructors
methods for initializing device using streams or using `AbstractTransportProtocol`
which holds connection and communication logic.

<aside class="notice">From FiscalDevice implements ONLY supported by device methods.
All other methods will throw not supported exception by default. </aside>

## Controls

Controls are interfaces which extends base functionality providing methods to work
with other external peripherals like `Drawer`, `Barcode`, `Paper cutter` etc...
Device class must implements all supported by device itself external peripherals and
implementations must be provided.

All controls has capability constants. Developers using `Fiscal Framework` can check
if connected device which they work with supports given controls by checking these
capabilities.

## Extensions

Extensions are mostly Country-based, because each country have some specials regarding
working with fiscal devices methods and some of them are mandatory, but only for specific
country. In this case country extensions holds all these methods. Also provides custom
properties and commands parameters.

## Properties

Properties are printer constants which indicates printer's abilities. Example for
properties are number of custom payments, number of VAT rates, header and footer lines,
max length of fiscal or non-fiscal text and many more...

# Add an device

![devices structure](../images/devices_structure.jpg)

In devices module create new class in country package. `bgr` is for Bulgaria.
Let's take for example `TestDevice.java` supporting Bulgaria.

The java class path in this case must be: `src/main/java/com/datecs/fiscal/annotated/bgr/TestDevice.java`

Now open this class.

<aside class="warning">Make sure newly created class extends FiscalDevice </aside>

Annotated it with following annotations:
> Initializing

```java
@Device
@Model("TEST-1")
@Variant("ALPHA")
@Localization(Country.BULGARIA)
@TransportProtocol(TransportProtocolV2.class)
public class TestDevice extends FiscalDevice {

  public TestDevice(InputStream input, OutputStream output, int encoding) {
    super(input, output, encoding);
  }

  ...
}
```

- **@Device** - which tells processor this is device to processor
- **@Model** - specify the device Model
- **@Variant** - specify the device Variant
- **@Localization** - specify device's Localization. All are located in `Country` enum.
- **@TransportProtocol** - specify class for communication. In this case we will use `TransportProtocolV2`

## Extend all needed modules

> Implements needed controls

```java
@Device
@Model("TEST-1")
@Variant("ALPHA")
@Localization(Country.BULGARIA)
@TransportProtocol(TransportProtocolV2.class)
public class TestDevice extends FiscalDevice implements JournalControl, DisplayControl {

  public TestDevice(InputStream input, OutputStream output, int encoding) {
    super(input, output, encoding);
  }

  @Override
  FiscalResponse getJournalDocument(int document)  {
    throw new NotSupportedException();
  }

  @Override
  FiscalResponse getJournalDocuments(String startDate, String endDate, JournalRecord type) {
    throw new NotSupportedException();
  }

  @Override
  FiscalResponse printJournalDocument(int document)  {
    throw new NotSupportedException();
  }

  @Override
  FiscalResponse getJournalDocumentRange() {
    throw new NotSupportedException();
  }

  @Override
  public FiscalResponse clearDisplay() {
      throw new NotSupportedException();
  }

  @Override
  public FiscalResponse displayTextOnFirstLine(String line) {
      throw new NotSupportedException();
  }

  @Override
  public FiscalResponse displayTextOnSecondLine(String line) {
      return null;
  }

  @Override
  public FiscalResponse displayDateTime() {
      return null;
  }

  ...
}
```

Our first test device supports electronic journal and external display. We need to
implements `JournalControl` and `DisplayControl`.

<aside class="notice">Processor automatically populate devices capabilities by checking all implemented interfaces. </aside>

## Fill the properties

TODO


## Bind statuses (if needed)

# Add an new property
# Add an new capability

# Commands

> Annotate command

```java
@CommandSpec(value = "49", buildNumber = 2)
public FiscalResponse sell(String item, double price, TaxGroup tax, double quantity) {
  return null;
}
```

Describing command is 4 stages process.

0. Define command specification
0. Define command Arguments if there's any
0. Define command response if there's any
0. Define validators and checks

First annotated the method with `@CommandSpec` annotation. The first parameter of annotation
maps method with command from the protocol in this case `49` command. You also can specify build number
from which command is supported, by default is supported from any build numbers.

## Arguments

> Arguments on sell

```java
 @CommandSpec(value = "49", buildNumber = 2)
 @ArgumentsSpec({
   @Paramater(value = "item") // Maps this parameter with String item parameter of the command method
 })
 public FiscalResponse sell(String item, double price, TaxGroup tax, double quantity) {
   return null;
 }
```

Let's take for example `sell` command.

` public FiscalResponse sell(String item, double price, TaxGroup tax, double quantity)`

Sell command have required arguments. These arguments must be populated by annotating the method.
We must specify `@ArgumentsSpec` with list of `@Parameter` or just put `@WithoutArguments` if command has no arguments.
In cases where method have some required parameters `@Parameter` annotations must explicitly maps to the methods arguments, by name.

For example: `String item` parameter must be mapped with `@Parameter(value = "Item")` annotation with value name same as the parameter.

<aside class="warning">@Parameter annotations must match all required method params!</aside>
<aside class="notice">Use @WithoutArguments if command has not arguments.</aside>

### @Parameter annotation

Parameter annotation is most powerful and complex annotation in this processor.
This annotation can be mapped to method parameter or can be placeholder.

Available placeholders are:

- **^t** which put TAB
- **^n** which put new line

> Example placeholder

```java
@CommandSpec(value = "49", buildNumber = 2)
@ArgumentsSpec({
  @Paramater(value = "^t") // placeholder tab
})
public FiscalResponse sell(String item, double price, TaxGroup tax, double quantity) {
  return null;
}
```

Parameter annotation supports:

- prefixes (when there's value) `withPrefix`
- suffixes (when there's value) `withSuffix`
- prefix separator (whenever there's value or not) `prefixSep`
- suffix separator (whenever there's value or not) `suffixSep`
- surround by (which is prefix + same suffix) `surroundBy`
- default value `defaultValue`
- resolver strategy `resolve`

> Example annotated command

```java
 @Override
 @CommandSpec("49")
 @ArgumentsSpec({
   @Parameter(value = "Item", suffixSep = "\t",
              resolve = @ResolveStrategy(GET_METHOD_PARAM)),
   @Parameter(value = "Tax", suffixSep = "\t",
              resolve = @ResolveStrategy(value = GET_METHOD_PARAM,
                resolvedValue = "getTaxGroup")),
   @Parameter(value = "Price", suffixSep = "\t",
              resolve = @ResolveStrategy(GET_METHOD_PARAM)),
   @Parameter(value = "Quantity", suffixSep = "\t",
              resolve = @ResolveStrategy(GET_METHOD_PARAM)),
   @Parameter(value = "DiscountType", suffixSep = "\t", defaultValue = "0"),
   @Parameter(value = "DiscountValue", suffixSep = "\t", defaultValue = "0"),
   @Parameter(value = "Department", suffixSep = "\t",
              resolve = @ResolveStrategy(value = GET_GLOBAL_PARAM, defaultValue = "0"))
 })
 @ResponseSpec(value = {@Parameter(ERROR_CODE), @Parameter(SLIP_NUMBER)}, parseBy = "\t")
 @WhenSuccess("0")
 public FiscalResponse sell(String item, double price, TaxGroup tax, double quantity) {
     return super.sell(item, price, tax, quantity);
 }
```

### @ResolveStrategy annotation

Command parameters can be resolved in different ways:

0. **DO_NOT_RESOLVE** - Can be not resolved, so don't resolve it anyway
0. **GET_METHOD_PARAM** - Resolve by getting value passed by method parameter
0. **GET_GLOBAL_PARAM** - Resolve by getting some global parameter set before execution
0. **CHECK_METHOD_PARAM** - Resolve by checking some global parameters set before execution
0. **CHECK_GLOBAL_PARAM** - Resolve by checking value passed by method parameters
0. **FROM_METHOD** - Resolve by getting method value and pass it to `resolvedValue`

<aside class="warning">Make sure the name of methods and **resolveValue** strings are same when resolving using method notation.</aside>

## Response

> Example response in subtotal

```java
@Override
@CommandSpec("51")
@ArgumentsSpec({
  @Parameter(value = "Print", suffixSep = "\t",
              resolve = @ResolveStrategy(value = CHECK_GLOBAL_PARAM,
                defaultValue = "0", resolvedValue = "1")),
  @Parameter(value = "DiscountType", suffixSep = "\t", defaultValue = "0"),
  @Parameter(value = "DiscountValue", suffixSep = "\t", defaultValue = "0")
})
@ResponseSpec(value = {
  @Parameter(ERROR_CODE), @Parameter(SLIP_NUMBER), @Parameter(SUBTOTAL),
  @Parameter(TAX_A), @Parameter(TAX_B), @Parameter(TAX_C), @Parameter(TAX_D),
  @Parameter(TAX_E), @Parameter(TAX_F), @Parameter(TAX_G), @Parameter(TAX_H)}, parseBy = "\t")
@WhenSuccess("0")
public FiscalResponse subtotal() throws FiscalException, NotSupportedException, IOException {
    return super.subtotal();
}
```

Response if described in `@ResponseSpec` annotation with list of `@Parameters` again,
but at the end of the declaration you optionally can specify on which symbol to split the
raw response using `parseBy` value.

For example if command returned `param1,param2,param3,optional1,optinal2` we split this
string on `,` and populate the tokens in map. Value on `@Parameter` annotation
specify the map key for given token.

In case where command does not return any response or response must be ignored, use `@WithoutResponse` annotation instead.

## Custom validators and bindings
## Custom methods (where there's no other way)
