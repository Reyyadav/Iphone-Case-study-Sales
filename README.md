# Project Report: Analysis of iPhone Sales Data
### 1. Data Preparation and Cleaning
### Objective: Prepare the iPhone dataset for analysis by renaming columns and handling missing values.

#### Column Renaming
To standardize column names, we replaced spaces with underscores. This helps in avoiding issues related to column name handling in further operations.


      df.columns = df.columns.str.replace(' ', '_')
      
### Handling Missing Values
Objective: Address missing Star Rating values in the dataset.

#### Method 1: Fill Missing Values with Global Average Rating
We calculated the average rating across all models and used it to fill missing values.


    average_rating = df['Star Rating'].mean(skipna=True)
    df['Star Rating'].fillna(average_rating, inplace=True)

#### Method 2: Fill Missing Values with Average Rating Based on RAM
To provide a more accurate imputation, we calculated average ratings by RAM configuration and filled missing values accordingly.

#### Method 1: Using Function
    def fill_missing_rating(row):
    if pd.isna(row['Star Rating']):
        return average_ratings_by_ram.get(row['RAM'], None)
    return row['Star Rating']

    df['Star Rating'] = df.apply(fill_missing_rating, axis=1)

#### Method 2: Using Merge
        average_ratings_by_ram = df.groupby('RAM')['Star Rating'].mean().reset_index()
        average_ratings_by_ram.rename(columns={'Star Rating': 'Average Rating'}, inplace=True)
        df = df.merge(average_ratings_by_ram, on='RAM', how='left')
        df['Star Rating'].fillna(df['Average Rating'], inplace=True)
        df.drop(columns=['Average Rating'], inplace=True)


### 2. Adding New Columns
Discount Percentage Calculation
To understand discount trends, we calculated the discount percentage based on the difference between MRP and sale price.


      df['Discount_Percentage'] = ((df['MRP'] - df['Sale Price']) / df['MRP']) * 100
      
### 3. Data Analysis
#### 3.1 Model with Highest Discount Percentage
To identify the model with the highest discount, we sorted the data based on the discount percentage.

      sorted_df = df.sort_values(by='Discount_Percentage', ascending=False)
      model_with_highest_discount = sorted_df.iloc[0]['Product Name']
      highest_discount_percentage = sorted_df.iloc[0]['Discount_Percentage']
      
### 3.2 Total Number of Models by Space Configuration
We computed the total number of models for each RAM configuration.


      space_configuration_counts = df['RAM'].value_counts()
      space_configuration_counts.columns = ['Space Configuration', 'Total Models']

      
### 3.3 Total Number of Models by Color
A custom function was used to extract color from product names, followed by counting the models by color.


        def extract_color(product_name):
            parts = product_name.split('(')
            if len(parts) > 1:
                color_part = parts[1].strip()
                if ',' in color_part:
                    color = color_part.split(',')[0].strip()
                    return color
            return None

        df['Color'] = df['Product Name'].apply(extract_color)
        color_counts = df['Color'].value_counts().reset_index()
        color_counts.columns = ['Color', 'Total Models']
        
### 3.4 Total Number of Models by iPhone Version
We extracted iPhone versions from product names and counted models by version.

      def extract_iphone_version(product_name):
          parts = product_name.split('iPhone')
          if len(parts) > 1:
              version = parts[1].strip().split(' ')[0]
              return 'iPhone ' + version
          return None

      df['iPhone Version'] = df['Product Name'].apply(extract_iphone_version)
      iphone_version_counts = df['iPhone Version'].value_counts().reset_index()
      iphone_version_counts.columns = ['iPhone Version', 'Total Models']

    
### 3.5 Top 5 Models with Highest Number of Reviews
To find the top models based on the number of reviews, we sorted the dataset and selected the top 5 entries.

      df_sorted = df.sort_values(by='Number Of Reviews', ascending=False)
      top_5_models = df_sorted.head(5)

### 3.6 Price Difference Between Highest and Lowest MRP
We calculated the price difference between the highest and lowest MRP values.


      highest_mrp = df['MRP'].max()
      lowest_mrp = df['MRP'].min()
      price_difference = highest_mrp - lowest_mrp

### 3.7 Total Number of Reviews for iPhone 11 and iPhone 12
We summarized the total reviews for iPhone 11 and iPhone 12 using two methods:

#### Method 1: Using str.contains

      df['iPhone Category'] = 'Other'
      df.loc[df['Product Name'].str.contains('iPhone 11', case=False), 'iPhone Category'] = 'iPhone 11'
      df.loc[df['Product Name'].str.contains('iPhone 12', case=False), 'iPhone Category'] = 'iPhone 12'
      category_reviews = df.groupby('iPhone Category')['Number Of Reviews'].sum().reset_index()
      result_df = category_reviews[category_reviews['iPhone Category'].isin(['iPhone 11', 'iPhone 12'])]
      
#### Method 2: Filtering and Summing

      iphone_11_reviews = df[df['Product Name'].str.contains('iPhone 11', case=False)]
      iphone_12_reviews = df[df['Product Name'].str.contains('iPhone 12', case=False)]
      total_reviews_11 = iphone_11_reviews['Number Of Reviews'].sum()
      total_reviews_12 = iphone_12_reviews['Number Of Reviews'].sum()
      result_df = pd.DataFrame({
          'iPhone Category': ['iPhone 11', 'iPhone 12'],
          'Total Reviews': [total_reviews_11, total_reviews_12]
      })
      
### 3.8 iPhone with the 3rd Highest MRP
To find the iPhone with the 3rd highest MRP, we sorted the data and selected the 3rd entry.

      sorted_df = df.sort_values(by='MRP', ascending=False).reset_index(drop=True)
      third_highest_mrp_iphone = sorted_df.loc[2]

### 3.9 Average MRP of iPhones Above 100,000
We calculated the average MRP for iPhones priced above 100,000.
      
      high_mrp_iphones = df[df['MRP'] > 100000]
      average_mrp = high_mrp_iphones['MRP'].mean()

### 3.10 iPhone with 128 GB Space and Highest Ratings to Review Ratio
To identify the iPhone with 128 GB space and the highest ratings-to-reviews ratio, we computed the ratio and selected the highest value.


      iphone_128gb = df[df['RAM'] == '128 GB']
      iphone_128gb['Ratings to Reviews Ratio'] = iphone_128gb['Number Of Ratings'] / iphone_128gb['Number Of Reviews']
      highest_ratio_iphone = iphone_128gb.loc[iphone_128gb['Ratings to Reviews Ratio'].idxmax()]

      
## Conclusion
This report provides a detailed analysis of iPhone sales data, including data cleaning, imputation, and various insights derived from the dataset. Key findings include the model with the highest discount, the distribution of models by space configuration and color, and average metrics for high-priced iPhones. This analysis serves as a foundation for further business intelligence and decision-making.
