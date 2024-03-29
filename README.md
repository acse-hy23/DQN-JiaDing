# DQN-JiaDing
Forked from [wenjian0202/amod-abm](https://github.com/wenjian0202/amod-abm)

<img src="https://github.com/wenjian0202/amod-abm/blob/master/media/demo.gif" width="400">

## what's `amod-abm`?

Basically, `amod-abm` is an **A**gent-**B**ased **M**odeling platform for simulating **A**utonomous **M**obility-**o**n-**D**emand systems. It is written in Python 3 and has an offline [Open Source Routing Machine](https://github.com/Project-OSRM/osrm-backend#open-source-routing-machine) working backend. The simulation platform is able to model *agents* (travelers, vehicles etc.) at the individual level and has been used in many large-scale applications, for which it provides tools to:
- support AMoD system design (fleet sizing, sharing policies, hailing rules, pricing etc.);
- test different vehicle operations and traveler behavior models;
- experiment with new dispatching algorithms for trip-vehicle assignment and real-time rebalancing. 

Almost effortlessly, this application could be applied to any urban setting to simulate AMoD systems (as long as you understand the demand there). The open-source nature also lends itself to customized extensions according to developers' own needs. You're all welcome to contribute to `amod-abm`! 

## what is included?

As of today, the following parts have been implemented:
- class `Model` for free-floating AMoD systems, with a fleet of vehicles and a central dispatcher which
  - assigns requests to vehicles based on Insertion Heuristics [[1]](http://www.sciencedirect.com/science/article/pii/0191261586900202)
  - (optional) reoptimizes the assignment based on Simulated Annealing [[2]](https://www.researchgate.net/publication/281445468_Dynamic_Shared-Taxi_Dispatch_Algorithm_with_Hybrid_Simulated_Annealing)
  - (optional) rebalances vehicles using either Simple Anticipatpry Rebalancing, Optimal Rebalancing Problem or Deep Q Network [[3]](https://mobility.mit.edu/publications/9999/wen-rebalancing-shared-mobility-demand-systems-reinforcement-learning-approach)
- a predefined demand matrix in `demand.py` with time-invariant demand volume for a list of OD pairs
- class `Veh` for (shared) autonomous vehicles
  - vehicle capacity can be set to 1 (no sharing), 2 (at most 2 travelers sharing at a time) or more
- class `Req` for requests
  - requests are generated based on their demand volumes, following Poisson process
  - requests can be either on-demand or in-advance
- class `OSRMEngine` for connecting to the OSRM routing server
  - OSRM should be compiled and map data preprocessed beforehand
  - OSRM is offline (in order to speed up) so only returns static routing
- class `RebalancingEnv` for training the deep Q network
  - it extends [keras-rl](http://keras-rl.readthedocs.io/en/latest/) and works with [Keras](https://keras.io/) and [TensorFlow](https://www.tensorflow.org/)
  - pre-computed DQN weights are in folder `weights` for use
  
The main function in `main.py` will simulate the system given input parameters from `Constants.py`. An example of the results of a typical simulation run could be found in folder `output`. System performance indicators for analysis include wait time, travel time, detour and service rate at the traveler side, as well as vehicle miles traveled, rebalancing distancc and average load at the operator side.

### upcoming features

The following features are expected to be available by the end of this year (2017):
- time-variant demand across a day
- automated interaction with the demand prediction models (so as to link with pricing)
- statistics regarding operational cost and revenue

### ongoing parts

The following parts of the code are still experimental. They might be NOT BUG-FREE:
- `simulated_annealing()` in class `Model`: slow and not very effective
- `dqn.py` for training DQN: slow by nature; current version overacts; multiagent model in development 

## how do I get started?

Python is an interpreted language and the core code could be executed without previously compiling into machine languages. However, OSRM, written in C++14, should be built from source beforehand.

The following installation guideline targets MacOS. For more information please go to OSRM [Wiki](https://github.com/Project-OSRM/osrm-backend#open-source-routing-machine). 

Install HomeBrew if not available:
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
Install wget if not available:
```
brew install wget
```
Similarly, install all other necessary dependencies:
```
brew install boost git cmake libzip libstxxl libxml2 lua tbb ccache
brew install GDAL
```
Navigate to a good directory, and clone the project from GitHub using git:
```
git clone https://github.com/wenjian0202/amod-abm.git
```
If you want to contribute back to `amod-abm` or keep fetching new features from `amod-abm` in the future, instead of cloning, fork the repo into your GitHub account before you clone: `git clone https://github.com/your_username/amod-abm.git`. 

Get into your project folder, and remove the compiled OSRM files:
```
cd amod-abm
rm -R osrm-backend-5.11.0
```
Get new OSRM source files and extract:
```
wget https://github.com/Project-OSRM/osrm-backend/archive/v5.11.0.tar.gz
tar -xzf v5.11.0.tar.gz
```
v5.11.0 is the [latest release](https://github.com/Project-OSRM/osrm-backend/releases) for the time being. To download the current version from GitHub, you can also do `git clone https://github.com/Project-OSRM/osrm-backend.git`.

Get into the folder:
```
cd osrm-backend-5.11.0
```
Make files:
```
mkdir build
cd build
cmake ../
make
cd ..
```
The `osrm-routed` executable should be working now. The next step is to grab a `.osm.pbf` OpenStreetMap extract either from [Mapzen](https://mapzen.com/data/metro-extracts/) or [Geofabrik](http://download.geofabrik.de/index.html). Pick up a place you like. For this demo, we use areas around MIT as a toy case:
```
wget https://s3.amazonaws.com/metro-extracts.mapzen.com/boston_massachusetts.osm.pbf
```
Extract the road network:
```
./build/osrm-extract boston_massachusetts.osm.pbf -p profiles/car.lua
```
Create the hierarchy:
```
./build/osrm-contract boston_massachusetts.osm.pbf
```
> The Open Source Routing Machine is a C++ implementation of a high-performance routing engine for shortest paths in OpenStreetMap road networks. It uses an implementation of Contraction Hierarchies and is able to compute and output a shortest path between any origin and destination within a few milliseconds.

We're about to launch our own routing engine! Run the OSRM engine and establish an HTTP server:
```
./build/osrm-routed boston_massachusetts.osrm
```
Here we are! Let's try sending an HTTP request and see what the response is like. Open your web browser, paste the following request and hit *Enter*. We'll find a route from Stata Center to Santouka (my favorite ramen!), in [JSON](http://www.json.org/) format:
```
http://0.0.0.0:5000/route/v1/driving/-71.0906,42.3616;-71.1159,42.3721?alternatives=false&steps=true
```
[General Options](https://github.com/Project-OSRM/osrm-backend/blob/master/docs/http.md) gives syntax for all possible services that OSRM is providing. 

Go back to your terminal. Use `Control + C` to terminate to engine. Get a coffee. And, this was easy right?

The class `OSRMEngine` has even made your life easier. It provides a series of functions for starting, calling and shutting down your engine. A demo of the simulation platform has been prepared. Run `python main.py` and see what's happening. 

## Requirements

- OS X >= 10.10
- XCode
- Python >= 3.6

## Support

You can post bug reports and feature requests in [Issues](https://github.com/wenjian0202/amod-abm/issues).

## Citing

If you use `amod-abm` in your research, you can cite it as follows:

```
@misc{wen2017amod-abm,
    author = {Wen, Jian},
    title = {amod-abm},
    year = {2017},
    publisher = {GitHub},
    journal = {GitHub repository},
    howpublished = {\url{https://github.com/wenjian0202/amod-abm}},
}
```

