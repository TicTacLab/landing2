---
title: MathEngine. File preparation.
---

## MathEngine

# Excel file preparation

In order to upload an Excel file to the Engine you MUST create IN and OUT sheets. These sheets are used for communication between Engine and your file.

<div class="well well-sm">
<b>Note!</b> All used cells in the "IN" and "OUT" sheets MUST have values. Cells that are not used MUST be cleared. You can clear cells by clicking Home Tab -> Editing Section -> Clear -> Clear All (Alt+H+E+A).
</div>

## "IN" sheet preparation

*IN* sheet is the entry point of your file for the Engine. This sheet contains all input parameters that your file accepts for calculation results and that your file depends on.

Typical *IN* parameters are:

* Match scores
* Match time
* Current game part
* Player's superiority
* Scores expectancy
* etc

This sheet MUST contain at least two *required* columns: **id** and **value**.

* **id** column MUST contain a unique parameter identifier. This parameter MUST be a number. 
Usually this value is the serial number of the parameter.
* **value** column is an appropriate value of each parameter. There are no restrictions on column values types.

### "IN" sheet example

![IN](/images/in-page.png)

## "OUT" sheet preparation

The "OUT" sheet contains the Excel file calculation result.
The content of this sheet will be returned by [MathEngine REST API](/en/doc/mengine/integration-guide/) as is.

**"OUT" sheet references MUST point <u>FROM</u> all data that you want to get from the file.**

Typical *OUT* values are:

* Odds
* Markets parameters
* Outcome parameters
* etc.

### "OUT" sheet example

![OUT](/images/out-page.png)

## Next steps

Now you can generate services using [MathEngine Administration Interface](/en/doc/user-guide/).

