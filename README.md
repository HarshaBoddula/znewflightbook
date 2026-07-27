# znewflightbook
Flight Booking Analytical List Page Fiori Elements App

**Output of the App:**

<img width="1854" height="967" alt="image" src="https://github.com/user-attachments/assets/92f8b54c-3b8e-4f80-be1f-fb4abda4b442" />

z_demo_flight_cds/README.md at main · GonzaloMB/z_demo_flight_cds · GitHub

Visual Filters in Analytical List Page | SAP Fiori Elements | Annotations & ABAP CDS


Back-end 🔩
1. ABAP CDS
	• DIMENSION: Airline
@AbapCatalog.sqlViewName: 'ZDIMEAIRLINE'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Airline'
@Analytics.dataCategory: #DIMENSION
define view Z_Dimension_Airline
  as select from scarr
{
      @ObjectModel.text.element: [ 'AirlineName' ]
  key carrid   as Airline,
  
      @Semantics.text: true
      carrname as AirlineName,
      
      @Semantics.currencyCode: true
      currcode as Currency
} 
	• DIMENSION: Connection
@AbapCatalog.sqlViewName: 'ZDIMECONNECT'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Connection'
@Analytics.dataCategory: #DIMENSION
@ObjectModel.representativeKey: 'FlightConnection'
define view Z_Dimension_Connection
  as select from spfli
  association [0..1] to Z_Dimension_Airline as _Airline on $projection.Airline = _Airline.Airline
{
      @ObjectModel.foreignKey.association: '_Airline'
  key carrid                    as Airline,
@ObjectModel.text.element: [ 'Destination' ]
  key connid                    as FlightConnection,
@Semantics.text: true
      concat(cityfrom,
        concat(' -> ', cityto)) as Destination,
_Airline
} 
	• DIMENSION: Customer
@AbapCatalog.sqlViewName: 'ZDIMECUSTOMER'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Customer'
@Analytics.dataCategory: #DIMENSION
define view Z_Dimension_Customer
  as select from scustom
  association [0..1] to I_Country as _Country on $projection.Country = _Country.Country
{
      @ObjectModel.text.element: [ 'CustomerName' ]
  key id      as Customer,
@Semantics.text: true
      name    as CustomerName,
@ObjectModel.foreignKey.association: '_Country'
      @Semantics.address.country: true
      country as Country,
@Semantics.address.city: true
      city    as City,
      
      _Country
} 
	• DIMENSION: Travel Agency
@AbapCatalog.sqlViewName: 'ZDIMETRVAGENCY'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Travel Agency'
@Analytics.dataCategory: #DIMENSION
define view Z_Dimension_TravelAgency
  as select from stravelag
  association [0..1] to I_Country as _Country on $projection.Country = _Country.Country
{
      @ObjectModel.text.element: [ 'TravelAgencyName' ]
  key agencynum as TravelAgency,
@Semantics.text: true
      name      as TravelAgencyName,
@ObjectModel.foreignKey.association: '_Country'
      @Semantics.address.country: true
      country   as Country,
@Semantics.address.city: true
      city      as City,
      
      _Country
} 
	• CUBE: Flight Bookings
