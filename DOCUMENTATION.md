---
title: ECSIM - a simple dynamic energy cost simulator
author: Jaap-Henk Hoepman
---

ECSIM is a simple dynamic energy cost simulator. It uses historical data about hourly electric energy costs together with average figures about the distribution of both energy consumption and solar energy production to compute the expected energy cost for a household. If a home battery is used, possibly attached to an electric vehicle, it can compute an optimal daily battery charging strategy that takes advantage of varying hourly tariffs to minimise the costs.

# Model

Assume a house with a local grid to which electric appliances (consuming energy), solar panels (returning energy) and an electric vehicle with a battery
(that can both be charge or return energy) are attached. Let

- $e_u$ be the total energy consumed by the electric appliances in a year, 
- $e_v$ be the total energy consumed by the electric vehicle in a year, and
- $e_s$ be the total solar energy returned by the solar panels in a year.

![Model](./model.pdf){#fig:figlabel}

All energy is measured in kWh. The local grid of the house is connected to the electrical grid through a (smart) meter. This meter measures:

- $e[d,h]$ the hourly amount of electric energy consumed (positive) or returned (negative) to the grid (where day $d \in [0,\ldots,364]$ and hour $h \in [0,\ldots,23]$).

The hourly amount of energy consumed or produced depends on the amount of energy produced by the solar panels that hour ($e_s[d,h]$), the amount of energy consumed by appliances that hour ($e_c[d,h]$) and the amount of energy flowing into or out of the battery ($b[d,h]$). The latter is called the battery charging strategy is discussed later. We have

$$e[d,h]=e_c[d,h]+b[d,h]-e_s[d,h].$$

The amount of energy produced by the solar panels at a particular hour on a particular day is estimated using 

- information about the average fraction of yearly solar production produced each month, and
- information about the average fraction of daily solar production produced each hour.

The same method is used to estimate the hourly energy consumption.

Let

- $e_c$ be the total energy consumed from the gird in a year, and
- $e_r$ be the total energy returned to the grid in in a year.

We have 
$$e_n = e_c - e_r = \sum_{d=0}^{354} \sum_{h=0}^{23} e[d,h].$$

Typically, some of the solar energy produced is immediately consumed by household appliances, and therefore some of the production and consumption is not measured by the meter. We have $e_c \le e_u$ and $e_r \le e_s$.

Assuming no losses when charging or discharging the battery, and provided the battery is empty at the start and the end of a year, we also have
$e_n = e_u + e_v - e_s$. This follows from the fact that whatever enters the battery needs to leave it at some point, and any excess solar energy not used to charge the battery needs to leave the local grid through the meter, and any additional energy necessary to charge the battery or run household appliances also needs to enter through the meter.

# Typical electricity cost structure

The price you pay for electricity is determined by several factors. There are *fixed* costs (e.g. network operator connection costs, and energy supplier subscription fees) and *variable* costs (that depend on the amount of electricity, measured in kWh, consumed or returned). Fixed costs are ignored in the simulation.

The variable costs can be further divided into

- raw electricity price (EPEX) $p$ (per kWh, changes every hour),
- energy supplier surcharge $s$ (per kWh), and
- government taxes and levies $t$ (per kWh).

Of these, only the electricity price itself is dynamic and changes every hour. The other factors are constant for a year. The tariffs for the next day are typically published at 15:00 the day before. These EPEX tariffs [apparently differ](https://mijn.easyenergy.com/nl/api/tariff/getapxtariffs?startTimestamp=2023-04-18T22%3A00%3A00.000Z&endTimestamp=2023-04-19T22%3A00%3A00.000Z&grouping=) for electricity consumption and energy returned (i.e. through solar panels or wind turbines). But most dynamic energy suppliers quote a *single* hourly tariff for both electrical energy consumed and returned. This is what the simulator also does (for technical reasons). We write $p[d,h]$ for the price for a kWh worth of energy on day $d \in [0,\ldots,364]$ at hour $h \in [0,\ldots,23]$. 

Government taxes and levies *only* apply to the *net* energy consumption over the (annual) billing period. If you consumed $e_c$ energy and returned $e_r$ energy in a year, the total taxes and levies due are determined by $e_n = e_c - e_r$: if $e_n \le 0$ *no* taxes or levies are due. In other words, 
if we define $Z(x)$ as 
$$Z(x) = \begin{cases} 
            x & \textit{if $x \ge 0$} \\
			0 & \textit{otherwise}
			\end{cases}
$$
then the total taxes due equal $Z(e_n) \times t$.

Energy supplier surcharges contribute to the energy cost in different ways. Some energy suppliers charge the surcharge *both* for energy consumed and for energy returned. Others only charge the surcharge for the excess of energy consumed (i.e. like the government taxes). 

The total cost for the electrical energy consumed for a full year is computed as follows

- For energy suppliers that charge the surcharge for both energy consumption and return, the total cost for electrical energy becomes:
  $$Z(e_n) \times t + \sum_{d=0}^{354} \sum_{h=0}^{23} e[d,h] \times p[d,h] + |e[d,h]| \times s$$
  (where $|x|$ denotes the absolute value of $x$). 
- For energy suppliers that charge the surcharge only for the excess energy consumed, the total cost for electrical energy becomes:
  $$Z(e_n) \times (s+t) + \sum_{d=0}^{354} \sum_{h=0}^{23} e[d,h] \times p[d,h]$$

In the simulator, two separate surcharges $s_n$ and $s_x$ are defined, and following formula for the total cost is used:

  $$Z(e_n) \times (s_x+t) + \sum_{d=0}^{354} \sum_{h=0}^{23} e[d,h] \times p[d,h] + |e[d,h]| \times s_n$$

Observe that the first case above corresponds to $s_n=s$ and $s_x=0$ while
the second case above corresponds to $s_n=0$ and $s_x=s$.

# Finding an optimal battery charge strategy

The battery in the electric vehicle is primarily used to power the vehicle of course. It can also be used to take advantage of the fluctuating dynamic prices by charging it when electricity is cheap and discharging it when electricity is more expensive. It can also be used to temporarily store solar energy to return it when local energy demand is high or electricity prices are high. To accomplish this, a *battery charging strategy* $b[d,h]$ must be computed that instructs the battery when to charge or discharge. 

The simulator computes this charging strategy, using the following (generic) method. It assumes it is given some advance knowledge of the hourly tariffs, future energy consumption and solar energy production. In practice, when energy tariffs for tomorrow are published at 15:00 today, the look-ahead interval is $33=24+9$ hours. So assume the following input (where $H$ is the last hour in the look-ahead interval):

- $e_c[h]$, the hourly energy consumption, for $h \in [0,\ldots,H]$,
- $e_s[h]$, the hourly solar energy production, for $h \in [0,\ldots,H]$,
- $e_v[h]$, the hourly electric vehicle energy consumption, for $h \in [0,\ldots,H]$,
- $p[h]$ the raw hourly energy tariff, for $h \in [0,\ldots,H]$.

The output is a battery charging schedule $b[h]$ for these hours. 

Let $e[h] =e_c[h]+b[h]-e_s[h]$. The goal is to minimise

$$ \sum_{h=0}^{H} e[h] \times p[h] + |e[h]| \times s_n$$ 

under certain constraints. (Observe that any energy consumed by the electric vehicle is hidden in the battery charge.)

These constraints are the following.



# Simulating



taxes: levies:
> Indien je meer teruglevert dan je verbruikt, dan kun je dus het volledige bedrag dat je in dat jaar hebt betaald aan belastingen en heffingen wegstrepen, maar je krijgt geen belastingen en heffingen vergoed over alles wat je daarboven nog hebt teruggeleverd. Je komt dus dan in feite op nul uit.

https://www.easyenergy.com/nl/klantenservice/stroom-terugleveren/vergoeding-belastingen-heffingen



https://www.easyenergy.com/media/1214/tarievenblad-stroom-en-gas-onbepaalde-tijd-01012023.pdf
