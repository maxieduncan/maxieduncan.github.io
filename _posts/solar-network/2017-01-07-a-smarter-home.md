---
layout: post
category : solarnetwork
title: "A Smarter Home"
tags : [energy, thoughts, greenstage, solarnetwork]
---
{% include JB/setup %}

Having recently read [The Grid](https://www.amazon.com/Grid-Fraying-Between-Americans-Energy/dp/1632865688) and getting involved in [SolarNetwork](http://solarnetwork.net/v4/) again via [Greenstage Power](http://greenstage.co.nz/power.html) (which first made me aware of many of these concepts), I thought it would be an interesting exercise to think about what might be involved in creating a Smarter Home.  

The term "Smart Home" is bandied about quite a bit these days, mainly in association with having a few connected devices such as a smart thermostat plus a few light bulbs, switches and speakers. These for the most part aren't actually that smart, simply offering some automation and the ability to be controlled by an app; it's much more about automation and convenience, both useful and appealing offerings, than creating a truly smart experience.

It becomes much more interesting when you think about the home responding invisibly to changes in the environment by reconfiguring the devices connected to the home network. More specifically a home that can respond to changes in supply and demand from the grid (or even disconnect from the grid entirely) through a combination of smart devices and battery storage, preferably backed by local electricity generation. It's this addition of local storage and local generation that creates the potential for a smarter home.

## Generation
Local generation of electricity is entirely optional as the grid can be used as the sole source of electricity. There are however both economic and resiliency reasons for having a local source of electricity generation; specifically the ability to use this electricity instead of the gird or to sell it back into the grid at peak times and secondly, having a source of electricity when the grid goes down (though this can be mitigated to a limited extent by local storage).

The most likely local source of electricity generation would be solar, with wind or even potentially hydro much rarer options. Diesel generation can't be ruled out but suffers from being dirty, temperamental and relying on an energy source (diesel) that needs to be shipped in.

## Storage
Having local storage in some form, most likely a battery pack such as [Tesla's Powerwall](https://www.tesla.com/en_NZ/powerwall), gives some huge advantages.

With local generation you have the ability to store energy for use during peak times or even to sell it back into the gird when prices are high. Even without local generation you can store energy when grid prices are low and use or sell it when the prices are high. Either way there are not only financial benefits for the home owner but benefits for the grid in either reduced demand from the home or additional supply depending on how the storage is used.

Assuming a variable local energy source such as solar, local storage also helps ensure a consistent local supply in the short term and access to energy that has been generated when demand wasn't high (such as using solar energy generated during the day in the evening).

The other benefit mentioned earlier is resiliency in the face of blackouts or brownouts, though without local generation this is very much dependent on the amount of local storage and length of the outage.

Even without a battery pack installed there's potential for electric cars to be used for energy storage, though this isn't a simple equation, as they still need to be available for transport and the technology isn't quite at a state to make it practical.

## Demand Response
Demand Response is where changes in a users consumptions are made in response to limited supply, typically caused by high demand, on the grid. This is often reflected by a higher price for electricity so there is motivation for consumers to reduce their demand during these periods. How this change in price is communicated to end users and how their demand is reduced is something that's ripe for improvement and dependent on your utilities offerings.

## Smart Devices
Homes are slowly filling up with "Smart" devices that offer a modicum of automation and power saving but the potential remains much more promising than the reality.

The most beneficial smart device readily available is the smart thermostat. Devices like the Nest will automatically change the temperature based on your preferences and schedule. This is important, as heating and cooling a home make up the majority of most homes power consumption. These devices however are not typically aware of the needs of the grid, they are after all user centric devices that may help lower consumption but don't do a lot to avoid peak times on the grid and, as most people follow similar patterns in consumption they typically increase demand at the same time.

The other smart devices tend to be less beneficial. Smart light bulbs offer some benefits in the way of automation such as turning on and off at scheduled times or when you leave the house or get home. Mostly they mean you don't have to get off the couch to switch the light. Smart speakers are almost entirely about convenience and less wires. Smart switches have some potential but they're binary options, they're either on or off; so while they can offer automation to turn devices on or off at certain times or remotely via an app, they're not as useful to schedule intensive activities at beneficial times, such as when the grid prices are at their lowest.

To really get there you need smart appliances that can be controlled remotely via a smart home hub. For example fridges and freezers that can be turned down when not in use or during peak demand; washers and driers that can be made to delay their cycles if grid prices are peaking; thermostats that react not only to user preferences but also to the current supply and demand of electricity.

## Smart Meters
Smart meters are installed by utilities to allow them to view real time (typically hourly) consumption and bill accordingly. They also potentially allow utilities to control users consumption to reduce demand during peak times, though this is something that naturally consumers are not comfortable with.

Ideally smart meters should provide a utility with an ideal opportunity to communicate with consumers about their consumption habits and when the peak times are that they can reduce consumption when prices are high; there aren't many examples out there where this has been well done.

## User Experience
So ideally what you want is some local generation, local storage and smart devices that when combined provide the ability to reduce consumption from the gird during peak times.

Equally important, if not more so for any solution to work, it needs to be non-intrusive and painless to interact with as well as provide real time and incentivising information for end users. Users who just want their electricity to be there as cheaply as possible (most consumers) shouldn't have to interact with the system if they don't want to; it should just be there and work, with an easy to understand bill at the end of the month. For those that do want it, a real time insight to their system as well as the option to control settings and see their impact on billing. Demand-response and it's benefits, as well as the ability for consumers to specify their willingness to engage, needs to be clearly communicated.

## Smart Home Hub
I thought it'd be a fun exercise to mock up an interface based on these concepts and the types of information that would be available. This isn't intended to drive what such an interface should look like, UX isn't part of my skill set, but rather is intended to provoke thought on the kind of information that would be available and how it might be used.

![Smart Home Hub Mock](/assets/posts/smart-home-hub-mock.png)

The options shown would change based on:
* whether you have local generation
* whether you have local storage
* the services offered by your utility (do they provide pricing data, can you sell electricity back into the grid, e.t.c)
* the smart devices installed in your home

There's a lot of information shown in this mock up, probably more than most people would want to see. It's also questionable how many of the preference options you would want to make accessible to consumers. There's an interesting discussion to be had around how much you want to make configurable by end users, compared to simply automating as much as possible based on previous observations and future prediction using machine learning.

### Machine Learning
There are a number of points where I show predictions of future consumption, generation and prices. These are all great cases for the use of machine learning, where previous consumption, generation, pricing and weather data would all be used to predict future trends which would be used to determine the best usage patterns of local storage, local generation and utility provided electricity.

For example a weather forecast for cold cloudy weather could cause you to use the grid to maximise local storage even though prices are higher than normal, so that you benefit later from what you expect to be even higher prices during the cold and cloudy period.

Another use would be determining if it's best to store or sell excess locally generated electricity, though this would be dependent on the services offered by the utility (e.g. if they have dynamic pricing or simply scheduled windows). Equally choosing whether to charge local storage from the grid, local generation or to not charge at all.

Machine learning would not only make these predictions available, but could potentially use them determine the best usage patterns. The outcomes we want to encourage are primarily a reduced electricity bill and secondarily reduced consumption, machine learning has the potential to identify these patterns much more reliably.

### Smart Devices
For the smart devices I've put in a very minimal interface. Ideally such devices would be auto discovered on the network and show up in the interface automatically; in reality security constraints and the diverse selection of devices available could make this impractical.

Most smart devices already offer their own apps that allow them to be controlled and scheduled, those that offer an API would allow them to integrated into a Smart Home Hub and controlled automatically. As such you can imagine significantly more options being available than the simple on off status switches I've shown in this mock. For example changing the thermostat by X degrees when prices are peaking or, delaying washing machines and dryers to a time when prices are low or there's enough local generation and/or storage available.

## SolarNetwork
The [SolarNetwork](https://github.com/SolarNetwork)  project on GitHub is an open source platform that offers most of the pieces needed to realise this, indeed it's one of the main reasons the project was created in the first place.

`SolarNetwork is a modern platform for energy management development. Capable of capturing energy information from a variety of sources, storing that information and beautifully visualising it the way you want to. SolarNetwork is 100% open-source, and focuses on the integration of distributed renewable energy sources. It’s built to be massively scalable, incredibly configurable and easy to adopt.  SolarNetwork is built for organisations who want to study and manage energy use and generation from a distributed set of assets, report on those assets and even control their behaviour remotely.`

The continuing drop in prices for both PV and battery storage, the increasing demand for green solutions to reduce the impact of global warming, all make this an evermore practical and needed solution.
