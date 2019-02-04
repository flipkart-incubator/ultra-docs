## Category - Travel

### Search Event
Search event should be sent from the screen where user searches for flight/bus/hotel.
```js
Search(fromLocation: string, toLocation: string[], date: Date[], roundTrip: boolean, travellerCount: number)
```

Create a search object by passing following arguments

1.	**fromLocation**: ```string```
	Source location will be city name for Bus, location of hotel in case of hotels and should be International airport code for flight category.

2.	**toLocation**: ```string[]```
	This array should contain destination location for flights/bus. Incase there are intermediate locations they should came in same array in order.
	This field can be empty for hotel booking.

3.	**date**: ```Date[]```
	This will contain array of dates for multi city travel. For hotel booking this will be start and end date of hotel reservation.

4.	**roundTrip**: ```boolean```
	True if it is a round trip

5.  **travellerCount**: ```number``` Number of travellers. Must be greater than zero.

**Sample**
##### For react native
```js
const search = new Travel.Search("BLR", ["DEL"], [new Date()], false, 1);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(search);
```
##### For webview
```js
const search = FKExtension.analyticsHelper.travel.search("BLR", ["DEL"], [new  Date()], false, 1);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(search);
```


### Select Event
Select event should be sent when user clicks and selects any flight/bus/hotel from the list.
```js
Select(name: string, category: string, price: number)
```
Following are the arguments for Select

1.	**name**: ```string```
	This will be flight name/hotel name or name of Bus operator

2.	**category**: ```string```
	It should have one of the following values,
	FLIGHT_INTERNATIONAL, FLIGHT_DOMESTIC, HOTEL_INTERNATIONAL, 	HOTEL_DOMESTIC, BUS
	
3.	**price**: ```number```
	Price of the product selected in Rupees. Price must not be negative.

**Sample**
##### For react native
```js
const select = new Travel.Select("6E135","FLIGHT_DOMESTIC",3500);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(select);
```
##### For webview
```js
const select = FKExtension.analyticsHelper.travel.select("6E135","FLIGHT_DOMESTIC",3500);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(select);

```

### ProceedToPay Event

Proceed to pay event should be sent when users completes the selection and is proceeding to the payment screen
```js
ProceedToPay(amountToPay: number, offerAmount: number, bookingCharge: number, vasCharge: number)
```

Following are the arguments for Proceed to pay event. All the values below are in Rupees and accepts decimal.

1.	**amountToPay**: ```number```
	Cost of Travel Product in Rupees. This should not be negative.

2.	(Optional) **offerAmount**: ```number```
	Total offer amount given to the customer. The value should not be negative.

3.	(Optional) **bookingCharge**: ```number```
	Amount charged as conveniance fees. The value should not be negative.

4.	(Optional) **vasCharge**: ```number```
	Total value added service charge for the customer. Value should not be negative.

**Sample**
##### For react native
```js
const pay = new Travel.ProceedToPay(3500, 0, 350, 0);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(pay);
```
##### For webview
```js
const pay = FKExtension.analyticsHelper.travel.proceedToPayEvent(3500, 0, 350, 0);
fkPlatform.getModuleHelper().getAnalyticsModule().pushEvent(pay);
```
