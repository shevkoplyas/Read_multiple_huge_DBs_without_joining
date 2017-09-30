# What's it all about

The goal of this 1-evening project is to prototype one of the core mechanisms of
backtest engine, which supposed to read hundreds of historical bars databases in
parallel sequentially and all ordered by bar's timestamp, but wihout "join".

The problem is the size of the data. For example 10 years of 1-minute bars for
S&P500 components takes about 40 GByte of sqlite3 db files (1 db file per one symbol).
And with limitted RAM we simply can't join them all ans then sort. We'll have
to open all 500 DBs, then "select *" ordered by timestamp and then manipulate
the array of "cursors" in order to read all 500 db/tables strictly sequentially
all ordered in time.

The project tested on NetBeans 8.1 (java version "1.8.0_92").

As a 1st step let's generate 333 fake historical db files with bogus data by
running included ruby script "./create_N_databases.rb". Its output is stored
in form of 333 files under "fake_historical_bars_databases/" folder (I left
those generated bogus .db files as a part of project, so you don't have to
generate new ones).

Next step is to look at the java source code:
  - class Read_huge_table_without_join has main() function, which
    creates a new instance of "HistoricalDataPlayer" and call for "historical_data_replay"
    passing reference to algo, which will get "on_bar()" callbacks along with
    from_epoch_s, to_epoch_s values which would say which interval we want to replay
    in our backtest scenario. Leaving values out and calling:

        // 1st example: replay all available data
        HistoricalDataPlayer hdbr = new HistoricalDataPlayer(algo);
        hdbr.historical_data_replay(historical_data_folder);

        // 2nd example: replay historical bars only from given time range
        long from_epoch_s = 1231;
        long to_epoch_s = 1233;
        hdbr = new HistoricalDataPlayer(algo);
        hdbr.historical_data_replay(historical_data_folder, from_epoch_s, to_epoch_s);

Now let's run it, here's the end of the output (only covers 2nd example):

> 2nd example: replay historical bars only from given time range [1231, 1233]
> Algo.on_bar(): bar_epoch_s=1231, sym=symbol_136
> Algo.on_bar(): bar_epoch_s=1231, sym=symbol_231
> Algo.on_bar(): bar_epoch_s=1231, sym=symbol_29
> Algo.on_bar(): bar_epoch_s=1231, sym=symbol_55
> Algo.on_bar(): bar_epoch_s=1232, sym=symbol_199
> Algo.on_bar(): bar_epoch_s=1232, sym=symbol_206
> Algo.on_bar(): bar_epoch_s=1232, sym=symbol_211
> Algo.on_bar(): bar_epoch_s=1232, sym=symbol_25
> Algo.on_bar(): bar_epoch_s=1232, sym=symbol_250
> Algo.on_bar(): bar_epoch_s=1232, sym=symbol_256
> Algo.on_bar(): bar_epoch_s=1232, sym=symbol_292
> Algo.on_bar(): bar_epoch_s=1232, sym=symbol_48
> Algo.on_bar(): bar_epoch_s=1232, sym=symbol_65
> Algo.on_bar(): bar_epoch_s=1232, sym=symbol_83
> Algo.on_bar(): bar_epoch_s=1233, sym=symbol_189
> Algo.on_bar(): bar_epoch_s=1233, sym=symbol_21
> Algo.on_bar(): bar_epoch_s=1233, sym=symbol_220
> Algo.on_bar(): bar_epoch_s=1233, sym=symbol_237
> Algo.on_bar(): bar_epoch_s=1233, sym=symbol_249
> Algo.on_bar(): bar_epoch_s=1233, sym=symbol_26
> Algo.on_bar(): bar_epoch_s=1233, sym=symbol_312
> Algo.on_bar(): bar_epoch_s=1233, sym=symbol_34
> Algo.on_bar(): bar_epoch_s=1233, sym=symbol_50
> Algo.on_bar(): bar_epoch_s=1233, sym=symbol_60

As you can see our "algo" got "on_bar()" in ascending order (epoch_s lineary growing).
And yes, we should pass "bar" object (which does not even exist in this demo) and
more than one algo might be interested in "subscribing" to replay and yes - somewhere
along the way in the middle of replay one of algos might decide to add more symbols
into subscription symbol set, but all these are outside of the scope of this "how to select from multiple huge DB/tables without join" example.

Cheers,
Dmitry Shevkoplyas
https://dimon.ca/