@AbapCatalog.sqlViewName: 'ZCUBEFLIGHTBOOK'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Bookings'
@Analytics.dataCategory: #CUBE
define view Z_Cube_FlightBookings
  as select from sbook
  association [0..1] to I_CalendarDate           as _CalendarDate on  $projection.FlightDate = _CalendarDate.CalendarDate
  association [0..1] to Z_Dimension_Airline      as _Airline      on  $projection.Airline = _Airline.Airline
  association [0..1] to Z_Dimension_Connection   as _Connection   on  $projection.Airline          = _Connection.Airline
                                                                  and $projection.FlightConnection = _Connection.FlightConnection
  association [0..1] to Z_Dimension_Customer     as _Customer     on  $projection.Customer = _Customer.Customer
  association [0..1] to Z_Dimension_TravelAgency as _TravelAgency on  $projection.TravelAgency = _TravelAgency.TravelAgency
{
  /** DIMENSIONS **/
@EndUserText.label: 'Airline'
  @ObjectModel.foreignKey.association: '_Airline'
  carrid                 as Airline,
@EndUserText.label: 'Connection'
  @ObjectModel.foreignKey.association: '_Connection'
  connid                 as FlightConnection,
@EndUserText.label: 'Flight Date'
  @ObjectModel.foreignKey.association: '_CalendarDate'
  fldate                 as FlightDate,
@EndUserText.label: 'Book No.'
  bookid                 as BookNumber,
@EndUserText.label: 'Customer'
  @ObjectModel.foreignKey.association: '_Customer'
  customid               as Customer,
@EndUserText.label: 'Travel Agency'
  @ObjectModel.foreignKey.association: '_TravelAgency'
  agencynum              as TravelAgency,
@EndUserText.label: 'Flight Year'
  _CalendarDate.CalendarYear,
@EndUserText.label: 'Flight Month'
  _CalendarDate.CalendarMonth,
@EndUserText.label: 'Customer Country'
  @ObjectModel.foreignKey.association: '_CustomerCountry'
  _Customer.Country      as CustomerCountry,
@EndUserText.label: 'Customer City'
  _Customer.City         as CustomerCity,
@EndUserText.label: 'Travel Agency Country'
  @ObjectModel.foreignKey.association: '_TravelAgencyCountry'
  _TravelAgency.Country  as TravelAgencyCountry,
@EndUserText.label: 'Travel Agency Customer City'
  _TravelAgency.City     as TravelAgencyCity,
/** MEASURES **/
@EndUserText.label: 'Total of Bookings'
  @DefaultAggregation: #SUM
  1                      as TotalOfBookings,
@EndUserText.label: 'Weight of Luggage'
  @DefaultAggregation: #SUM
  @Semantics.quantity.unitOfMeasure: 'WeightUOM'
  luggweight             as WeightOfLuggage,
@EndUserText.label: 'Weight Unit'
  @Semantics.unitOfMeasure: true
  wunit                  as WeightUOM,
@EndUserText.label: 'Booking Price'
  @DefaultAggregation: #SUM
  @Semantics.amount.currencyCode: 'Currency'
  forcuram               as BookingPrice,
@EndUserText.label: 'Currency'
  @Semantics.currencyCode: true
  forcurkey              as Currency,
// Associations
  _Airline,
  _CalendarDate,
  _CalendarDate._CalendarMonth,
  _CalendarDate._CalendarYear,
  _Connection,
  _Customer,
  _Customer._Country     as _CustomerCountry,
  _TravelAgency,
  _TravelAgency._Country as _TravelAgencyCountry
} 
	• QUERY: Flight Bookings
@AbapCatalog.sqlViewName: 'ZQUERYFLIGHTBOOK'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Bookings'
@Analytics.query: true
@VDM.viewType: #CONSUMPTION
define view Z_Query_FlightBookings
  as select from Z_Cube_FlightBookings
{
    /** DIMENSIONS **/
    
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    Airline, 
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    FlightConnection, 
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    FlightDate, 
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    Customer, 
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    TravelAgency, 
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    CalendarYear,
    @AnalyticsDetails.query.display: #TEXT
    @AnalyticsDetails.query.axis: #FREE
    CalendarMonth,
    @AnalyticsDetails.query.display: #TEXT
    @AnalyticsDetails.query.axis: #FREE
    CustomerCountry,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    CustomerCity,
    @AnalyticsDetails.query.display: #TEXT
    @AnalyticsDetails.query.axis: #FREE
    TravelAgencyCountry,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    TravelAgencyCity,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    Currency,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    WeightUOM,
    
    /** MEASURES **/
    
    TotalOfBookings, 
    WeightOfLuggage,
    BookingPrice,
    
    @EndUserText.label: 'Average Weight Per Flight'
    @AnalyticsDetails.exceptionAggregationSteps.exceptionAggregationBehavior: #AVG
    @AnalyticsDetails.exceptionAggregationSteps.exceptionAggregationElements: [ 'Airline', 'FlightConnection', 'FlightDate' ]
    @AnalyticsDetails.query.formula: '$projection.WeightOfLuggage'
    @AnalyticsDetails.query.decimals: 0
    0 as AverageWeightPerFlight
} 
Front-End ⌨️

From <https://github.com/GonzaloMB/z_demo_flight_cds/blob/main/README.md> 

