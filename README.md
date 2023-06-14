# ECSIM

A simple energy cost simulator to compute the cost of electricity under a dynamic (hourly) tariff system.

# Usage

```
usage: ecsim [-h] [-z SOLAR] [-c CONSUMPTION] [-v VEHICLE] [-b BATTERY] [-f] [-n] [-m MAXCHARGE] [-p LOPRICE] [-P HIPRICE] [-e ENERGYTAX] [-s SURCHARGE] [-x XSURCHARGE] [-d DEBUG]
             [-a START] [-t VAT] [-o] [-y YEAR]

ECSIM. A dynamic energy cost simulator.

options:
  -h, --help            show this help message and exit
  -z SOLAR, --solar SOLAR
                        Total solar production per year in kWh (default = 0).
  -c CONSUMPTION, --consumption CONSUMPTION
                        Total energy consumption per year in kWh (default = 0).
  -v VEHICLE, --vehicle VEHICLE
                        Total electric vehicle energy usage per year in kWh (default = 0).
  -b BATTERY, --battery BATTERY
                        Total battery capacity in kWh (default = 0).
  -f, --fixed           Simulate a dynamic or fixed price model (defualt is dynamic).
  -n, --notaverage      Use different usage-return hourly tariff (defualt is to use an average hourly tariff).
  -m MAXCHARGE, --maxcharge MAXCHARGE
                        Maximum battery charge per hour in kW (default = 11).
  -p LOPRICE, --loprice LOPRICE
                        Minimum energy price per kWh in a year (default = € 0,05).
  -P HIPRICE, --hiprice HIPRICE
                        Maximum energy price per kWh in a year (default = € 0,20).
  -e ENERGYTAX, --energytax ENERGYTAX
                        Energy tax per kWh (default = € 0,15).
  -s SURCHARGE, --surcharge SURCHARGE
                        Supplier surcharge per kWh, always levied (in both directions, default = € 0,02).
  -x XSURCHARGE, --xsurcharge XSURCHARGE
                        Supplier surcharge per kWh, levied only for excess return (default = € 0,00).
  -d DEBUG, --debug DEBUG
                        Debug level (0, the default, is off).
  -a START, --start START
                        Hour of the day the simulation starts each day (when tomorrows tariffs are published, default = 15).
  -t VAT, --vat VAT     Value added tax (VAT), in percentages (defualt is 21).
  -o, --orientation     Simulate south (default) or east/west (option selected) orientation of panels.
  -y YEAR, --year YEAR  Year to simulate (defualt = 2022).
```
  
For example, to compute the expected price to pay in 2019 when having 3000 kWh hour of solar panels installed when consuming 2300 kWh energy at home, and 3500 kWh energy for your electrical vehicle with a 45 kWh battery, run 

```
./ecsim -z 3000 -c 2300 -v 3500 -b 45 -y 2019 
```

(this assumes default start time of 15:00, southern orientation of solar panels, a 2 cents surcharge for every kWh used or returned, and 21% VAT over total energy consumption). 
  
# Notes

The simulator takes average monthly/hourly fluctuations in solar production and electricity consumption into account, but these average historical fractions are based on a situation where few households use electricity for heating. (gas is predominantly used for this purpose in The Netherlands.)

A 'phone' flat rate table with an hourly fee of 10 cents can be selected for testing purposes by setting ```-y=1984```

# Installation

ECSIM is written in Python 3 and relies on the OR-Tools released by Google. See [its documentation](https://developers.google.com/optimization/install/) on how to install. 
