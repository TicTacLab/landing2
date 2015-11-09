---
title: MathEngine. File preparation.
---

## MathEngine

# Excel file preparation

In order to upload an Excel file to the engine you should create IN and OUT sheets. These sheets are needed for engine inter-operation with your file.

<div class="well well-sm">
<b>Note!</b> All used cells in the "IN" and "OUT" sheets must have values. Cells that are not used should be cleared. You can clear cells by clicking Home Tab -> Editing Section -> Clear -> Clear All (Alt+H+E+A).
</div>

## "IN" sheet preparation

*IN* sheet is the entry point of your file for the engine. This sheet should contain all input parameters that your model will accept for calculation.

**References from the "value" column from the "IN" sheet should point <u>TO</u> the input parameters of your Excel file.**

Typical *IN* parameters are:

* Match scores
* Match time
* Current game part
* Player's superiority
* Scores expectancy
* etc

This sheet should contain at least two *required* columns: **id** and **value**.

The **id** column should contain a unique parameter identifier. This parameter should be a number. 
Usually this value is the serial number of the parameter.

The **value** column is an appropriate value of each parameter. There are no restrictions on column values types.

### "IN" sheet example

![IN](/images/in-page.png)

## "OUT" sheet preparation

The "OUT" sheet contains the Excel file calculation result.
The content of this sheet will be returned by [MathEngine REST API](/en/doc/mengine/integration-guide/) as is.

**"OUT" sheet references should point <u>FROM</u> all data that you want to get from the file.**

Typical *OUT* values are:

* Coefficients
* Markets parameters
* Outcome parameters
* etc.

### "OUT" sheet example

![OUT](/images/out-page.png)

## Next steps

Now you can generate services using [MathEngine Administration Interface](/en/doc/user-guide/).

