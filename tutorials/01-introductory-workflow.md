# Python ChatGPT Tutorial

An introductory workflow to demonstrate use cases for developing Python code with ChatGPT or other large language models.

In this introduction, we will use the ChatGPT web interface to develop a Python workflow for analyzing the occurrence of baby names in the United States. Since ChatGPT will handle a lot of the syntax and code generation, we will focus on the following aspects of developing a workflow:

- Problem decomposition
- Debugging
- Explaining code
- Improving performance

We note that this is the dataset we have used for the previous two Python workshops, in which we coded everything by hand. Let's see how our progress in those two workshops compares with ChatGPT!

## Decomposition

Following recommendations and leading practices for using AI to generate code, we want our prompts to be specific to the desired results. Because of this, we can't really use a single, complex, multi-step prompt to generate an entire workflow. Instead, we need to think about the discrete steps in the process - we need to decompose the process - and write prompts that specify the context of the problem as well as the desired output or outcome.

Let's say we want to find out the most popular girl and boy names for the year 2010. The data are available from the following URL

~~~
https://raw.githubusercontent.com/unmrds/cc-R-RStudio/master/cc-R-Names/data/2010
~~~

The steps in our process are

1. Download the data
1. Convert the data to a Pandas dataframe
1. Group the data by sex
1. Sort the data by count of babies with each name
1. Get the top result for both girls and boys
1. As a bonus, we will plot the top 3 most popular names for each sex

Some of these steps will require multiple prompts. 

### Download the data

It is recommended to use comments or docstrings when engineering code prompts. Note the use of the ```import``` function in the prompt. This helps ChatGPT know which language to use in the response. Without it, you might get some perfectly good R code but no Python code!

~~~
"""
download the file https://raw.githubusercontent.com/unmrds/cc-R-RStudio/master/cc-R-Names/data/2010 and save
it as '2010.csv' in the current working directory
"""

import
~~~

Copy and paste the response into your Python environment. Try to run it - does it work?


#### Explain code

Code responses may or may not come back with comments. Depending on our level of experience with Python and different libraries, we may or may not be able to understand what the code is doing. Since understandin is necessary for documenting and troubleshooting, it can be useful to ask ChatGPT to explain code. Below is the code I got as a response to the first prompt, which we can use in a new prompt to request an explanation.

~~~
# Explain

import requests

url = "https://raw.githubusercontent.com/unmrds/cc-R-RStudio/master/cc-R-Names/data/2010"
file_name = "2010.csv"

response = requests.get(url)

if response.status_code == 200:
    with open(file_name, 'wb') as file:
        file.write(response.content)
    print(f"File '{file_name}' downloaded and saved successfully.")
else:
    print(f"Failed to download the file. Status code: {response.status_code}")
~~~

Does the explanation make sense? If your response was different, ask ChatGPT to explain it now.

### Read the data

The data have been saved to our computer, but are not yet available to us in a format that we can analyze. We want to use the Pandas library to read the data into a dataframe. Do we need to be specific about which library to use? Let's see!

Also, since we've been getting Python code so far, let's see what happens if we don't use a leading term like ```import```.

~~~
"""
Read the file 2010.csv into a table and print the first five rows.
"""
~~~

As above, copy and paste the response into your Python environment. Try to run it - does it work?

### Group data

Here we may run into our first hiccup, since the next step of our process requires us to group the data in the dataframe we just created. The current state of our local Python environment is maintained in the sense that we can refer to a data structure and the reference is understood. But since ChatGPT isn't executing code, there is state to be maintained. What happens if we try the following prompt?

~~~
"""
Group the data by sex.
"""
~~~

In the saved example, ChatGPT worked around this problem by coding the whole process of reading and grouping the data, though we note that the filename from the previous prompt was used. 

But with longer workflows we want to avoid redundant steps. To work around this, we can include the names of objects we're referring to in our prompts. Also keep in mind that we may need to refer to the grouped data in a later step, so be sure to include the name for the grouped data.

We note as well that ChatGPT is importing libraries for us, which is useful but we only need to import a library once. Let's include a direction in our prompt to not import pandas.

~~~
"""
Pandas has already been imported. Given a dataframe called 'df' with columns named 'name,' 'sex,' and 'count,' group the data by sex. Saved the grouped data to a variable named 'grouped_sex'.
"""
~~~

### Sort data

The data in this particular case have already been sorted by count, highest to lowest. This may not always be the case, so before we can determine what is the most popular name by sex we also want to sort the data. This makes our overall workflow more robust.

~~~
"""
Pandas has been imported. Given a grouped dataframe named 'grouped_sex,' sort the groups by the variable 'count.'
"""
~~~

My response uses a ```lambda``` function. I don't use those often, so I would like an explanation.

~~~
# Explain 

