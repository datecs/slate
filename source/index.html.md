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
With variety of devices and their functionality, there can be found minor protocol differences
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

This is the minimum requirement for compiling device through the processor.
In this case processor will generates device which supports nothing, cool ah?

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

Our first test device supports electronic journal and external display, so we need to
implements `JournalControl` and `DisplayControl`.

<aside class="success">Processor automatically populate devices capabilities by checking all implemented interfaces. </aside>

Notice that **not annotated** methods and commands are not processing by the processor.

## Fill the properties

```java
  @Device(
          programmableItems = 12000,
          displayTextLength = 20,
          printTextLength = 36,
          vatRates = 8,
          customPayments = 0
  )
  @Model("DP15")
  @Variant("KL")
  @Localization(Country.BULGARIA)
  @TransportProtocol(TransportProtocolV1.class)
  class DP15 extends FMP10 implements DisplayControl, JournalControl {
    ...
  }
```

Device properties must be described in `@Device` annotation. There are some required and some
optional properties to be filled.

# Add an new property

The only thing you need is to add the desired property in `@Deivce` annotation class.
After that depending on that if the property is optional or required you must give
values to all supported devices.

# Add an new capability

> Example of capability

```java
public boolean capDisplay() { return false; }
```

Adding new capability requires adding method starting `cap` in `FiscalDeviceInfo` class.
For example: `boolean capDisplay() { return false; }`

## Resolving

At device compilation, processor will try to match this method with one of all implemented
or copied methods or by getting all implemented interfaces. If processor cannot resolve
the right capability value, you must set it explicitly. By default all capabilities are `false`
and properties are with empty or 0 values.

![devices structure](../images/capabilities_mapping.jpg)

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

### Status reported by response

Some commands returns execution status or result in the response.
You can handle this with `@WhenFailed` or `@WhenSuccess` annotations.

`@WhenSuccess("0")` for example will validate that first argument of the response is 0.
If argument value is something different then exception will be thrown. There are
situations where the result is merged with the next argument for example `"P0001"`,
where `P` is the status and `0001` first usable argument. In this case we can use
`@WhenSuccess(value = "P", split = 1)` - this will tell the processor to split
the first argument and do the necessaries checks.

<aside class="notice">@WhenFailed works as same way as @WhenSuccess</aside>

## Custom validators and bindings

Custom validators and biding are needed in some cases. Cases where the input parameter must
be processed before it will be send to the command or device. Cases where you must get
value from somewhere else.

This processor supports 2 types of bindings (validators)

### Status bindings

> Binding status bits to error response for the command

```java
@Override
@CommandSpec("53")
@ArgumentsSpec({
       @Parameter("^t"),
       @Parameter(value = "Mode", resolve = @ResolveStrategy(value = GET_METHOD_PARAM, resolvedValue = "getPaidMode")),
       @Parameter(value = "Amount", resolve = @ResolveStrategy(GET_METHOD_PARAM))
})
@ResponseSpec({@Parameter("PaidCodeAmount")})
@CheckStatusSpec(value = {
  @StatusSpec(statusByte = 0, statusBit = 5),
  @StatusSpec(statusByte = 4, statusBit = 5)
}, resolveValue = "getErrorCodeFromStatus")
public FiscalResponse total(PaidMode mode, double amount) throws FiscalException, NotSupportedException, IOException {
   return super.total(mode, amount);
}
```

Some devices reports their status by low level status bytes and bits. Every status bit mean
something. In some cases reported statuses indicates error on command execution instead of
returning error code in the command response. Example for error status is when paper ends.

Using `@CheckStatusSpec` you can specify and bind error statuses to the command response, using
array of `@StatusSpec`.

In the example above we tell the processor that if **after** command execution bytes `0.5` and `4.5` are
on, this indicates error while executing the command. Passing method name to `resolveValue`
will map these statuses to error constants and messages which can be found in `FiscalException` and `FiscalErrorCodes`

### Method bindings (proxies)

Looking at `total` command above you can find line looking this:

` @Parameter(value = "Mode", resolve = @ResolveStrategy(value = GET_METHOD_PARAM, resolvedValue = "getPaidMode")),`

This line must tell you something like this:

*"There is parameter named Mode, which must be resolved by getting
Mode method parameter and resolve the real value by putting current value
as argument to getPaidMode and return me the right value."*