Back-end 🔩
1. ABAP CDS
	• DIMENSION: Airline
	@AbapCatalog.compiler.compareFilter: true
	@AccessControl.authorizationCheck: #CHECK
	@EndUserText.label: 'Airline'
	
	@Analytics.dataCategory: #DIMENSION
	
	define view entity Z_Dimension_Airline
	  as select from /dmo/carrier
	{
	      @ObjectModel.text.element: [ 'AirlineName' ]
	  key carrier_id   as Airline,
	  
	      @Semantics.text: true
	      name as AirlineName,
	      
	      @Semantics.currencyCode: true
	      currency_code as Currency
	} 
	• DIMENSION: Connection
@AbapCatalog.sqlViewName: 'ZDIMECONNECT'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Connection'
@Analytics.dataCategory: #DIMENSION
@ObjectModel.representativeKey: 'FlightConnection'
define view Z_Dimension_Connection
  as select from /dmo/connection
  association [0..1] to Z_Dimension_Airline as _Airline on $projection.Airline = _Airline.Airline
{
      @ObjectModel.foreignKey.association: '_Airline'
  key carrier_id                    as Airline,
@ObjectModel.text.element: [ 'Destination' ]
  key connection_id                    as FlightConnection,
@Semantics.text: true
      concat(airport_from_id,
        concat(' -> ', airport_to_id)) as Destination,
_Airline
} 
	• DIMENSION: Customer
@AbapCatalog.sqlViewName: 'ZDIMECUSTOMER'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Customer'
@Analytics.dataCategory: #DIMENSION
define view Z_Dimension_Customer
  as select from/dmo/customer
  association [0..1] to I_Country as _Country on $projection.Country = _Country.Country
{
      @ObjectModel.text.element: [ 'CustomerName' ]
   customer_id      as Customer,
@Semantics.text: true
      first_name    as CustomerName,
@ObjectModel.foreignKey.association: '_Country'
      @Semantics.address.country: true
      country_code as Country,
@Semantics.address.city: true
      city    as City,
      
      _Country
} 
	• DIMENSION: Travel Agency
