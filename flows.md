---
layout: home 
sidebar:
  nav: sidemain
title: Sample Flows 
sort: 0
---


## Inventory Information Server (Parking Aggregator / Operator)
This is a system holding inventory information about connected parking locations.  
The information is stored in an APDS conformant __hierarchy__.

Example: an airport has two car parks, one capable of AVP and another one (regular car park with no additional services)

<img src="https://docs.ai-parking.cloud/assets/images/flows/place_hierarchy_1.png" width="80%">

Any system acting as a user/client towards the inventory server can read this hierarchy information and e.g. store it locally (and refresh it sporadically).  

# Use Case 1: Manual Parking
This is the basic use case where a traditional vehicle is being used. An overview of the identified use cases can be found [here](https://docs.ai-parking.cloud/use_cases). In this section, we'll look at the following steps of this use case:

* Before and while driving
  * Where is the parking facility?
  * How much does it cost?
  * Selection of a parking facility
  * Activation of in-vehicle navigation to entry
* Arriving/Entry
  * Open barrier
  * start parking transaction
* Where is the spot?
  * Occupancy information about area
  * Occupancy information about spot
* Navigation to spot
  * Guiding of customer within garage and 3D navigation map to free slot
* Park in/out
  * Manual park in/out
  * In slot or manual relocation for charging and other value-added services
* Charging
  * Manual plugin of cable
  * Manual selection of tariff
  * Manual start of charging service
* Navigation to exit
  * Guiding of customer within garage and 3D navigation map from slot to exit
* Exit
  * Open barrier
  * Stop parking transaction
* Payment
  * Charge parking fee to customer account
  * Charge charging fee to customer account


## Availability query (where is the parking facility, and how much does it cost?)
When the GET /places request is made, certain filter criteria can be specified in the query to narrow the search result down. 

### Example
The user backend serving a vehicle's user frontend wants to find parking options available at a specific destination (and in its vicinity):  

```bash
GET /places?expand=all&latitude=48.177624&longitude=11.556482&radius=1000
```

* the _expand_ parameter value of _all_ indicates that the response shall include all details about the matching places
* the _latitude_, _longitude_ and _radius_ parameters specify the destination and the acceptable radius around it

The inventory system will respond with something like this:

```json
{
  "meta": {
    "referenceInstant": 1741612330,
    "offset": 0,
    "pageSize": 100,
    "total": 1
  },
  "data": [
    {
      "id": "carPark1Id",
      "version": 1,
      "name": [
        {
          "language": "de",
          "string": "BMW Welt Parkhaus Nordwest"
        }
      ],
      "rightSpecifications": [
        { 
          "id": "publicParkingRightSpec",
          "version": 1
        }
      ],
      "indicativePointLocation": {
        "type": "Point",
        "coordinates": [
          11.552002,
          48.1858071
        ]
      },
      "description": [
        {
          "language": "en",
          "string": "The preferred parking location for visitors to the BMW Welt"
        }
      ],
      
      "areaType": "generalParking",
      "characteristics": {
        "accessControlled": true,
        "evChargingPoints": 15,
        "openToPublic": true,
        "spacesTotal": 552,
        "structureGrade": "underGround",
        "structureType": "offStreetStructure"
      },
      "contacts": [
        {
          "organisationName": [
            {
              "language": "de",
              "string": "BMW"
            }
          ],
          "type": "operator",
          "emails": [
            {
              "address": "customersupport@bmw.com",
              "typeCode": "customerService"
            }
          ]
        }
      ],
      "paymentMethods": [
        {
          "paymentMode": [
            "payAfterExit", "payAndExit"
          ]
        }
      ],
      "placeStreetAddress": {
        "postCode": "80809",
        "city": [
          {
            "language": "de",
            "string": "München"
          },
          {
            "language": "en",
            "string": "Munich"
          }
        ],
        "countryCode": "DE",
        "addressLines": [
          {
            "type": "street",
            "order": 0,
            "text": "Moosacher Straße 61"
          }
        ]
      }      
    }
  ]
}
```
The caller will then have to query the details of the referenced right specification. _Right Specifications_ provide further details on constraints and eligibility criteria that apply in a certain place.

```
GET /rights/specs/publicParkingRightSpec?expand=all
```

The inventory system will provide the requested details:  

```json
{
  "id": "publicParkingRightSpec",
  "version": 1,
  "rateEligibility": [
    {
      "eligibility": [
        {
          "qualifications": [
            {
              "withReservation": true,
            } 
          ]
        }
      ],
      "rateTable": {
        "id": "publicTariffId",
        "version": 1
      },
      "hierarchyElements": [
        {
          "id": "carPark1Id",
          "version": 1
        }
      ]
    }
  ],
  "validity": {
    "validityStatus": "definedByValidityTimeSpec",
    "validityTimeSpecification": {
      "overallStartTime": "2025-01-01T00:00:00Z"
    }
  },
  "transferable": false
}
```

Depending on the user interaction design of the user frontend, the system could now query tariff details to answer the "How much does it cost?" question (or do this later when requesting a quote in preparation of a reservation).  

```json
{
    "id": "publicTariffId",
    "version": 1,
    "rateTableName": [
        {
            "language": "en",
            "string": "Public Parking Tariff"
        }
    ],
    "rateLineCollections": [
        {
            "applicableCurrency": "EUR",
            "collectionSequence": 0,
            "maxTime": "PT12H",
            "rateLines": [
                {
                    "sequence": 0,
                    "description": [
                        {
                            "language": "en",
                            "string": "up to 1 hour"
                        }
                    ],
                    "rateLineType": "incrementingRate",
                    "value": 10.0,
                    "incrementPeriod": "PT1H"
                },
                {
                    "sequence": 1,
                    "description": [
                        {
                            "language": "en",
                            "string": "up to 2 hours"
                        }
                    ],
                    "rateLineType": "incrementingRate",
                    "value": 10.0,
                    "incrementPeriod": "PT1H"
                },
                {
                  "sequence": 2,
                  "description": [
                    {
                      "language": "en",
                      "string": "any following hour"
                    }
                  ],
                  "rateLineType": "incrementingRate",
                  "value": 4.0,
                  "incrementPeriod": "PT1H"
                }
            ],
            "relativeTimes": true
        }
    ],
    "availability": "public",
    "rateResponsibleParty": "BMW"
}
```

## Making a reservation
Once the driver has - based on the previously obtained inventory information - decided to use the car park at the destination, the user backend will have to  
* obtain a binding offer (APDS term: quote) and then
* confirm this (i.e. book it)

When the POST /quotes QuoteRightRequest is sent, the right specification to be applied shall be referenced. The list of available right specifications was retrieved earlier during the discovery process (in our first example, it was just one).

```
POST /quotes
```
_(the request body contains a so-called QuoteRightRequest)_
```json
{
  "id": "requestId",
  "version": 1,
  "periodStart": "2025-03-31T17:00:00Z",
  "periodEnd": "2025-04-02T10:00:00Z",
  "requestTime": "2025-03-10T18:32:12Z",
  "eligibility": {
    "qualifications": [
      {
        "vehicleCharacteristics": [
          {
            "avpClassification": {
              "entryDefinedValue": "avpVehicleType001"
            }
          }
        ]
      }
    ]
  },
  "referencedRightSpecification": {
    "id": "avpRightSpec",
    "version": 1
  }
}
```
Based on this request, the parking aggregator / the parking operator backend system will check availability of AVP services in this car park for this period of time.

The backend will then respond with a corresponding _QuoteRightResponse_:

```json
{
  "id": "responseId",
  "version": 1,
  "start": "2025-03-31T17:00:00Z",
  "end": "2025-04-02T10:00:00Z",
  "requestTime": "2025-03-10T18:32:12Z",
  "quoteRequestId": "requestId",
  "responseTime": "2025-03-10T18:33:02:01Z",
  "options": [
    {
      "elementId": {
        "id": "opt1",
        "version": 1
      }, 
      "exact": true,
      "quoteExpiration": "2025-03-10T22:00:00Z",
      "financialQuote": {
        "id": "finQuote1",
        "version": 1,
        "serviceProvider": {
          "id": "QPARK",
          "version": 1
        },
        "transactionId": "CONF982379278340782347",
        "value": {
          "currencyType": "EUR",
          "currencyValue": 30
        },
        "taxIncluded": true
      }
    }
  ]
}
```

In our example, the requested options can be met. The user backend is informed about the availability and the associated costs for this service. The received quote remains binding until 10pm the same day.

The user backend can then confirm the booking by asking the parking backend to create a corresponding APDS assigned right.

```
POST /rights/assigned
```
The request body contains the reference to the previously obtained quote
```json
{
  "quoteResponseId": {
    "id": "responseId",
    "version": 1
  }
}
```

The parking system will then have to confirm this:

```json
{
  "code": 201,
  "status": "ok",
  "message": "booking confirmed"
}
```

With that, the AVP parking service has been booked, and the car park should be awaiting the drivers arrival at the specified date and time.

# Use Case 2: Manual Parking (L2 parking assistance)
This is the basic use case where a traditional vehicle or a vehicle equipped with L2 parking assistance is being used. An overview of the identified use cases can be found [here](https://docs.ai-parking.cloud/use_cases). In this section, we'll look at the following steps of this use case:

* Before and while driving
  * Where is the parking facility?
  * How much does it cost?
  * Selection of a parking facility
  * Activation of in-vehicle navigation to entry
* Arriving/Entry
  * Open barrier
  * start parking transaction
* Where is the spot?
  * Occupancy information about area
  * Occupancy information about spot
* Navigation to spot
  * Guiding and driving (L2) of customer within garage and 3D navigation map to free slot
* Park in/out
  * L2 park in/out
  * In slot or L2-assisted relocation for charging and other value-added services
* Charging
  * Manual plugin of cable
  * Manual selection of tariff
  * Manual start of charging service
* Navigation to exit
  * Guiding and driving (L2) of customer within garage and 3D navigation map from slot to exit
* Exit
  * Open barrier
  * Stop parking transaction
* Payment
  * Charge parking fee to customer account
  * Charge charging fee to customer account


## Availability query (where is the parking facility, and how much does it cost?)
When the GET /places request is made, desired service types can be specified in the query. The caller can differentiate between __required qualifications__ and __optional qualifications__.  

If none of the parking locations in the inventory matches the _required qualifications_, the result set will be empty. If the qualifications are tagged as _optional_, the result might as well include non-AVP locations. Especially in the early days of publicly available AVP, this might be required/helpful due to an initially low number of matching locations.  


### Example
The user backend serving a vehicle's user frontend wants to find out if AVP services are available at a specific destination:  

```bash
GET /places \
    ?expand=all&latitude=48.177624&longitude=11.556482&radius=1000\
    &required_qualifications=avp\
    &vehicle_classification=avpVehicleType001
```

* the _expand_ parameter value of _all_ indicates that the response shall include all details about the matching places
* the _latitude_, _longitude_ and _radius_ parameters specify the destination and the acceptable radius around it
* the _required_qualifications_ parameter indicates that the caller is only interested in locations that offer AVP services
* the _vehicle_classification_ parameter specifies the AVP vehicle profile under which the vehicle falls (the classification identifiers are to be mutually defined by the OEMs)


The inventory system will respond with something like this:

```json
{
  "meta": {
    "referenceInstant": 1741612330,
    "offset": 0,
    "pageSize": 100,
    "total": 1
  },
  "data": [
    {
      "id": "carPark1Id",
      "version": 1,
      "name": [
        {
          "language": "de",
          "string": "BMW Welt Parkhaus Nordwest"
        }
      ],
      "rightSpecifications": [
        { 
          "id": "avpRightSpec",
          "version": 1
        }
      ],
      "indicativePointLocation": {
        "type": "Point",
        "coordinates": [
          11.552002,
          48.1858071
        ]
      },
      "description": [
        {
          "language": "en",
          "string": "The preferred parking location for visitors to the BMW Welt"
        }
      ],
      
      "areaType": "generalParking",
      "characteristics": {
        "accessControlled": true,
        "evChargingPoints": 15,
        "openToPublic": true,
        "spacesTotal": 552,
        "structureGrade": "underGround",
        "structureType": "offStreetStructure"
      },
      "contacts": [
        {
          "organisationName": [
            {
              "language": "de",
              "string": "BMW"
            }
          ],
          "type": "operator",
          "emails": [
            {
              "address": "customersupport@bmw.com",
              "typeCode": "customerService"
            }
          ]
        }
      ],
      "paymentMethods": [
        {
          "paymentMode": [
            "payAfterExit", "payAndExit"
          ]
        }
      ],
      "placeStreetAddress": {
        "postCode": "80809",
        "city": [
          {
            "language": "de",
            "string": "München"
          },
          {
            "language": "en",
            "string": "Munich"
          }
        ],
        "countryCode": "DE",
        "addressLines": [
          {
            "type": "street",
            "order": 0,
            "text": "Moosacher Straße 61"
          }
        ]
      }      
    }
  ]
}
```
The caller will then have to query the details of the referenced right specification.

```
GET /rights/specs/avpRightSpec?expand=all
```

The inventory system will provide the requested details:  

```json
{
  "id": "avpRightSpec",
  "version": 1,
  "rateEligibility": [
    {
      "eligibility": [
        {
          "qualifications": [
            {
              "withReservation": true,
              "vehicleCharacteristics": [
                {
                  "avpClassification": {
                    "entryDefinedValue": "avpVehicleType001"
                  }
                },
                {
                  "avpClassification": {
                    "entryDefinedValue": "avpVehicleType002"
                  }
                }
              ] 
            } 
          ]
        }
      ],
      "rateTable": {
        "id": "avpTariffId",
        "version": 1
      },
      "hierarchyElements": [
        {
          "id": "carPark1Id",
          "version": 1
        }
      ]
    }
  ],
  "validity": {
    "validityStatus": "definedByValidityTimeSpec",
    "validityTimeSpecification": {
      "overallStartTime": "2025-01-01T00:00:00Z"
    }
  },
  "transferable": false
}
```

From this response, the caller can see that his vehicle is eligible to use AVP services in the car park.  
Depending on the user interaction design of the user frontend, the system could now query tariff details (or do this later when requesting a quote in preparation of a reservation).  

```json
{
    "id": "avpTariffId",
    "version": 1,
    "rateTableName": [
        {
            "language": "en",
            "string": "AVP Parking Tariff"
        }
    ],
    "rateLineCollections": [
        {
            "applicableCurrency": "EUR",
            "collectionSequence": 0,
            "maxTime": "PT12H",
            "rateLines": [
                {
                    "sequence": 0,
                    "description": [
                        {
                            "language": "en",
                            "string": "up to 1 hour"
                        }
                    ],
                    "rateLineType": "incrementingRate",
                    "value": 10.0,
                    "incrementPeriod": "PT1H"
                },
                {
                    "sequence": 1,
                    "description": [
                        {
                            "language": "en",
                            "string": "up to 2 hours"
                        }
                    ],
                    "rateLineType": "incrementingRate",
                    "value": 10.0,
                    "incrementPeriod": "PT1H"
                },
                {
                  "sequence": 2,
                  "description": [
                    {
                      "language": "en",
                      "string": "any following hour"
                    }
                  ],
                  "rateLineType": "incrementingRate",
                  "value": 4.0,
                  "incrementPeriod": "PT1H"
                }
            ],
            "relativeTimes": true
        }
    ],
    "availability": "public",
    "rateResponsibleParty": "BMW"
}
```

## Making a reservation
Once the driver has - based on the previously obtained inventory information - decided to use the AVP car park at the destination, the user backend will have to  
* obtain a binding offer (APDS term: quote) and then
* confirm this (i.e. book it)

When the POST /quotes QuoteRightRequest is sent, a right specification shall be used that has a matching vehicleCharacteristics. The list of available right specifications was retrieved earlier during the discovery process.

```
POST /quotes
```
_(the request body contains a so-called QuoteRightRequest)_
```json
{
  "id": "requestId",
  "version": 1,
  "periodStart": "2025-03-31T17:00:00Z",
  "periodEnd": "2025-04-02T10:00:00Z",
  "requestTime": "2025-03-10T18:32:12Z",
  "eligibility": {
    "qualifications": [
      {
        "vehicleCharacteristics": [
          {
            "avpClassification": {
              "entryDefinedValue": "avpVehicleType001"
            }
          }
        ]
      }
    ]
  },
  "referencedRightSpecification": {
    "id": "avpRightSpec",
    "version": 1
  }
}
```
Based on this request, the parking aggregator / the parking operator backend system will check availability of AVP services in this car park for this period of time.

The backend will then respond with a corresponding _QuoteRightResponse_:

```json
{
  "id": "responseId",
  "version": 1,
  "start": "2025-03-31T17:00:00Z",
  "end": "2025-04-02T10:00:00Z",
  "requestTime": "2025-03-10T18:32:12Z",
  "quoteRequestId": "requestId",
  "responseTime": "2025-03-10T18:33:02:01Z",
  "options": [
    {
      "elementId": {
        "id": "opt1",
        "version": 1
      }, 
      "exact": true,
      "quoteExpiration": "2025-03-10T22:00:00Z",
      "financialQuote": {
        "id": "finQuote1",
        "version": 1,
        "serviceProvider": {
          "id": "QPARK",
          "version": 1
        },
        "transactionId": "CONF982379278340782347",
        "value": {
          "currencyType": "EUR",
          "currencyValue": 30
        },
        "taxIncluded": true
      }
    }
  ]
}
```

In our example, the requested options can be met. The user backend is informed about the availability and the associated costs for this service. The received quote remains binding until 10pm the same day.

The user backend can then confirm the booking by asking the parking backend to create a corresponding APDS assigned right.

```
POST /rights/assigned
```
The request body contains the reference to the previously obtained quote
```json
{
  "quoteResponseId": {
    "id": "responseId",
    "version": 1
  }
}
```

The parking system will then have to confirm this:

```json
{
  "code": 201,
  "status": "ok",
  "message": "booking confirmed"
}
```

With that, the AVP parking service has been booked, and the car park should be awaiting the drivers arrival at the specified date and time.


# Use Case 3: AVP Parking L4 (type 1,2,3)
This is the use case where a vehicle or a vehicle equipped with L4 capabilities is being used. An overview of the identified use cases can be found [here](https://docs.ai-parking.cloud/use_cases). In this section, we'll look at the following steps of this use case:

* Before and while driving
  * Where is the AVP parking facility?
  * How much does it cost?
  * Selection of a parking facility
  * Activation of in-vehicle navigation to entry
* Arriving/Entry
  * Open barrier
  * start parking transaction
* Where is the spot?
  * Occupancy information about area
  * Occupancy information about spot
* Navigationto spot
  * Guiding and driving (L4) of customer within garage and 3D navigation map to free spot
* Park in/out
  * Automated park in/out
  * Automated relocation for charging and other services
* Charging
  * Automated plugin of cable
  * Automated selection of tariff
  * Automatic start of charging service
* Navigation to exit
  * Guiding and driving (L4) of customer within garage and 3D navigation map from slot to exit
* Exit
  * Open barrier
  * Stop parking transaction
* Payment
  * Charge parking fee to customer account
  * Charge charging fee to customer account


## Availability query (where is the parking facility, and how much does it cost?)
When the GET /places request is made, desired service types can be specified in the query. The caller can differentiate between __required qualifications__ and __optional qualifications__.  

If none of the parking locations in the inventory matches the _required qualifications_, the result set will be empty. If the qualifications are tagged as _optional_, the result might as well include non-AVP locations. Especially in the early days of publicly available AVP, this might be required/helpful due to an initially low number of matching locations.  


### Example
The user backend serving a vehicle's user frontend wants to find out if AVP services are available at a specific destination:  

```bash
GET /places \
    ?expand=all&latitude=48.177624&longitude=11.556482&radius=1000\
    &required_qualifications=avp\
    &vehicle_classification=avpVehicleType001
```

* the _expand_ parameter value of _all_ indicates that the response shall include all details about the matching places
* the _latitude_, _longitude_ and _radius_ parameters specify the destination and the acceptable radius around it
* the _required_qualifications_ parameter indicates that the caller is only interested in locations that offer AVP services
* the _vehicle_classification_ parameter specifies the AVP vehicle profile under which the vehicle falls (the classification identifiers are to be mutually defined by the OEMs)


The inventory system will respond with something like this:

```json
{
  "meta": {
    "referenceInstant": 1741612330,
    "offset": 0,
    "pageSize": 100,
    "total": 1
  },
  "data": [
    {
      "id": "carPark1Id",
      "version": 1,
      "name": [
        {
          "language": "de",
          "string": "BMW Welt Parkhaus Nordwest"
        }
      ],
      "rightSpecifications": [
        { 
          "id": "avpRightSpec",
          "version": 1
        }
      ],
      "indicativePointLocation": {
        "type": "Point",
        "coordinates": [
          11.552002,
          48.1858071
        ]
      },
      "description": [
        {
          "language": "en",
          "string": "The preferred parking location for visitors to the BMW Welt"
        }
      ],
      
      "areaType": "generalParking",
      "characteristics": {
        "accessControlled": true,
        "evChargingPoints": 15,
        "openToPublic": true,
        "spacesTotal": 552,
        "structureGrade": "underGround",
        "structureType": "offStreetStructure"
      },
      "contacts": [
        {
          "organisationName": [
            {
              "language": "de",
              "string": "BMW"
            }
          ],
          "type": "operator",
          "emails": [
            {
              "address": "customersupport@bmw.com",
              "typeCode": "customerService"
            }
          ]
        }
      ],
      "paymentMethods": [
        {
          "paymentMode": [
            "payAfterExit", "payAndExit"
          ]
        }
      ],
      "placeStreetAddress": {
        "postCode": "80809",
        "city": [
          {
            "language": "de",
            "string": "München"
          },
          {
            "language": "en",
            "string": "Munich"
          }
        ],
        "countryCode": "DE",
        "addressLines": [
          {
            "type": "street",
            "order": 0,
            "text": "Moosacher Straße 61"
          }
        ]
      }      
    }
  ]
}
```
The caller will then have to query the details of the referenced right specification.

```
GET /rights/specs/avpRightSpec?expand=all
```

The inventory system will provide the requested details:  

```json
{
  "id": "avpRightSpec",
  "version": 1,
  "rateEligibility": [
    {
      "eligibility": [
        {
          "qualifications": [
            {
              "withReservation": true,
              "vehicleCharacteristics": [
                {
                  "avpClassification": {
                    "entryDefinedValue": "avpVehicleType001"
                  }
                },
                {
                  "avpClassification": {
                    "entryDefinedValue": "avpVehicleType002"
                  }
                }
              ] 
            } 
          ]
        }
      ],
      "rateTable": {
        "id": "avpTariffId",
        "version": 1
      },
      "hierarchyElements": [
        {
          "id": "carPark1Id",
          "version": 1
        }
      ]
    }
  ],
  "validity": {
    "validityStatus": "definedByValidityTimeSpec",
    "validityTimeSpecification": {
      "overallStartTime": "2025-01-01T00:00:00Z"
    }
  },
  "transferable": false
}
```

From this response, the caller can see that his vehicle is eligible to use AVP services in the car park.  
Depending on the user interaction design of the user frontend, the system could now query tariff details (or do this later when requesting a quote in preparation of a reservation).  

```json
{
    "id": "avpTariffId",
    "version": 1,
    "rateTableName": [
        {
            "language": "en",
            "string": "AVP Parking Tariff"
        }
    ],
    "rateLineCollections": [
        {
            "applicableCurrency": "EUR",
            "collectionSequence": 0,
            "maxTime": "PT12H",
            "rateLines": [
                {
                    "sequence": 0,
                    "description": [
                        {
                            "language": "en",
                            "string": "up to 1 hour"
                        }
                    ],
                    "rateLineType": "incrementingRate",
                    "value": 10.0,
                    "incrementPeriod": "PT1H"
                },
                {
                    "sequence": 1,
                    "description": [
                        {
                            "language": "en",
                            "string": "up to 2 hours"
                        }
                    ],
                    "rateLineType": "incrementingRate",
                    "value": 10.0,
                    "incrementPeriod": "PT1H"
                },
                {
                  "sequence": 2,
                  "description": [
                    {
                      "language": "en",
                      "string": "any following hour"
                    }
                  ],
                  "rateLineType": "incrementingRate",
                  "value": 4.0,
                  "incrementPeriod": "PT1H"
                }
            ],
            "relativeTimes": true
        }
    ],
    "availability": "public",
    "rateResponsibleParty": "BMW"
}
```

## Making a reservation
Once the driver has - based on the previously obtained inventory information - decided to use the AVP car park at the destination, the user backend will have to  
* obtain a binding offer (APDS term: quote) and then
* confirm this (i.e. book it)

When the POST /quotes QuoteRightRequest is sent, a right specification shall be used that has a matching vehicleCharacteristics. The list of available right specifications was retrieved earlier during the discovery process.

```
POST /quotes
```
_(the request body contains a so-called QuoteRightRequest)_
```json
{
  "id": "requestId",
  "version": 1,
  "periodStart": "2025-03-31T17:00:00Z",
  "periodEnd": "2025-04-02T10:00:00Z",
  "requestTime": "2025-03-10T18:32:12Z",
  "eligibility": {
    "qualifications": [
      {
        "vehicleCharacteristics": [
          {
            "avpClassification": {
              "entryDefinedValue": "avpVehicleType001"
            }
          }
        ]
      }
    ]
  },
  "referencedRightSpecification": {
    "id": "avpRightSpec",
    "version": 1
  }
}
```
Based on this request, the parking aggregator / the parking operator backend system will check availability of AVP services in this car park for this period of time.

The backend will then respond with a corresponding _QuoteRightResponse_:

```json
{
  "id": "responseId",
  "version": 1,
  "start": "2025-03-31T17:00:00Z",
  "end": "2025-04-02T10:00:00Z",
  "requestTime": "2025-03-10T18:32:12Z",
  "quoteRequestId": "requestId",
  "responseTime": "2025-03-10T18:33:02:01Z",
  "options": [
    {
      "elementId": {
        "id": "opt1",
        "version": 1
      }, 
      "exact": true,
      "quoteExpiration": "2025-03-10T22:00:00Z",
      "financialQuote": {
        "id": "finQuote1",
        "version": 1,
        "serviceProvider": {
          "id": "QPARK",
          "version": 1
        },
        "transactionId": "CONF982379278340782347",
        "value": {
          "currencyType": "EUR",
          "currencyValue": 30
        },
        "taxIncluded": true
      }
    }
  ]
}
```

In our example, the requested options can be met. The user backend is informed about the availability and the associated costs for this service. The received quote remains binding until 10pm the same day.

The user backend can then confirm the booking by asking the parking backend to create a corresponding APDS assigned right.

```
POST /rights/assigned
```
The request body contains the reference to the previously obtained quote
```json
{
  "quoteResponseId": {
    "id": "responseId",
    "version": 1
  }
}
```

The parking system will then have to confirm this:

```json
{
  "code": 201,
  "status": "ok",
  "message": "booking confirmed"
}
```

With that, the AVP parking service has been booked, and the car park should be awaiting the drivers arrival at the specified date and time.




## Prerequisites
### OEM-defined classification
The OEMs offering AVP will have to mutually maintain a publicly available list of AVP vehicle classifications.

Example:

```json
{
  "id": "AVP-CLASSIFICATION-LIST",
  "version": 1,
  "creator": {
    "id": "CarOEMs",
    "version": 1
  },
  "locator": "https://codelists.oemrepository.online",
  "userDefinedCodeListEntries": [
    {
      "entryIndex": 0,
      "definedValue": "avpVehicleType001",
      "entryDescription": "optional description of the AVP vehicle classification"
    },
    {
      "entryIndex": 1,
      "definedValue": "avpVehicleType002",
      "entryDescription": "optional description of the AVP vehicle classification"
    }
  ]
}
```