# Sort the groups within 'grouped_sex' by the variable 'count'
sorted_grouped_sex = grouped_sex.apply(lambda group: group.sort_values(by='count'))

# Now, 'sorted_grouped_sex' contains the groups sorted by 'count'
~~~

We've done a decent amount of data manipulation here. Let's try a sanity check.

~~~
"""
Inspect the grouped and sorted dataframe 'sorted_grouped_sex.'
"""
~~~

This looks okay, but we wanted to sort the data from highest to lowest count values. We can guess what we did wrong, but how do we fix it in our code?

#### Debug

Before we start debugging our code, it is useful to think about the different kinds of errors we can encounter when programming. Broadly, errors fall into two different types. Most common are syntax errors, which can be thought of as cases when some code doesn't follow the rules of the language. Try this example:

~~~
"""
for i in range(0, 10)
print(i)
"""
~~~

If we plug this prompt into ChatGPT, it very handily points out the syntax error and offers a correction. If we copy the corrected code into our environment, however, we may not get the result we expect when we run it. 

Without yet running the code, what kind of output do we expect from the following:

~~~
for i in range(10):
    print(i)
~~~

If we are unfamiliar with the way Python creates ranges, we might expect a printout of the numbers 1 - 10. That is not what we get when we run the code. What is happening instead?

This is an example of the other type of error that can occur when code runs - the syntax is correct - but the result is not correct or not what we expected. If we want to print a list of numbers from 1 - 10, we could just use a new prompt to do that. But in this case we also want to know why the code we have doesn't produce the result we expect. Let's try to debug this instead.

~~~
"""
# Why does the following code not print the numbers 1-10?

for i in range(10):
    print(i)
"""
~~~

The response should include not only some corrected code, but also an explanation as to why the previous code didn't work they way we thought it should.

Let's use this approach to debug our sort code.

~~~
"""
# Pandas has been imported. Given a dataframe grouped by 'sex' to be sorted by 'count,' 
# why does this code not sort 'count' from highest to lowest?

# Sort the groups within 'grouped_sex' by the variable 'count'
sorted_grouped_sex = grouped_sex.apply(lambda group: group.sort_values(by='count'))

# Now, 'sorted_grouped_sex' contains the groups sorted by 'count'
"""
~~~

Copy the response into your Python environment. Does it work as expected? Is the explanation provided by ChatGPT clear? If we inspect the result, it is good to keep in mind that Python will print out the first and last five rows of the data frame, not the first five rows of each group. So this looks pretty good!


### Filter data

Now that we have downloaded our data, read it into a dataframe, and grouped and sorted it we still need to filter the data based on a specific value. In this case, for both sexes we want to find the name that has the highest count. 

Note that we are using the original grouped data, and not the sorted data. This is because Python has a ```max()``` function that will return the highest value in a numeric column, without really requiring the data to be sorted. 

~~~
"""
Pandas has been imported. Given a Pandas GroupBy object named 'grouped_sex,' find the name for both 'F' and 'M' groups that has the highest count.
"""
~~~

We note that arriving at the correct prompt in this case took several iterations. This cycle of ```prompt -> test -> revise``` is a general description of an AI-assisted Python workflow.

### Plot popular names

We have so far reproduced the entire workflow from our previous Python workshops. As an added bonus, let's see what it takes to plot the top three names for both sexes.

Let's start with what may be the most natural of natural language prompts.

~~~
"""
Given a Pandas GroupBy object named 'grouped_sex,' plot the three most popular names by sex.
"""
~~~

Allowing that ChatGPT will not always generate the same code for the same prompt, the response received while developing this workflow included a function. 

~~~
import matplotlib.pyplot as plt

# Assuming you have a Pandas GroupBy object 'grouped_sex' and you want to plot the top 3 names by count for each sex

# Define a function to plot the most popular names by sex
def plot_top_names(grouped_df, sex):
    top_names = grouped_df.get_group(sex).nlargest(3, 'count')
    plt.figure(figsize=(10, 6))
    plt.bar(top_names['name'], top_names['count'])
    plt.title(f"Top 3 Names for {sex}")
    plt.xlabel("Name")
    plt.ylabel("Count")
    plt.xticks(rotation=45)
    plt.show()

# Plot the top 3 names for 'F' (female) and 'M' (male)
plot_top_names(grouped_sex, 'F')
plot_top_names(grouped_sex, 'M')
~~~

There's quite a bit happening here - as needed, we can ask ChatGPT to explain the code.

~~~
# Explain

import matplotlib.pyplot as plt

# Assuming you have a Pandas GroupBy object 'grouped_sex' and you want to plot the top 3 names by count for each sex

# Define a function to plot the most popular names by sex
def plot_top_names(grouped_df, sex):
    top_names = grouped_df.get_group(sex).nlargest(3, 'count')
    plt.figure(figsize=(10, 6))
    plt.bar(top_names['name'], top_names['count'])
    plt.title(f"Top 3 Names for {sex}")
    plt.xlabel("Name")
    plt.ylabel("Count")
    plt.xticks(rotation=45)
    plt.show()

