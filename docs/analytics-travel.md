## Category - Travel

### Search Event
Search event should be sent from the screen where user searches for flight/bus/hotel.
```js
Search(fromLocation: string, toLocation: string[], date: Date[], roundTrip: boolean, travellerCount: number)
```

Create a search object by passing following arguments

1.	Source location: ```string```
	Source location will be city name for Bus, location of hotel in case of hotels and should be International airport code for flight category.

2.	Destination location: ```string[]```
	This array should contain destination location for flights/bus. Incase there are intermediate locations they should came in same array in order.
	This field can be empty for hotel booking.

3.	Date: ```Date[]```
	This will contain array of dates for multi city travel. For hotel booking this will be start and end date of hotel reservation.

4.	RoundTrip: ```boolean```
	True if it is a round trip

5. Traveller Count: ```number``` Number of travellers. Must be greater than zero.

**Sample**
##### For react native
```js
const search = new Travel.Search("BLR", ["DEL"], [new Date()], false, 1);
```
##### For webview
```js
const search = FKEvents.Travel.Search("BLR", ["DEL"], [new  Date()], false, 1);
```


### Select Event
Select event should be sent when user clicks and selects any flight/bus/hotel from the list.
```js
Select(name: string, category: string, price: number)
```
Following are the arguments for Select

1.	Name: ```string```
	This will be flight name/hotel name or name of Bus operator

2.	Category: ```string```
	It should have one of the following values,
	FLIGHT_INTERNATIONAL, FLIGHT_DOMESTIC, HOTEL_INTERNATIONAL, 	HOTEL_DOMESTIC, BUS
	
3.	Price: ```number```
	Price of the product selected in Rupees.

**Sample**
```js
const select = new Travel.Select(Travel.Select("6E135","FLIGHT_DOMESTIC",3500);
```

### ProceedToPay Event

Proceed to pay event should be sent when users completes the selection and is proceeding to the payment screen
```js
ProceedToPay(amountToPay: number, offerAmount: number, bookingCharge: number, vasCharge: number)
```

Following are the arguments for Proceed to pay event. All the values below are in Rupees and accepts decimal.

1.	Cost of Product: ```number```
	Price of travel ticket in Rupees.

2.	(Optional)Offer Amount: ```number```
	Total offer amount given to the customer.

3.	(Optional)Booking Charge: ```number```
	Amount charged as conveniance fees.

4.	(Optional)Value added service charge: ```number```
	Total value added service charge for the customer.

**Sample**
```js
const pay = new Travel.ProceedToPay(3760, 0, 350, 0);
```
