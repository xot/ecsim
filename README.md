# ECSIM

A simple energy cost simulator to compute the cost of electricity under a dynamic (hourly) tariff system.

# Usage

```
usage: ecsim [-h] [-z SOLAR] [-c CONSUMPTION] [-b BATTERY] [-m MAXCHARGE] [-v VEHICLE]
             [-p LOPRICE] [-P HIPRICE] [-e ENERGYTAX] [-s SURCHARGE] [-x XSURCHARGE] [-d]
             [-a START] [-t VAT] [-o] [-y YEAR]

ECSIM. A dynamic energy cost simulator.

options:
  -h, --help            show this help message and exit
  -z SOLAR, --solar SOLAR
                        Total solar production per year in kWh.
  -c CONSUMPTION, --consumption CONSUMPTION
                        Total energy consumption per year in kWh.
  -b BATTERY, --battery BATTERY
                        Total battery capacity in kWh.
  -m MAXCHARGE, --maxcharge MAXCHARGE
                        Maximum battery charge per hour in kW.
  -v VEHICLE, --vehicle VEHICLE
                        Total electric vehicle energy usage per year in kWh.
  -p LOPRICE, --loprice LOPRICE
                        Minimum energy price per kWh in a year.
  -P HIPRICE, --hiprice HIPRICE
                        Maximum energy price per kWh in a year.
  -e ENERGYTAX, --energytax ENERGYTAX
                        Energy tax per kWh.
  -s SURCHARGE, --surcharge SURCHARGE
                        Supplier surcharge per kWh, always levied (in both directions).
  -x XSURCHARGE, --xsurcharge XSURCHARGE
                        Supplier surcharge per kWh, levied only for excess return.
  -d, --dynamic         Simulate a dynamic or fixed price model.
  -a START, --start START
                        Hour of the day the simulation starts each day (when tomorrows
                        tariffs are published).
  -t VAT, --vat VAT     Value added tax (VAT), in percentages.
  -o, --orientation     Simulate south (default) or east/west (option selected)
                        orientation of panels.
  -y YEAR, --year YEAR  Year to simulate.
```
  
For example, to compute the expected price to pay in 2019 when having 3000 kWh hour of solar panels installed when consuming 2300 kWh energy at home, and 3500 kWh energy for your electrical vehicle with a 45 kWh battery, run 

```
./ecsim -z 3000 -c 2300 -v 3500 -b 45 -y 2019 
```

(this assumes default start time of 15:00, southern orientation of solar panels, a 2 cents surcharge for every kWh used or returned, and 21% VAT over total energy consumption). 
  
# Installation

ECSIM is written in Python 3 and relies on the OR-Tools released by Google. See [its documentation](https://developers.google.com/optimization/install/) on how to install. 
