# Optiver-Ready-Trader-Go

<img width="500" alt="image" src="https://user-images.githubusercontent.com/8297863/235014898-e37b28aa-d479-42ce-b6c2-c60dc18c9e7d.png">

## How the Competition Works

This repository documents our submission for the [Optiver Ready Trader Go](https://readytradergo.optiver.com/) competition. This competition involves creating a algorithmic trading strategy to trade a Future and ETF, both of which are highly correlated.

<img width="614" alt="image" src="https://user-images.githubusercontent.com/8297863/235014377-92f87103-6bfa-456d-9ef8-f0d3ea3f20f2.png">

The Future was highly liquid, while the ETF was highly illiquid.

<img width="574" alt="image" src="https://user-images.githubusercontent.com/8297863/235014437-95749871-a6e3-42e7-9233-6982be55db20.png">

We were also provided with some basic template files (refer to the [/ready_trader_go](https://github.com/keanekwa/Optiver-Ready-Trader-Go/tree/main/ready_trader_go) folder) and some market data for backtesting (refer to the [/data](https://github.com/keanekwa/Optiver-Ready-Trader-Go/tree/main/data) folder) to interface with the trading system, and we had to follow these [competition rules](https://readytradergo.optiver.com/how-to-play/).

## Getting started

To run Ready Trader Go, you'll need Python version 3.11 and PySide6. You
can download Python from [www.python.org](https://www.python.org).

Once you have installed Python, you'll need to create a Python virtual
environment, and you can find instructions for creating and using virtual
environments at
[docs.python.org/3/library/venv.html](https://docs.python.org/3/library/venv.html).

To use the Ready Trader Go graphical user interface, you'll need to install
the [PySide6 package](https://pypi.org/project/PySide6/) which you can do by
running

```shell
pip3 install PySide6
```

in your Python virtual environment.

## Running a Ready Trader Go match

Before you can run an autotrader there must be a corresponding JSON configuration
file in the same directory as your autotrader executable.

To run a Ready Trader Go match with one or more autotraders, simply run:

```shell
python3 rtg.py run [AUTOTRADER FILENAME [AUTOTRADER FILENAME]]
```

For example:

```shell
python3 rtg.py run arbitrage
```

## Autotrader configuration

Each autotrader is configured with a JSON file like this:

    {
      "Execution": {
        "Host": "127.0.0.1",
        "Port": 12345
      },
      "Information": {
        "Type": "mmap",
        "Name": "info.dat"
      },
      "TeamName": "TraderOne",
      "Secret": "secret"
    }

The elements of the autotrader configuration are:

* Execution - network address for sending execution requests (e.g. to place
an order)
* Information - details of a memory-mapped file used for information messages
broadcast by the exchange simulator
* TeamName - name of the team for this autotrader (each autotrader in a match
  must have a unique team name)
* Secret - password for this autotrader

## Simulator configuration

The market simulator is configured with a JSON file called "exchange.json".
Here is an example:

    {
      "Engine": {
        "MarketDataFile": "data/market_data.csv",
        "MarketEventInterval": 0.05,
        "MarketOpenDelay": 5.0,
        "MatchEventsFile": "match_events.csv",
        "ScoreBoardFile": "score_board.csv",
        "Speed": 1.0,
        "TickInterval": 0.25
      },
      "Execution": {
        "host": "127.0.0.1",
        "Port": 12345
      },
      "Fees": {
        "Maker": -0.0001,
        "Taker": 0.0002
      },
      "Information": {
        "Type": "mmap",
        "Name": "info.dat"
      },
      "Instrument": {
        "EtfClamp": 0.002,
        "TickSize": 1.00
      },
      "Limits": {
        "ActiveOrderCountLimit": 10,
        "ActiveVolumeLimit": 200,
        "MessageFrequencyInterval": 1.0,
        "MessageFrequencyLimit": 50,
        "PositionLimit": 100
      },
      "Traders": {
        "TraderOne": "secret",
        "ExampleOne": "qwerty",
        "ExampleTwo": "12345"
      }
    }

The elements of the autotrader configuration are:

* Engine - source data file, output filename, simulation speed and tick interval
* Execution - network address to listen for autotrader connections
* Fees - details of the fee structure
* Information - details of a memory-mapped file use to broadcast information
messages to autotraders
* Instrument - details of the instrument to be traded
* Limits - details of the limits by which autotraders must abide
* Traders - team names and secrets of the autotraders

**Important:** Each autotrader must have a unique team name and password
listed in the 'Traders' section of the `exchange.json` file.

## The Ready Trader Go command line utility

The Ready Trader Go command line utility, `rtg.py`, can be used to run or
replay a match. For help, run:

```shell
python3 rtg.py --help
```

## Running a match

To run a match, use the "run" command and specify the autotraders you
wish to participate in the match:

```shell
python3 rtg.py run [AUTOTRADER FILENAME [AUTOTRADER FILENAME]]
```

Each autotrader must have a corresponding JSON file (with the same filename,
but ending in ".json" instead of ".py") which contains a unique team name
and the team name and secret must be listed in the `exchange.json` file.

It will take approximately 60 minutes for the match to complete and several
files will be produced:

* `autotrader.log` - log file for an autotrader
* `exchange.log` - log file for the simulator
* `match_events.csv` - a record of events during the match
* `score_board.csv` - a record of each autotrader's score over time

To aid testing, you can speed up the match by modifying the "Speed" setting
in the "exchange.json" configuration file - for example, setting the speed
to 2.0 will halve the time it takes to run a match. Note, however, that
increasing the speed may change the results.

When testing your autotrader, you should try it with different sample data
files by modifying the "MarketDataFile" setting in the "exchange.json"
file.

## Replaying a match

To replay a match, use the "replay" command and specify the name of the
match events file you wish to replay:

```shell
python3 rtg.py replay match_events.csv
```

# Models We Tried

We tried out a variety of different strategies, which can be split into 2 categories: arbitrage through pairs trading, and market making around the midprice.

We first started by experimenting with the arbitrage strategies, where we performed pairs trading on the ETF and Future by buying the underpriced instrument and selling the overpriced instrument. This was implemented in the following strategies:
- Arbitrage strategy between the ETF and Future. Refer to [arbitrage.py](https://github.com/keanekwa/Optiver-Ready-Trader-Go/blob/main/arbitrage.py)
- Arbitrage strategy that took into account the momentum of the prices (e.g. if both the ETF and Future prices are going up, buy more of the underpriced instrument, and sell a smaller quantity of the overpriced instrument). Refer to [momentumArbitrage.py](https://github.com/keanekwa/Optiver-Ready-Trader-Go/blob/main/momentumArbitrage.py)

While the above models were effective by themselves, they performed poorly when put alongside other traders, due to the diminishing amount of arbitrage opportunities for each trader.

As such, we decided to switch to a market-neutral market-making approach instead, whereby we seeked to profit from trading the bid-ask spread. This was implemented in the following strategies:
- Mid-price trader. Refer to [midtrader.py](https://github.com/keanekwa/Optiver-Ready-Trader-Go/blob/main/midtrader.py)
- Mid-price trader with adjustment for current inventory levels. Theoretically this should be more profitable than the midtrader above, but it was not the case for us. Refer to [midtraderInvAdj.py](https://github.com/keanekwa/Optiver-Ready-Trader-Go/blob/main/midtraderInvAdj.py)
- [Final model that was submitted] Avellaneda-Stoikov Market Making Model. This model allowed us to adjust for inventory and volatility in a much more profitable way. Refer to [ASModel.py](https://github.com/keanekwa/Optiver-Ready-Trader-Go/blob/main/ASModel.py)

From extensive testing, we concluded that the Avellaneda-Stoikov Market Making Model was the most reliable model across different market conditions.

## Avellaneda-Stoikov Market Making Model

This model works by finding an optimal reservation price (i.e. fair price we wish to trade our instrument), and optimal bid-ask spread (i.e. how wide of a spread we wish to market make).

<img width="690" alt="image" src="https://user-images.githubusercontent.com/8297863/235012352-8d091bc7-5d1e-404c-891f-37c1f7b6a381.png">

## How to Run the Model(s)

You'll need Python version 3.11 or above and [PySide6](https://pypi.org/project/PySide6/) for the GUI. To install PySide6, you can run:
```
pip3 install PySide6
```

Afterwards, you can run the different autotrader algorithms against each other by running:
```
python3 rtg.py run [AUTOTRADER FILENAME [AUTOTRADER FILENAME]]
```

For example, this is how to run the Avellaneda-Stoikov model alone:
```
python3 rtg.py run ASModel.py
```

And this is how to run all the models to see how they compare:
```
python3 rtg.py run ASModel.py arbitrage.py momentumArbitrage.py midtrader.py midtraderInvAdj.py 
```

To backtest against different market data, update the "MarketDataFile" attribute in the exchange.json file. To increase the speed of backtesting, adjust the "Speed" attribute.

<img width="330" alt="image" src="https://user-images.githubusercontent.com/8297863/235015806-e11980ac-cace-42a4-9856-efe1a2ffded9.png">

## References
- [Avellaneda M. & Stoikov S. (2006). High Frequency Trading in a Limit Order Book](https://www.researchgate.net/publication/24086205_High_Frequency_Trading_in_a_Limit_Order_Book)
- [Hummingbot Foundation. (2021). A comprehensive guide to Avellaneda & Stoikov’s market-making strategy](https://blog.hummingbot.org/2021-04-avellaneda-stoikov-market-making-strategy/)

## Collaborators
- [Beckham Wee](https://github.com/bckhm)
- [Riley Ang](https://github.com/rileyang1)
- Regan Tan
