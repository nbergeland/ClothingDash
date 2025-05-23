# <ins>ClothingDash</ins>
## <ins>ClothingDash Overview</ins>
This document provides a high-level introduction to ClothingDash, a Plotly Dash web application that creates interactive analytics dashboards for clothing sales data. ClothingDash processes synthetic sales data for 1,000 clothing SKUs and presents five different visualization types through a reactive web interface.

## <ins>System Architecture</ins>
ClothingDash follows a simple three-component architecture where a CSV data source feeds into a Jupyter notebook-based Dash application that generates interactive web visualizations.

## <ins>Core Components</ins>
![Screenshot](CC.png)


## <ins>High-Level System Flow</ins>
![Screenshot](HLSF.png)
This diagram shows how the clothing_sales.csv file flows through pandas.read_csv() into the main callback function update_charts(), which processes user inputs from dcc.Dropdown and dcc.DatePickerRange components to generate charts via plotly.express and serve them through the app.run() web server.Data 

## <ins>Processing Pipeline</ins>
ClothingDash implements a reactive data processing pipeline where user filter selections trigger data transformations and chart regeneration across all visualization types simultaneously.
![Screenshot](DPP.png)
The pipeline begins with pd.read_csv() loading the CSV data with date parsing. User interactions with category_dropdown.value and date picker components trigger boolean indexing operations that create a filtered DataFrame (dff). This filtered data undergoes groupby() aggregation before being passed to individual plotly.express chart generation functions.

## <ins>Application Structure</ins>
The app.ipynb file contains the complete Dash application implementation using a single-file architecture pattern common in data science workflows.

### <ins>Core Application Components</ins>	

![Screenshot](CAC.png)

## <ins>Visualization Types</ins>:	
ClothingDash generates five distinct chart types from the same filtered dataset, providing multiple analytical perspectives on clothing sales patterns.
![Screenshot](VT.png)
Each chart type uses different groupby() operations and aggregation functions on the filtered DataFrame (dff) to create specialized views of the sales data. The reactive callback system ensures all five charts update simultaneously when filter values change.

## <ins>Clothing Sales Dataset & Dashboard Creation</ins>	

### <ins>Dataset Generation Overview</ins>
This document outlines the creation of a synthetic clothing sales dataset and the interactive Plotly Dash application built atop it. The data simulates monthly sales for 1,000 unique SKUs over a 10-year period, each annotated with detailed product attributes and revenue calculations.

#### <ins>Data Source & Schema</ins>
![Screenshot](DSS.png)
#### <ins>Dataset Creation Pipeline</ins>
1.	Attribute Sampling
•	1,000 SKUs generated as zero-padded identifiers.
•	Categories assigned randomly from a predefined list; subcategories, colors, sizes, and prices sampled accordingly.
2.	Temporal Expansion
•	Monthly index created spanning May 2015 through April 2025 (120 periods).
•	Cartesian product of SKUs × months yields 120,000 rows.
3.	Sales Simulation & Revenue
•	Poisson distributions parameterized per category model units sold.
•	Revenue computed per row, rounded to two decimal places.
4.	Output
•	Final CSV exports:
•	clothing_sales_10yr.csv (no product name)
•	clothing_sales_10yr_with_name.csv (includes product_name column)

#### <ins>Prompting & Interactive Dashboard</ins>
Data Requests
	•	Generated 2-year synthetic dataset for 1,000 SKUs
	•	Extended to 10-year monthly data
	•	Added product_name field on demand
Dashboard Iteration
	•	Built Plotly Dash app with filters and five chart types
	•	Enhanced visuals with 3D charts per feedback
 App Structure
	•	Entry: app.py (or use app.ipynb + JupyterDash)
	•	Layout: Category dropdown + date‐picker range
	•	Interactivity: One @app.callback updates all charts
#### <ins>Visualization Types</ins>
1.	Line Chart – Revenue over time by category
2.	Bar Chart – Total revenue per category
3.	Scatter Plot – Price vs. units sold (with optional 3D variant)
4.	Heatmap – Monthly revenue intensity by category
5.	Pie Chart – Revenue distribution by category

#### <ins>Prompt for Dataset Creation</ins>
"Create a synthetic 10-year (2015–2025) monthly sales CSV for 1,000 clothing SKUs—including sku, date, category, subcategory, product_name, color, size, price, units_sold, and revenue."

# <ins>Code</ins>:	

````
import dash
from dash import dcc, html, Input, Output
import pandas as pd
import plotly.express as px

# Load dataset
df = pd.read_csv('clothing_sales.csv', parse_dates=['date'])
````
````
# Initialize the Dash app
app = dash.Dash(__name__)
app.title = "Clothing Sales Dashboard"

# Unique categories for dropdown
categories = df['category'].unique()

