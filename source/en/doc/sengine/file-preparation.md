---
title: Settlement Engine. Подготовка файла
no_translation: true
---

## Settlement Engine

# File preparation

## Facts about events in the match

*Settlement Engine* uses event log for settling markets when the match is going on.

Examples of "facts" about events:

* *Goal* was scored by *Team 1* in the *1st Half* at the *34 minute*.
* *Goal* was scored by *Team 2* in the *2st Half* at the *46 minute*.

This view of events in the log allows to settle outcome of any complexity you might have.

Firstly, you need to describe *Event Types* in the Excel sheet *EventType*.

## *EventType* sheet

This sheet must contain table of 3 columns:

* EventType - Type of Event (Goal, Red Card, Penalty, etc.)
* Attribute - Name of the event attribute (Team, Time, Position, etc.)
* Value     - Value event attribute can get (Team1, "21:33", etc.)

### Example of *EventType* sheet

![EventType](/images/event-type-sengine.png)

## *EventLog* sheet

First row of this sheet must contain only unique values from column *Attribute* from sheet *EventType*.

This sheet can be used for testing your algorithms for settling bets. You can fill rows in this sheet to emulate some game occasions.

After you upload file into *Settlement Engine* this sheet will be cleared.

![EventLog](/images/event-log-sengine.png)

## *MarketTemplates* sheet

This sheet is mandatory for validating Events that are passed into the system.

First column must contain values from column *Market_type_name* from *OUT* sheet. Other columns are filled out by names of Event Types from *EventType* sheet.

Cell values at the crossing are the *Attributes* that required for settling this bet.


### Example of *MarketTemplates* sheet

![EventLog](/images/market-templates-sengine.png)

## *OUT* sheet

This sheet must contain result of settling of outcomes. *Market_type_name* is required column.

## Example of *OUT* sheet

![EventLog](/images/out-sengine.png)

## Next steps
[BetEngines Administration Interface](/ru/doc/user-guide/)

<div class="well well-sm">
<b>Warning!</b> *OUT*, *MarketTemplates*, *EventLog*, *EventType* must not contain empty cells. All cells must have a value. To definitely clear unused cells use Clear -> Clear All button.
</div>
