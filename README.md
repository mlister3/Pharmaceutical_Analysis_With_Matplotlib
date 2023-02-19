# Pharmaceutical_Analysis_With_Matplotlib

### Use Matplotlib module to visualize trial data and perfrom analysis.

## Background
> You've just joined Pymaceuticals, Inc., a new pharmaceutical company that specializes in anti-cancer medications. Recently, it began screening for potential treatments for squamous cell carcinoma (SCC), a commonly occurring form of skin cancer.

> As a senior data analyst at the company, you've been given access to the complete data from their most recent animal study. In this study, 249 mice who were identified with SCC tumors received treatment with a range of drug regimens. Over the course of 45 days, tumor development was observed and measured. The purpose of this study was to compare the performance of Pymaceuticals’ drug of interest, Capomulin, against the other treatment regimens.

> The executive team has tasked you with generating all of the tables and figures needed for the technical report of the clinical study. They have also asked you for a top-level summary of the study results.

## Instructions
> This assignment is broken down into the following tasks:

1. Prepare the data.

2. Generate summary statistics.

3. Create bar charts and pie charts.

4. Calculate quartiles, find outliers, and create a box plot.

5. Create a line plot and a scatter plot.

6. Calculate correlation and regression.

7. Submit your final analysis.

- - -

## Prepare the Data
1. Run the provided package dependency and data imports, and then merge the mouse_metadata and study_results DataFrames into a single DataFrame.
```python
    # Dependencies and Setup
    import matplotlib.pyplot as plt
    import pandas as pd
    import scipy.stats as st
    from scipy.stats import linregress

    # Study data files
    mouse_metadata_path = "data/Mouse_metadata.csv"
    study_results_path = "data/Study_results.csv"

    # Read the mouse data and the study results
    mouse_metadata = pd.read_csv(mouse_metadata_path)
    study_results = pd.read_csv(study_results_path)``

    # Combine the data into a single dataset
    Combined_Pharm_Data = mouse_metadata.merge(study_results, on="Mouse ID")
```

2. Display the number of unique mice IDs in the data, and then check for any mouse ID with duplicate time points. Display the data associated with that mouse ID, and then create a new DataFrame where this data is removed. Use this cleaned DataFrame for the remaining steps.

3. Display the updated number of unique mice IDs.
> See [pymaceuticals_plot_analysis.ipynb ](./pymaceuticals_plot_analysis.ipynb) for points 2 & 3. 

## Generate Summary Statistics
> Create a DataFrame of summary statistics. Remember, there is more than one method to produce the results you're after, so the method you use is less important than the result.

### Your summary statistics should include:

- A row for each drug regimen. These regimen names should be contained in the index column.

- A column for each of the following statistics: mean, median, variance, standard deviation, and SEM of the tumor volume.

```python
    Drug_Groupby_df = CCMData.groupby("Drug Regimen")

    # Use groupby and summary statistical methods to calculate the following properties of each drug regimen: 
    # mean, median, variance, standard deviation, and SEM of the tumor volume.
    mean = Drug_Groupby_df["Tumor Volume (mm3)"].mean()
    median = Drug_Groupby_df["Tumor Volume (mm3)"].median()
    variance = Drug_Groupby_df["Tumor Volume (mm3)"].var()
    standard_deviation = Drug_Groupby_df["Tumor Volume (mm3)"].std()
    SEM = Drug_Groupby_df["Tumor Volume (mm3)"].sem()

    # Assemble the resulting series into a single summary DataFrame.
    summary_statistics = pd.DataFrame({"Mean Tumor Volume": mean, "Median Tumor Volume": median, "Tumor Volume Variance": variance, "Tumor Volume Std. Dev.": standard_deviation, "Tumor Volume Std. Err.": SEM})
```

## Create Bar Charts and Pie Charts
1. Generate two bar charts. Both charts should be identical and show the total number of time points for all mice tested for each drug regimen throughout the study.

    - Create the first bar chart with the Pandas DataFrame.plot() method.

    - Create the second bar chart with Matplotlib's pyplot methods.

2. Generate two pie charts. Both charts should be identical and show the distribution of female versus male mice in the study.

    - Create the first pie chart with the Pandas DataFrame.plot() method.

    - Create the second pie chart with Matplotlib's pyplot methods.

> See [pymaceuticals_plot_analysis.ipynb ](./pymaceuticals_plot_analysis.ipynb) for charts. 