Another example is when binding statuses. Look again at the `total` command above.
Reading `@CheckStatusSpec` must tell you something right this:

*"Processor must check statuses 0.5 and 4.5 after execution of this command.
Resolve the errors using getErrorCodeFromStatus method and."*

In most cases proxies methods input in single string and the output is whatever
you need - `boolean`, `Enum`, `String` etc...

## Custom methods (where there's no other way)

In some cases the annotations DSL is not enough. Less effort is required if
you hand write the method, but how can this be copied into generated file.
There's way to tell the processor *Copy this method for me* using `@Copy` annotation.

On copy processor will figure if the method override or not and will put `@Override`
if needed!

# Testing

This project uses [Cucumber](https://cucumber.io/) [BDD](https://www.google.bg/#q=BDD)
testing suite because we are testing both SDK and device behavior.
Cucumber uses Gherkins - Business Readable, Domain Specific Language.
All test case scenarios are located in `/devices/src/test/resources`
and the scenarios code-specific implementations can be found under
`/devices/src/test/java` directory.

![testing structure](../images/cucumber.jpg)

## DI

As you can see all BDD tests are separated in `feature` files in different categories.
All implementation code-specific steps are located in `StepDefs` java files.
Because of this separation we use dependency injection to pass different objects
between the `StepDefs` files. Example for passing object is the `FiscalDevice`
objects with communicates with the connected device. DI helps us pass single
instance on multiple java classes.

Don't worry we provide handy way to create easily tests.

## Writing tests

Writing test contains 3 steps.

0. Create `feature` file and write scenarios
0. Write `StepDefs` for the scenarios.
0. Make some preprocess checks, because some devices MUST fail on particular tests
and some devices MUST NOT fail.

This looks something like this and it's part of the features and step definitions:

> In the feature

```Gherkins
Background: To test journal we need normal functionality device supporting journal control checking capability
    Given fiscal device in normal state
    And device supports journal
```

> And the step definition

```java

// In CommonStepDefs.java

@Given("^fiscal device in normal state$")
public void fiscalDeviceInNormalState() throws Throwable {
   assertNotNull("Device cannot be null", device());
   device().checkAndResolve();
}

 // In JournalStepDefs.java

 @And("^device supports journal$")
public void deviceSupportsJournal() throws Throwable {
    assertTrue(device().getInfo().capJournal());
}
```

<aside class="success">Note that feature can combine step definitions
implemented in different java classes.</aside>

### Create feature

We follow the official cucumber guide.

> Example feature for reports

```Gherkins
Feature: Reports
  This feature shows and check execution of several command which prints reports.
  Reports can be by date and by number. There are also X and Z daily reports.

  Background: To test reports we need normal functionality device
    Given fiscal device in normal state

  Scenario: Print X report
    When device supports X report
    And initialize X report
    Then command must execute normally

  Scenario: Print Z report
    When device supports Z report
    And initialize Z report
    Then command must execute normally
```
After we are done defining scenarios it's time to implement the step definitions.

### Implement step definitions

> Reports step definitions

```java
public class ReportsStepDefs extends StepDefsWrapper {
  public ReportsStepDefs(StepDefsBase base) {
      super(base);
  }

  @When("^initialize X report$")
  public void initializeXReport() throws Throwable {
      try {
          device().printXReport();
      } catch (Exception e) {
          exception(e);
      }
  }

  @When("^initialize Z report$")
  public void initializeZReport() throws Throwable {
      try {
          device().printZReport();
      } catch (Exception e) {
          exception(e);
      }
  }

  @When("^device supports X report$")
  public void deviceSupportsXReport() throws Throwable {
      assertTrue(device().getInfo().capPrintXReport());
  }

  @When("^device supports Z report$")
  public void deviceSupportsZReport() throws Throwable {
      assertTrue(device().getInfo().capPrintZReport());
  }
}
```

And here is the tricky part!

### Cucumber wrappers

![cucumber wrapper](../images/cucumber_wrapper.jpg)

All step definitions java classes must extends our wrapper/helper class named `StepDefsWrapper`
which provides access to the connected device, setters and getters of the response
and more.

Setters and getters to the response in needed in case you want to check response
and/or exception after command execution.

<aside class="warning">Because cucumber-jvm does not support junit rules, we try-catch
and save exceptions. Then check them.</aside>

<aside class="note">StepDefsWrapper holds instance of StepDefBase which is injection using
pico container.</aside>
