---
layout: post
title: How Anyone Can Download and Use Historical Weather Data in Python's Pandas Library
subtitle: There's a wealth of data out there for anyone who understands how to access it!
cover-img: /assets/img/rain.jpg
gh-repo: daattali/beautiful-jekyll
gh-badge: []
tags: []
comments: true
author: Gregory Lent
---

## Global Historical Climatology Network Weather Data

In the United States, an organization called the Global Historical Climatology Network (GHCN) maintains over 100,000 weather stations. These stations record daily statistics like maximum/minimum temperature, precipitation amounts, snow levels, and more. This data is made freely available to the public. This is an incredible resource not only for researchers, business owners, and other people who would like to use the data for professional purposes, but also for hobbyists who are interested in things like comparing rainfall or temperature between years. For example, if you have exceptional rainfall in your area, it can be fun to check how it stacks up against other historically large storms! 

Unfortunately, however, the data looks like this: 

~~~
USC00052223,19971101,TMAX,172,,,0,0800
USC00052223,19971102,TMAX,111,,,0,0800
USC00052223,19971103,TMAX,117,,,0,0800
USC00052223,19971104,TMAX,200,,,0,0800
USC00052223,19971105,TMAX,194,,,0,0800
USC00052223,19971106,TMAX,150,,,0,0800
USC00052223,19971107,TMAX,211,,,0,0800
USC00052223,19971108,TMAX,222,,,0,0800
USC00052223,19971109,TMAX,117,,,0,0800
USC00052223,19971110,TMAX,17,,,0,0800
USC00052223,19971111,TMAX,33,,,0,0800
USC00052223,19971112,TMAX,-28,,,0,0800
USC00052223,19971113,TMAX,39,,,0,0800
~~~

If you're anything like me, this reads as complete nonsense to you. In this tutorial, I will walk you through the daunting process of acquiring a local historical weather data file, using the Python library Pandas to clean the data and translate it into something we can understand, and using the MatPlotLib Python library to generate some simple visualizations. To follow along with this tutorial, make sure you have [Pandas](https://pandas.pydata.org/pandas-docs/stable/getting_started/install.html) and [MatPlotLib](https://matplotlib.org/stable/users/installing/index.html) installed and ready to go. 

This tutorial is intended as a starting point for anyone who would like to use historical weather data for any purpose; once the data is cleaned and filed into a Pandas DataFrame, it is ready for any kind of analysis that you would like to perform!

**Finding Your Local GHCN Station**

The first thing we're going to do is visit the [GHCND Map Viewer](https://www.arcgis.com/apps/mapviewer/index.html?webmap=d4fb04e6b89c4a4c958efd9b3b8c092d). You should see something like this:

![map-1](https://Green9090.github.io/assets/img/map-1.png){: .mx-auto.d-block :}

Each dot on this map is a weather station which can provide historical weather data at its location. For this tutorial we're interested particularly in the blue dots, which represent GHCN stations; the other stations can be useful, but provide different kinds of data in different formats, so this guide won't work for them without some modifications.

The first thing I would recommend is to click the "Properties" icon in the top right of the window and turn the "Transparency" slider up. This will make the dots see-through, allowing us to navigate to a desired location much more easily:

![map-1](https://Green9090.github.io/assets/img/map-2.png){: .mx-auto.d-block :}

Much better.

For this tutorial, I will be using weather data from Denver, Colorado since it has a good variety of weather conditions. I place my mouse pointer over the desired location and scroll up until I have zoomed in enough to distinguish individual stations. Then, I open the filter menu (third option down on the right) and click "Add expression." 

![denver-1](https://Green9090.github.io/assets/img/denver-1.png){: .mx-auto.d-block :}

This expression will allow us to filter out dots that are not relevant to our interests. Many of these stations, for example, will have gone out of operation decades ago. We'd like to find stations that have reasonably current data. Click on the top drop down, which currently says "Creator," and scroll down to "End Date." Then, change the second drop down to "is after." I'll choose January 1, 2021 as the latest end date I'm willing to accept.

![denver-2](https://Green9090.github.io/assets/img/denver-2.png){: .mx-auto.d-block :}

Notice that all but one blue dot has now been grayed out; this means that they do not fit my search criteria. Let's click on the remaining blue dot to get some information about it. 

![station-1](https://Green9090.github.io/assets/img/station-1.png){: .mx-auto.d-block :}

Great; it looks like this station has been in operation since 1997 and has data at least as current as March 2021. In my experience, this map is often out of date, and this case is a good example. Even though it lists an end date of March 2021, the data provided by this station is actually current up to March 2024 (the present date as of the time of this writing)! If you're looking for very current data, you might need to lower your standards and hope that the end dates of your local stations are similarly lagged. Searching for recent end dates is still a useful way to rule out the many stations that went out of service decades ago.

Now that we've found a station with a timeframe we're happy with, we need to find its station ID. Scroll down to the bottom of this box and you will find a Station ID field:

![station-2](https://Green9090.github.io/assets/img/station-2.png){: .mx-auto.d-block :}

In this example, my station ID is "USC00052223." Copy this to your clipboard; we'll use it in the next step.

**Downloading and Unpacking the Station Data**

Now that we have a station we're interested in, it's time to track down the data file. We'll be using NOAA's (National Oceanic and Atmospheric Administration) [daily weather database](https://www.ncei.noaa.gov/pub/data/ghcn/daily/by_station/). This website is a list of all of the daily station data NOAA provides, one file for each station. Press Control+F on your keyboard to bring up a search box, and copy the station ID you're interested in into the box. You may need to wait longer than you expect for your browser to locate the right file (this is a LONG list), but after a few moments you should be taken to a link to the data file you want. In my case, I'm downloading the file [USC00052223.csv.gz](https://www.ncei.noaa.gov/pub/data/ghcn/daily/by_station/USC00052223.csv.gz).

The next step is to unpack this file. I found that the native Windows unzipping program was not capable of opening these .gz files for some reason. The free, open source unzipping program [7-zip](https://www.7-zip.org/download.html) is a good way to open these if you don't already have a program that can manage it. Once you have the file open, click the extract button on 7-zip (or any other unzipping software) to take the .csv file out of the archive. 

Once you have the file, you can open it in a text editor like notepad or a spreadsheet software like Microsoft Excel or Google Slides. You should see something similar to what I presented at the top of the article; in other words, a bunch of nonsense separated by commas. Read on to find out how to translate this file into something we can use.

**How the Data is Formatted**

The source of the information presented in this section is the [readme file](https://www.ncei.noaa.gov/pub/data/ghcn/daily/readme.txt) provided by GHCN; if you have any questions about the data not answered here, you can refer to the readme. 

Let's look at a line of this data file:

~~~
USC00052223,19980125,TMIN,-6,,I,0,0800
~~~

You may now notice that the first column of the data is the station ID. This is going to be the same 11 characters repeated on each line of the data file.

The second number is the date. It's presented in the somewhat confusing format YYYYMMDD. This line of data is from January 25, 1998.

The next column is called the "element" column. This tells us what kind of measurement is being recorded. In this case, the element is "TMIN"; the minimum temperature recorded on that day at that station. Other columns of interest to us are "TMAX" (maximum temperature), "PRCP" (amount of rainfall), "SNOW" (amount of snowfall), and "SNWD" (depth of snow on the ground). There are a number of more specialized elements that are not consistently recorded by all stations; for example, "AWDR" (average wind direction).

Following the element column, we have the "measurement" column. This is the measurement associated with the given element on this day. In this case, -6 is the lowest temperature recorded at this station on January 25, 1998. The units may surprise you: this measurement is recorded in tenths of a degree Celsius! The lowest temperature recorded by the station was -0.6 degrees Celsius, or about 30.9 degrees Fahrenheit. Some other surprising units: PRCP is measured in tenths of a millimeter, but snowfall and snow depth are measured in plain old millimeters. It's easy to trip up on these! In this tutorial, we'll be using Python code to properly convert these measurements into more familiar and practical units. If you're looking at columns not covered here, look up the units in the readme to be sure they're what you expect.

After the measurement column, we have the flag columns, which flag lines of data that may be problematic or flawed in some way. Depending on the kind of analysis you are doing, you may want to filter out measurements that display certain kinds of flags. For our purposes, we will be ignoring these flags and using all of the data.

The first kind of flag is the MFLAG, or measurement flag. Since this column is blank for us (notice the double commas), there were no problems recorded for this measurement. Possible values are things like "L" (temperature appears to be lagged with respect to reported hour of observation) and "P" (identified as "missing presumed zero"). 

The second flag column is the QFLAG, or quality flag. This flags various kinds of consistency checks that are run by GHCN on all entries. Here, our QFLAG reads "I," meaning "failed internal consistency check." 

Finally, we have the SFLAG, or source flag. Unlike the other two flags, this is not a measure of data reliability, but simply records which organization provided the data. In our case, the flag is set to 0, which means "U.S. Cooperative Summary of the Day (NCDC DSI-3200)." 

The last column of the dataset is the time of observation in HHMM format (military time). In this case, the temperature was recorded at 8:00 am.  

Remember, for a full list of elements, flags, and other information, refer to the [readme](https://www.ncei.noaa.gov/pub/data/ghcn/daily/readme.txt)!

## Converting the Data into a Pandas DataFrame

Now that we understand how the data is structured, we can use software to convert it into something a little more useful. Notice that the data is in what's called narrow form, or long form. If you look closely, you will see that you have multiple rows for each date, each one with a different entry in the element column to distinguish them from one another. For analysis and general readability, we typically prefer data to be in a wide form, with one row per date and columns corresponding to the different kinds of measurements taken on that date. We also would like our numbers to be consistent and readable; in this case I chose to convert all measurements to inches or degrees Fahrenheit and stored dates as Python Datetime objects. Here's a Python function to accomplish this with precipitation, snow, snow depth, and min/max temperature:

{% highlight python linenos %}
import pandas as pd

# Takes in a file path to a GHCN datafile and returns a nice Pandas Dataframe
def import_weather_data(path):
    df = pd.read_csv(path, 
                     names=['station', 'date', 'element', 'measurement', 'mflag', 'qflag', 'sflag', 'time'])
    
    df = df.drop(['station', 'mflag', 'qflag', 'sflag', 'time'], axis=1)

    df['date'] = pd.to_datetime(df['date'], format='%Y%m%d')

    df = df.pivot(index='date', columns='element', values='measurement')
    df = df.reset_index().rename(columns={'index': 'date'})
    # Stop the columns from being named "element"
    df.columns.name = None

    # Keep any of these columns that actually exist in the dataframe (stops the 
    # script from crashing if one of these measurements is missing)
    df = df[[column for column in df.columns if column in ['date', 'PRCP','SNOW','SNWD','TMAX','TMIN']]]

    # temperatures are recorded in tenths of a degree Celsius, so t/10 is in Celsius, 
    # and C*(9/5) + 32 gives Fahrenheit
    if 'TMAX' in df.columns:
        df['TMAX'] = (df['TMAX']/10)*(9/5) + 32
    if 'TMIN' in df.columns:
        df['TMIN'] = (df['TMIN']/10)*(9/5) + 32
    # mm/25.4 gives inches
    if 'SNOW' in df.columns:
        df['SNOW'] = df['SNOW']/25.4
    if 'SNWD' in df.columns:
        df['SNWD'] = df['SNWD']/25.4
    # PRCP is in tenths of a mm, so we divide by 10 and then convert to inches
    if 'PRCP' in df.columns:
        df['PRCP'] = (df['PRCP']/10)/25.4

    return df
{% endhighlight %}

Now we can run this function by giving it a file path to a GHCN data file: 

{% highlight python linenos %}
df = import_weather_data('E:/Coding/Python/Weather/USC00052223.csv')
print(df.head())
{% endhighlight %}

Make sure you change the file path to the location on your own computer where you saved the .csv file we extracted earlier. 

Our output should look something like this:

~~~
        date  PRCP  SNOW  SNWD   TMAX   TMIN
0 1997-11-01   0.0   0.0   0.0  62.96  37.04
1 1997-11-02   0.0   0.0   0.0  51.98  33.98
2 1997-11-03   0.0   0.0   0.0  53.06  28.04
3 1997-11-04   0.0   0.0   0.0  68.00  33.08
4 1997-11-05   0.0   0.0   0.0  66.92  32.00
~~~

Much better! Now we have the data in a readable and usable format.

**Visualizing the Data**

This dataframe is much easier to read than the original file, but it's still difficult to draw any conclusions from it. Let's look at a graph of the daily rainfall in Denver.

{% highlight python linenos %}
from matplotlib import pyplot as plt

plt.plot(df['date'], df['PRCP'])
plt.xlim(min(df['date']), max(df['date']))

plt.title("Denver, Colorado Precipitation History")
plt.xlabel("Date")
plt.ylabel("Rainfall in inches")

plt.show()
{% endhighlight %}

![Denver-daily-rainfall](https://Green9090.github.io/assets/img/Denver-daily-rainfall.png){: .mx-auto.d-block :}

This is nice if we are interested in picking out particular storms with a lot of rainfall. On the other hand, we might be interested in general rainfall trends, which are hard to read from this graph. Let's look at a graph of total rainfall by year instead. 

{% highlight python linenos %}
# Function to take a weather dataframe and return a yearly aggregated dataframe of the given element.
def yearly(df, element):
    df['year'] = df['date'].dt.year

    yearly_df = df.groupby('year')[element].sum().reset_index()
    yearly_df.columns = ['year', element]
    # Remove the first and last years since they don't contain a full year of data
    yearly_df = yearly_df.iloc[1:-1, :]

    return yearly_df


yearly_rainfall_df = yearly(df, 'PRCP')

plt.plot(yearly_rainfall_df['year'], yearly_rainfall_df['PRCP'])
plt.xlim(min(yearly_rainfall_df['year']), max(yearly_rainfall_df['year']))

plt.title("Denver, Colorado Yearly Precipitation History")
plt.xlabel("Year")
plt.ylabel("Total Annual Rainfall in Inches")

plt.show()
{% endhighlight %}

This code makes a new DataFrame with a column for year and a column for total rainfall in that year, then makes a new plot using this DataFrame. Note that we removed the first and last years from the data since they don't represent a full year of rainfall, and thus can't be compared meaningfully to the other years.

![Denver-annual-rainfall](https://Green9090.github.io/assets/img/Denver-annual-rainfall.png){: .mx-auto.d-block :}

While this smooths a lot of information out of our graph, it's also much easier to read! We can now quickly identify that 2015 was the rainiest year on record, and 2010 was the driest. 

**Extending This Code**

Now that you have the weather data you're looking for in a convenient Pandas dataframe, you can do whatever you like with it! The graphs we generated are simple examples to get you started. You can easily modify them to look at other aspects of the data. For example, changing every instance of 'PRCP' to 'SNOW' will graph snowfall, or changing it to 'TMAX' will graph maximum daily temperature. There are also plenty of more involved analyses you could do with this data. In the future, I plan to write some articles about statistical analysis of weather data. Stay tuned!