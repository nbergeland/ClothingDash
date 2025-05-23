#ClothingDash
## ClothingDash Overview
This document provides a high-level introduction to ClothingDash, a Plotly Dash web application that creates interactive analytics dashboards for clothing sales data. ClothingDash processes synthetic sales data for 1,000 clothing SKUs and presents five different visualization types through a reactive web interface.

## System Architecture
ClothingDash follows a simple three-component architecture where a CSV data source feeds into a Jupyter notebook-based Dash application that generates interactive web visualizations.

## Core Components
<ins>Component</ins>	        <ins>Component File</ins>	                  <ins>Purpose</ins>

Data Source	                       clothing_sales.csv	                         Raw sales data for 1,000 clothing SKUs
Application	                       app.ipynb	                                 Dash web application with data processing and visualization logic
Documentation	                     README.md	                                 System description and usage information
