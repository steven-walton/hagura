---
layout: post
title:  "Hydrographs in Python"
date:   2017-11-9
categories: python
---

Hydrographs are an excellent way to clearly visualize how precipitation events affect measured stream or sewer flows. While it is quite possible to develop hydrographs in Excel, there's some advantages to using python's Pandas and Matplotlib libraries over Excel:

  * Reusability: You can easily adapt your Python code for new data sets, with most robust options for automatically cleaning/processing data
  * Scalability: Handle larger data sets, or data from a large number of source files, with minimal changes to your code
  * Datetimes: Something Pandas does especially well is handle datetime objects. Less hassle resampling data or formatting a datetime axis on a plot.
  * Aesthetic: I think some of Matplotlib's plotstyles just look very good "out of the box" with only minimal adjustments.

In this example, we'll work with a set of flow meter and rainfall data to plot a set of hydrographs. I've provided a comma-delimited .csv files of the data: [flow.csv]({{ "/assets/flow.csv" }}), [precip.csv]({{ "/assets/precip.csv" }})
We have three meters - 'A', 'B', and 'C', and would like to plot a hydrograph for each meter in a single figure.

First, import data using pandas. I am resampling the precipitation data to 12-hour timesteps, because it makes for a more legible bar chart given our date range. If you are plotting up data over a short date range, you can shorter timesteps. If you are attempting to plot up years of data without resampling hourly precipitation data, the bars on your bar chart will be very thin and difficult to see.

```
import pandas as pd
import matplotlib.pyplot as plt
from matplotlib import dates as mdates

plt.style.use('ggplot') # My personal preference for plotstyle

df = pd.read_csv('flow.csv', parse_dates=True)
precip = pd.read_csv('precip.csv', parse_dates=True)
precip = precip.resample('12H').sum()

```
First, we want to set up our figure, which will consist of 3 subplots on a single 11" x 17" sheet. We will also need to setup a secondary axis for plotting our precipitation data.

```
fig, axs = plt.subplots(3, 1, figsize=(17,11))
axs2 = [ax.twinx() for ax in axs]
```   

We'd now like to iterate through each axis object in `axs` and plot a corresponding column of our flow DataFrame (df). We can achieve this with `zip`.

```
for ax, ax2, column in zip(axs, axs2, df):
    ax.plot(df[column], color='r')
    # When plotting precipitation, set width based on resample frequency in days
    ax2.bar(precip.index, precip['rainfall [in]'],
            color='#5287A7', alpha=0.35, width=0.5, align='edge')
    # Do any axis formatting you want to do on ax, ax2 inside the for loop
```

## Example axis formatting

For my axis formatting, I personally like to color both the y-ticks and y-ticklabels on `ax` to match the red color of my plotted flow rates, and likewise on `ax2` to match the plot color of the precipitation data. This makes it easy for the reader to distinguish visually what data matches the primary axis versus the secondary axis.

```
    ax.set_title(column)
    ax.set_ylabel('Flow [gpm]', color='#BA3723')
    ax2.set_ylabel('Rainfall [in]', color='#3a7998')

    for tick_label in ax.get_yticklabels():
        tick_label.set_color('#BA3723')
    for tick_label in ax2.get_yticklabels():
        tick_label.set_color('#5287A7')

```

We can do some easy formatting of our datetime axis and gridlines using Matplotlib's `mdates` module. In this case, we'll use it to set our minor grid to days, and set up our date labels to be marked weekly with the string format "Aug 15".

```
    dateFmt = mdates.DateFormatter('%b %d')
    ax.xaxis.set_major_locator(mdates.DayLocator(interval=7))
    ax.xaxis.set_major_formatter(dateFmt)
    ax.xaxis.set_minor_locator(mdates.DayLocator(interval=1))
    ax.grid(b=True, which='minor')
    ax2.grid(b=False)  
```

Once we're happy with our formatting, we can output with `fig.savefig('output.pdf')`.

Here's our result from the example data I provided:
![output.png]({{ "/assets/output.png" }})

The full-size pdf file: [output.pdf]({{ "/assets/output.pdf" }})
