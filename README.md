# IMC Prosperity 3 Backtester

[![Build Status](https://github.com/jmerle/imc-prosperity-3-backtester/workflows/Build/badge.svg)](https://github.com/jmerle/imc-prosperity-3-backtester/actions/workflows/build.yml)
[![PyPI Version](https://img.shields.io/pypi/v/prosperity3bt)](https://pypi.org/project/prosperity3bt/)

This repository contains a backtester [IMC Prosperity 3](https://prosperity.imc.com/) algorithms, based on [my backtester for Prosperity 2](https://github.com/jmerle/imc-prosperity-2-backtester). The output it generates closely matches the format of the output generated by the official submission environment and is therefore compatible with my [Prosperity 3 Visualizer](https://github.com/jmerle/imc-prosperity-3-visualizer) (assuming your code contains the visualizer's required prerequisites as explained on the visualizer's homepage).

## Usage

Basic usage:
```sh
# Install the latest version of the backtester
$ pip install -U prosperity3bt

# Run the backtester on an algorithm using all data from round 0
$ prosperity3bt <path to algorithm file> 0
```

Run `pip install -U prosperity3bt` again when you want to update the backtester to the latest version.

Some more usage examples:
```sh
# Backtest on all days from round 1
$ prosperity3bt example/starter.py 1

# Backtest on round 1 day 0
$ prosperity3bt example/starter.py 1-0

# Backtest on round 1 day -1 and round 1 day 0
$ prosperity3bt example/starter.py 1--1 1-0

# Backtest on all days from rounds 1 and 2
$ prosperity3bt example/starter.py 1 2

# You get the idea

# Merge profit and loss across days
$ prosperity3bt example/starter.py 1 --merge-pnl

# Automatically open the result in the visualizer when done
# Assumes your algorithm logs in the visualizer's expected format
$ prosperity3bt example/starter.py 1 --vis

# Write algorithm output to custom file
$ prosperity3bt example/starter.py 1 --out example.log

# Skip saving the output log to a file
$ prosperity3bt example/starter.py 1 --no-out

# Backtest on custom data
# Requires the value passed to `--data` to be a path to a directory that is similar in structure to https://github.com/jmerle/imc-prosperity-3-backtester/tree/master/prosperity3bt/resources
$ prosperity3bt example/starter.py 1 --data prosperity3bt/resources

# Print trader's output to stdout while running
# This may be helpful when debugging a broken trader
$ prosperity3bt example/starter.py 1 --print
```

## Order Matching

Orders placed by `Trader.run` at a given timestamp are matched against the order depths and market trades of that timestamp's state. Order depths take priority, if an order can be filled completely using volume in the relevant order depth, market trades are not considered. If not, the backtester matches your order against the timestamp's market trades. In this case the backtester assumes that for each trade, the buyer and the seller of the trade are willing to trade with you instead at the trade's price and volume. Market trades are matched at the price of your orders, e.g. if you place a sell order for €9 and there is a market trade for €10, the sell order is matched at €9 (even though there is a buyer willing to pay €10, this appears to be consistent with what the official Prosperity environment does).

Matching orders against market trades can be configured through the `--match-trades` option:
- `--match-trades all` (default): match market trades with prices equal to or worse than your quotes.
- `--match-trades worse`: match market trades with prices worse than your quotes, inspired by [team Linear Utility's Prosperity 2 write-up](https://github.com/ericcccsliu/imc-prosperity-2).
- `--match-trades none`: do not match market trades against orders.

Limits are enforced before orders are matched to order depths. If for a product your position would exceed the limit, assuming all your orders would get filled, all your orders for that product get canceled.

## Data Files

Data for the following rounds is included:
- Round 0: prices and anonymized trades data on RAINFOREST_RESIN and KELP that was used during tutorial submission runs. The anonymized trades data is derived from the submission of an algorithm that places no orders.
- Round 1: prices and anonymized trades data on RAINFOREST_RESIN and KELP.
- Round 6: prices and anonymized trades data that was used during submission runs. Round 6 day X represents the submission data of round X, where X = 0 means the tutorial round. The anonymized trades data is derived from the submission of an algorithm that places no orders.

## Environment Variables

During backtests two environment variables are set for the trader to know the round and day it's being backtested on. The environment variable named `PROSPERITY3BT_ROUND` contains the round number and `PROSPERITY3BT_DAY` contains the day number. Note that these environment variables do not exist in the official submission environment, so make sure the code you submit doesn't require them to be defined.

## Development

Follow these steps if you want to make changes to the backtester:
1. Install [uv](https://docs.astral.sh/uv/).
2. Clone (or fork and clone) this repository.
3. Open a terminal in your clone of the repository.
4. Create a venv with `uv venv` and activate it.
5. Run `uv sync`.
6. Any changes you make are now automatically taken into account the next time you run `prosperity3bt` inside the venv.