## Calculate Quartiles, Find Outliers, and Create a Box Plot
1. Calculate the final tumor volume of each mouse across four of the most promising treatment regimens: Capomulin, Ramicane, Infubinol, and Ceftamin. Then, calculate the quartiles and IQR, and determine if there are any potential outliers across all four treatment regimens. Use the following substeps:

    - Create a grouped DataFrame that shows the last (greatest) time point for each mouse. Merge this grouped DataFrame with the original cleaned DataFrame.

    - Create a list that holds the treatment names as well as a second, empty list to hold the tumor volume data.

    - Loop through each drug in the treatment list, locating the rows in the merged DataFrame that correspond to each treatment. Append the resulting final tumor volumes for each drug to the empty list.

    - Determine outliers by using the upper and lower bounds, and then print the results.

2. Using Matplotlib, generate a box plot that shows the distribution of the final tumor volume for all the mice in each treatment group. Highlight any potential outliers in the plot by changing their color and style.

```python
    Select_Regimens = ["Capomulin", "Ramicane", "Infubinol", "Ceftamin"]
    Filtered_Reg = CCMData[CCMData["Drug Regimen"].isin(Select_Regimens)]

    # Start by getting the last (greatest) timepoint for each mouse
    last_timepoint = Filtered_Reg.groupby(["Mouse ID"])["Timepoint"].max()
    last_timepoint = last_timepoint.reset_index()

    # Merge this group df with the original DataFrame to get the tumor volume at the last timepoint
    final_tumor_volume = pd.merge(last_timepoint, Filtered_Reg, on=["Mouse ID", "Timepoint"], how="left")
    final_tumor_volume = final_tumor_volume[["Mouse ID", "Drug Regimen", "Tumor Volume (mm3)"]]
    
    # Create empty list to fill with tumor vol data (for plotting)
    tumor_vol_data = []

    # Put treatments into a list for for loop (and later for plot labels)
    for drug in Select_Regimens:
        # Locate the rows which contain mice on each drug and get the tumor volumes
        drug_df = final_tumor_volume[final_tumor_volume["Drug Regimen"] == drug]
        tumor_volumes = drug_df["Tumor Volume (mm3)"]
    
        # Calculate the IQR and quantitatively determine if there are any potential outliers. 
        quartiles = tumor_volumes.quantile([.25,.5,.75])
        lowerq = quartiles[0.25]
        upperq = quartiles[0.75]
        iqr = upperq-lowerq 
    
        # Determine outliers using upper and lower bounds
        lower_bound = lowerq - (1.5*iqr)
        upper_bound = upperq + (1.5*iqr)
    
        # add subset
        outliers = tumor_volumes[(tumor_volumes < lower_bound) | (tumor_volumes > upper_bound)]
        print(f"{drug}: Lower bound = {lower_bound:.2f}, Upper bound = {upper_bound:.2f}, Outliers = {outliers.tolist()}")
    
    # Generate a box plot that shows the distrubution of the tumor volume for each treatment group.
    Drugs_plot = [final_tumor_volume.loc[final_tumor_volume["Drug Regimen"] == "Capomulin"]["Tumor Volume (mm3)"],
                final_tumor_volume.loc[final_tumor_volume["Drug Regimen"] == "Ramicane"]["Tumor Volume (mm3)"],
                final_tumor_volume.loc[final_tumor_volume["Drug Regimen"] == "Infubinol"]["Tumor Volume (mm3)"],
                final_tumor_volume.loc[final_tumor_volume["Drug Regimen"] == "Ceftamin"]["Tumor Volume (mm3)"]]

    Window = plt.figure(figsize=(10, 5))
    Chart = Window.add_subplot(111)

    box_plot = Chart.boxplot(data_to_plot, labels=Select_Regimens)

    for flier in box_plot['fliers']:
        flier.set(marker='o', color='red')
    
    Chart.set_ylabel("Final Tumor Volume (mm3)")
```

![Box Plot](Chart_Out/Screenshot%202023-02-19%20124059.png)

## Create a Line Plot and a Scatter Plot
1. Select a mouse that was treated with Capomulin, and generate a line plot of tumor volume versus time point for that mouse.

2. Generate a scatter plot of tumor volume versus mouse weight for the Capomulin treatment regimen.

## Calculate Correlation and Regression
1. Calculate the correlation coefficient and linear regression model between mouse weight and average tumor volume for the Capomulin treatment.

2. Plot the linear regression model on top of the previous scatter plot.

- - -

## References

Data generated by MockarooLinks, LLC (2022). Realistic Data Generator.

- - -

© 2023 edX Boot Camps LLC