@AbapCatalog.sqlViewName: 'ZDIMETRVAGENCY'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Travel Agency'
@Analytics.dataCategory: #DIMENSION
define view Z_Dimension_TravelAgency
  as select from /dmo/agency
  association [0..1] to I_Country as _Country on $projection.Country = _Country.Country
{
      @ObjectModel.text.element: [ 'TravelAgencyName' ]
  key agency_id as TravelAgency,
@Semantics.text: true
      name      as TravelAgencyName,
@ObjectModel.foreignKey.association: '_Country'
      @Semantics.address.country: true
      country_code   as Country,
@Semantics.address.city: true
      city      as City,
      
      _Country
} 
	• CUBE: Flight Bookings
	
	@AbapCatalog.viewEnhancementCategory: [#NONE]
	@AccessControl.authorizationCheck: #NOT_REQUIRED
	@EndUserText.label: 'Flight Bookings'
	@Analytics.dataCategory: #CUBE
	define view entity Z_Cube_FlightBookings
	as select from /dmo/booking
	inner join   /dmo/travel on /dmo/travel.travel_id = /dmo/booking.travel_id
	association [0..1] to I_CalendarDate           as _CalendarDate on  $projection.FlightDate = _CalendarDate.CalendarDate
	association [0..1] to Z_Dimension_Airline      as _Airline      on  $projection.Airline = _Airline.Airline
	association [0..1] to Z_Dimension_Connection   as _Connection   on  $projection.Airline          = _Connection.Airline
	and $projection.FlightConnection = _Connection.FlightConnection
	association [0..1] to Z_Dimension_Customer     as _Customer     on  $projection.Customer = _Customer.Customer
	association [0..1] to Z_Dimension_TravelAgency as _TravelAgency on  $projection.TravelAgency = _TravelAgency.TravelAgency
	{
	/** DIMENSIONS **/
	@EndUserText.label: 'Airline'
	@ObjectModel.foreignKey.association: '_Airline'
	/dmo/booking.carrier_id    as Airline,
	@EndUserText.label: 'Connection'
	@ObjectModel.foreignKey.association: '_Connection'
	/dmo/booking.connection_id as FlightConnection,
	@EndUserText.label: 'Flight Date'
	@ObjectModel.foreignKey.association: '_CalendarDate'
	/dmo/booking.flight_date   as FlightDate,
	@EndUserText.label: 'Book No.'
	/dmo/booking.booking_id    as BookNumber,
	@EndUserText.label: 'Customer'
	@ObjectModel.foreignKey.association: '_Customer'
	/dmo/booking.customer_id   as Customer,
	@EndUserText.label: 'Travel Agency'
	@ObjectModel.foreignKey.association: '_TravelAgency'
	/dmo/travel.agency_id      as TravelAgency,
	@EndUserText.label: 'Flight Year'
	_CalendarDate.CalendarYear,
	@EndUserText.label: 'Flight Month'
	_CalendarDate.CalendarMonth,
	@EndUserText.label: 'Customer Country'
	@ObjectModel.foreignKey.association: '_CustomerCountry'
	_Customer.Country          as CustomerCountry,
	@EndUserText.label: 'Customer City'
	_Customer.City             as CustomerCity,
	@EndUserText.label: 'Travel Agency Country'
	@ObjectModel.foreignKey.association: '_TravelAgencyCountry'
	_TravelAgency.Country      as TravelAgencyCountry,
	@EndUserText.label: 'Travel Agency Customer City'
	_TravelAgency.City         as TravelAgencyCity,
	/** MEASURES **/
	@EndUserText.label: 'Total of Bookings'
	@DefaultAggregation: #SUM
	1                          as TotalOfBookings,
	//@EndUserText.label: 'Weight of Luggage'
	//  @DefaultAggregation: #SUM
	//  @Semantics.quantity.unitOfMeasure: 'WeightUOM'
	//  luggweight             as WeightOfLuggage,
	//@EndUserText.label: 'Weight Unit'
	//  @Semantics.unitOfMeasure: true
	//  wunit                  as WeightUOM,
	@EndUserText.label: 'Booking Price'
	@DefaultAggregation: #SUM
	@Semantics.amount.currencyCode: 'Currency'
	/dmo/booking.flight_price  as BookingPrice,
	@EndUserText.label: 'Currency'
	//  @Semantics.currencyCode: true
	/dmo/booking.currency_code as Currency,
	// Associations
	_Airline,
	_CalendarDate,
	_CalendarDate._CalendarMonth,
	_CalendarDate._CalendarYear,
	_Connection,
	_Customer,
	_Customer._Country         as _CustomerCountry,
	_TravelAgency,
	_TravelAgency._Country     as _TravelAgencyCountry
	}
	• QUERY: Flight Bookings
@AbapCatalog.sqlViewName: 'ZQUERYFLIGHTBOOK'
@AbapCatalog.compiler.compareFilter: true
@AccessControl.authorizationCheck: #CHECK
@EndUserText.label: 'Flight Bookings'
@Analytics.query: true
@VDM.viewType: #CONSUMPTION
define view Z_Query_FlightBookings
  as select from Z_Cube_FlightBookings
{
    /** DIMENSIONS **/
    
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    Airline, 
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    FlightConnection, 
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    FlightDate, 
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    Customer, 
    @AnalyticsDetails.query.display: #KEY_TEXT
    @AnalyticsDetails.query.axis: #FREE
    TravelAgency, 
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    CalendarYear,
    @AnalyticsDetails.query.display: #TEXT
    @AnalyticsDetails.query.axis: #FREE
    CalendarMonth,
    @AnalyticsDetails.query.display: #TEXT
    @AnalyticsDetails.query.axis: #FREE
    CustomerCountry,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    CustomerCity,
    @AnalyticsDetails.query.display: #TEXT
    @AnalyticsDetails.query.axis: #FREE
    TravelAgencyCountry,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    TravelAgencyCity,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    Currency,
    @AnalyticsDetails.query.display: #KEY
    @AnalyticsDetails.query.axis: #FREE
    WeightUOM,
    
    /** MEASURES **/
    
    TotalOfBookings, 
    WeightOfLuggage,
    BookingPrice,
    
    @EndUserText.label: 'Average Weight Per Flight'
    @AnalyticsDetails.exceptionAggregationSteps.exceptionAggregationBehavior: #AVG
    @AnalyticsDetails.exceptionAggregationSteps.exceptionAggregationElements: [ 'Airline', 'FlightConnection', 'FlightDate' ]
    @AnalyticsDetails.query.formula: '$projection.WeightOfLuggage'
    @AnalyticsDetails.query.decimals: 0
    0 as AverageWeightPerFlight
} 
Front-End ⌨️

From <https://github.com/GonzaloMB/z_demo_flight_cds/blob/main/README.md> 