# Layout
app.layout = html.Div(style={'fontFamily': 'Arial, sans-serif', 'margin': '20px'}, children=[
    html.H1("Clothing Sales Dashboard", style={'textAlign': 'center'}),
    
    # Filters
    html.Div([
        html.Div([
            html.Label("Select Category"),
            dcc.Dropdown(
                id='category-dropdown',
                options=[{'label': cat, 'value': cat} for cat in categories],
                value=list(categories),
                multi=True
            )
        ], style={'width': '48%', 'display': 'inline-block'}),
        
        html.Div([
            html.Label("Select Date Range"),
            dcc.DatePickerRange(
                id='date-picker',
                start_date=df['date'].min(),
                end_date=df['date'].max(),
                display_format='YYYY-MM'
            )
        ], style={'width': '48%', 'display': 'inline-block', 'float': 'right'}),
    ], style={'padding': '10px', 'backgroundColor': '#f9f9f9', 'borderRadius': '5px'}),
    
    # Charts
    html.Div([
        dcc.Graph(id='revenue-line-chart'),
        dcc.Graph(id='top-categories-bar'),
        dcc.Graph(id='price-units-scatter'),
        dcc.Graph(id='sales-heatmap'),
        dcc.Graph(id='category-pie-chart'),
    ], style={'marginTop': '20px'})
])

# Callbacks to update all five charts
@app.callback(
    [
        Output('revenue-line-chart', 'figure'),
        Output('top-categories-bar', 'figure'),
        Output('price-units-scatter', 'figure'),
        Output('sales-heatmap', 'figure'),
        Output('category-pie-chart', 'figure'),
    ],
    [
        Input('category-dropdown', 'value'),
        Input('date-picker', 'start_date'),
        Input('date-picker', 'end_date')
    ]
)
def update_charts(selected_categories, start_date, end_date):
    # Filter data
    mask = (
        df['category'].isin(selected_categories) &
        (df['date'] >= pd.to_datetime(start_date)) &
        (df['date'] <= pd.to_datetime(end_date))
    )
    dff = df.loc[mask]

    # 1. Line chart: Total Revenue Over Time by Category
    revenue_time_cat = (
        dff.groupby(['date', 'category'])['revenue']
           .sum()
           .reset_index()
    )
    fig1 = px.line(
        revenue_time_cat,
        x='date', y='revenue', color='category',
        title='Revenue Over Time by Category',
        labels={'date':'Date','revenue':'Revenue','category':'Category'}
    )

    # 2. Bar chart: Revenue by Category
    cat_revenue = (
        dff.groupby('category')['revenue']
           .sum()
           .reset_index()
           .sort_values('revenue', ascending=False)
    )
    fig2 = px.bar(
        cat_revenue,
        x='category', y='revenue',
        title='Total Revenue by Category',
        labels={'revenue':'Revenue','category':'Category','product_name':'Product Name'}
    )

    # 3. Scatter: Price vs. Units Sold
    sample = dff.sample(min(len(dff), 500))
    fig3 = px.scatter(
        sample,
        x='price', y='units_sold',
        color='category',
        title='Price vs. Units Sold',
        hover_data=['sku'],
        labels={'price':'Price','units_sold':'Units Sold','category':'Category'}
    )

    # 4. Heatmap: Monthly Revenue by Category
    heatmap_df = (
        dff.groupby([dff['date'].dt.to_period('M').dt.to_timestamp(), 'category'])['revenue']
           .sum()
           .reset_index()
           .pivot(index='date', columns='category', values='revenue')
           .fillna(0)
    )
    fig4 = px.imshow(
        heatmap_df.T,
        labels={'x':'Month', 'y':'Category', 'color':'Revenue'},
        x=heatmap_df.index.astype(str),
        y=heatmap_df.columns,
        title='Monthly Revenue Heatmap'
    )
    fig4.update_xaxes(tickangle=45)

    # 5. Pie chart: Revenue Distribution by Category
    total_rev = cat_revenue
    fig5 = px.pie(
        total_rev,
        names='category',
        values='revenue',
        title='Revenue Distribution by Category'
    )

    return fig1, fig2, fig3, fig4, fig5

if __name__ == '__main__':
    app.run(debug=True)
````
# <ins>Static Dash Preview</ins>:

<img width="1262" alt="ROT" src="https://github.com/user-attachments/assets/645d5a78-dd35-419d-9312-f661b0b17cc0" />
<img width="1277" alt="PUS" src="https://github.com/user-attachments/assets/822a7cf9-5bf3-4975-b9f7-d69f665a6355" />
<img width="882" alt="PIE" src="https://github.com/user-attachments/assets/d6f1bd7f-c909-43e2-b1a6-7b7f0d339aff" />
<img width="1033" alt="Heat" src="https://github.com/user-attachments/assets/ecb8a987-42ac-4920-8a1c-9207fb70df31" />
<img width="1048" alt="TRC" src="https://github.com/user-attachments/assets/17e88372-c822-4edb-954d-1b196c057458" />