# Plot the top 3 names for 'F' (female) and 'M' (male)
plot_top_names(grouped_sex, 'F')
plot_top_names(grouped_sex, 'M')
~~~


#### Improve performance

We have a workflow! We would download a new dataset and get the top names by sex and plot the top three names by sex, so long as additional datasets have the same structure as our 2010 data. 

But we've already noted some redundancy in our process - having to remind ChatGPT that pandas has been imported, reference existing objects and data structures by name, etc. Let's now see if we can use ChatGPT to improve on our workflow.

The next prompt is something of a hail-mary.

~~~
"""
Improve the performance of the following code. The code should download a data file, save it to the local computer, open it as a dataframe in Pandas, find the most popular name by sex, and plot the most popular three names by sex.
"""

import requests

url = "https://raw.githubusercontent.com/unmrds/cc-R-RStudio/master/cc-R-Names/data/2010"
file_name = "2010.csv"

response = requests.get(url)

if response.status_code == 200:
    with open(file_name, 'wb') as file:
        file.write(response.content)
    print(f"File '{file_name}' downloaded and saved successfully.")
else:
    print(f"Failed to download the file. Status code: {response.status_code}")

import pandas as pd

file_name = "2010.csv"

# Read the CSV file into a DataFrame
df = pd.read_csv(file_name)

# Print the first five rows of the DataFrame
print(df.head(5))

grouped_sex = df.groupby('sex')

# Find the names with the highest count for 'F' and 'M' groups
highest_count_names = grouped_sex.apply(lambda group: group[group['count'] == group['count'].max()]['name'])

import matplotlib.pyplot as plt

# Assuming you have a Pandas GroupBy object 'grouped_sex' and you want to plot the top 3 names by count for each sex

# Define a function to plot the most popular names by sex
def plot_top_names(grouped_df, sex):
    top_names = grouped_df.get_group(sex).nlargest(3, 'count')
    plt.figure(figsize=(10, 6))
    plt.bar(top_names['name'], top_names['count'])
    plt.title(f"Top 3 Names for {sex}")
    plt.xlabel("Name")
    plt.ylabel("Count")
    plt.xticks(rotation=45)
    plt.show()

# Plot the top 3 names for 'F' (female) and 'M' (male)
plot_top_names(grouped_sex, 'F')
plot_top_names(grouped_sex, 'M')
~~~

Compare the response with the original code. Is there much of a difference? Does the explanation of improvements make sense? Copy and run the code to see if it works!

The response returned while we were developing this workshop worked. Let's see if we can build on this improvement to

1. Download and unzip eleven years' worth of data
1. Find the most popular name for both sexes for each year
1. Plot the three most popular names for both sexes, for each year

We still need to tell ChatGPT the URL to the zipped data.

https://github.com/unmrds/cc-R-RStudio/raw/master/cc-R-Names/data.zip

~~~
"""
Improve the following code to download and unzip data from the url https://github.com/unmrds/cc-R-RStudio/raw/master/cc-R-Names/data.zip. For each file in the unzipped data, open the file as a dataframe in Pandas, find the most popular name by sex, and plot the most popular three names by sex.
"""

import requests
import pandas as pd
import matplotlib.pyplot as plt

# Define the URL and file name
url = "https://raw.githubusercontent.com/unmrds/cc-R-RStudio/master/cc-R-Names/data/2010"
file_name = "2010.csv"

# Download the file and save it locally
response = requests.get(url)
if response.status_code == 200:
    with open(file_name, 'wb') as file:
        file.write(response.content)
        print(f"File '{file_name}' downloaded and saved successfully.")
else:
    print(f"Failed to download the file. Status code: {response.status_code}")

# Read the CSV file into a DataFrame
df = pd.read_csv(file_name)

# Group the data by 'sex'
grouped_sex = df.groupby('sex')

# Find the most popular name by sex
most_popular_names = grouped_sex['count'].idxmax().apply(lambda x: df['name'][x])

# Print the most popular name by sex
print(most_popular_names)

# Plot the top 3 names by count for each sex
def plot_top_names(grouped_df, sex):
    top_names = grouped_df.get_group(sex).nlargest(3, 'count')
    plt.figure(figsize=(10, 6))
    plt.bar(top_names['name'], top_names['count'])
    plt.title(f"Top 3 Names for {sex}")
    plt.xlabel("Name")
    plt.ylabel("Count")
    plt.xticks(rotation=45)
    plt.show()

# Plot the top 3 names for 'F' (female) and 'M' (male)
plot_top_names(grouped_sex, 'F')
plot_top_names(grouped_sex, 'M')
~~~

Copy and test the response in your environment. Does it work?