# Carbon Majors EDA

### The Big Picture: Historical Trend

```python
# Global Emissions Trend (1862 - 2023)
yearly_emissions = df.groupby('year')['total_emissions_MtCO2e'].sum().reset_index()

# Plot
plt.figure(figsize=(12, 6))
plt.plot(yearly_emissions['year'], yearly_emissions['total_emissions_MtCO2e'], marker='o')
plt.title("Global Emissions Trend (1862-2023)")
plt.xlabel("Year")
plt.ylabel("Total Emissions (MtCO2e)")
plt.grid(True)
plt.show()
```
![1](https://github.com/user-attachments/assets/860a97cb-46f9-4bb0-a103-10e5cd2bd8bf)

### Major Historical Contributors

```python
total_emissions_overall = df['total_emissions_MtCO2e'].sum()
emissions_by_commodity = df.groupby('commodity')['total_emissions_MtCO2e'].sum()
commodity_contribution = (emissions_by_commodity / total_emissions_overall * 100).sort_values(ascending=False)
print(commodity_contribution)

# Plot
plt.figure(figsize=(10, 6))
commodity_contribution.plot(kind='bar')
plt.title('Overall % Contribution to Emissions by Commodity')
plt.xlabel('Commodity')
plt.ylabel('Percentage (%)')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```
```plaintext
commodity
Oil & NGL               39.007241
Bituminous Coal         20.554965
Natural Gas             19.352282
Metallurgical Coal       5.515879
Thermal Coal             3.835309
Sub-Bituminous Coal      3.578223
Anthracite Coal          3.141895
Lignite Coal             2.746686
Cement                   2.267521
Name: total_emissions_MtCO2e, dtype: float64
```
![2](https://github.com/user-attachments/assets/08abcf7a-17fc-4487-b829-2e61b991c9d2)

```python
entity_emissions = df.groupby('parent_type')['total_emissions_MtCO2e'].sum()
num_slices = len(entity_emissions)

# 'colorblind' palette
palette_colors = sns.color_palette('colorblind', n_colors=num_slices)

# Plot
plt.figure(figsize=(8, 6))
entity_emissions.plot(kind='pie', autopct='%1.1f%%', colors=palette_colors)
plt.title("Emissions by Entity Type")
plt.ylabel("") 
plt.show()
```
![3](https://github.com/user-attachments/assets/25933450-5ab0-4b90-90d7-37dafec63c80)

```python
top_emitters = df.groupby('parent_entity')['total_emissions_MtCO2e'].sum() \
                .nlargest(10).reset_index()

# Plot
plt.figure(figsize=(10, 6))
plt.barh(top_emitters['parent_entity'], top_emitters['total_emissions_MtCO2e'], color='red')
plt.title("Top 10 Emitting Companies")
plt.xlabel("Total Emissions (MtCO2e)")
plt.gca().invert_yaxis() 
plt.show()
```
![4](https://github.com/user-attachments/assets/42853143-3a2c-425e-8e24-7c1837271e99)

### Understanding the data (Record counts)

```python
record_counts = df.groupby(['commodity', 'production_unit']).size().unstack(fill_value=0)

# bar plot
record_counts.plot(kind='bar', figsize=(14, 7), width=0.8)

plt.title('Number of Records per Commodity and Production Unit')
plt.xlabel('Commodity')
plt.ylabel('Number of Records')
plt.xticks(rotation=45, ha='right')
plt.legend(title='Production Unit')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
```
![5](https://github.com/user-attachments/assets/d4f15f0c-ceda-4b54-85a1-42e0c93e829d)

### Distribution Insights (Emissions Records)
```python
df['decade'] = (df['year'] // 10) * 10 # Ensure 'decade' column exists

plt.figure(figsize=(12, 7))

# Create median line
median_line_properties = {
    'color': 'darkorange',  
    'linewidth': 0.5   
}

sns.boxplot(data=df, x='decade', y='total_emissions_MtCO2e', palette='viridis', medianprops = median_line_properties)

plt.title('Distribution of Emissions Records (MtCO2e) by Decade')
plt.xlabel('Decade Starting Year')
plt.ylabel('Total Emissions (MtCO2e) per Record')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```
![6](https://github.com/user-attachments/assets/5094ac45-3279-475e-a69a-cd6b17657034)

```python
parent_type = df.groupby('parent_type')['total_emissions_MtCO2e'].median().sort_values(ascending=False).index

median_line = {
    'color': 'darkorange', 
    'linewidth': 2
}

# box plot
plt.figure(figsize=(10, 7)) # Adjust figure size if needed

sns.boxplot(
    data=df,
    x='parent_type',                   
    y='total_emissions_MtCO2e',     
    order= parent_type,  
    palette='magma',               
    medianprops=median_line
)

plt.title('Distribution of Emissions Records (MtCO2e) by Parent Type')
plt.xlabel('Parent Type')
plt.ylabel('Total Emissions (MtCO2e) per Record')
plt.xticks(rotation=0) 
plt.tight_layout()
plt.show()
```
![7](https://github.com/user-attachments/assets/2c0e8a48-4089-4081-b843-d4c75765d672)

```python
plt.figure(figsize=(14, 7))
sns.boxplot(x='commodity', y='total_emissions_MtCO2e', data=df)
plt.title('Distribution of Total Emissions (MtCO2e) by Commodity')
plt.xlabel('Commodity')
plt.ylabel('Total Emissions (MtCO2e)')
plt.xticks(rotation=45, ha='right') 
plt.tight_layout()
plt.show()
```
![8](https://github.com/user-attachments/assets/6f00fe62-713e-4c13-9987-008a89ba7df0)

### Investigating Specifci Observation (Records > 1500 MtCO2)
```python
df_nation_state = df[df['parent_type'] == 'Nation State'].copy() # Use .copy() to avoid SettingWithCopyWarning if you modify later


# Comparison 
high_emission_threshold = 1000
nation_state_high_emitters = df_nation_state[df_nation_state['total_emissions_MtCO2e'] > high_emission_threshold]

if not nation_state_high_emitters.empty:
    print(f"\nCommodity breakdown for Nation State records with emissions > {high_emission_threshold} MtCO2e:")
    print(nation_state_high_emitters['commodity'].value_counts())

    # Visualize this breakdown
    plt.figure(figsize=(10, 5))
    nation_state_high_emitters['commodity'].value_counts().plot(kind='bar', color=sns.color_palette('tab10'))
    plt.title(f'Commodities in Nation State Records > {high_emission_threshold} MtCO2e')
    plt.ylabel('Number of Records')
    plt.xlabel('Commodity')
    plt.xticks(rotation=45, ha='right')
    plt.tight_layout()
    plt.show()
else:
    print(f"\nNo records found")
```
```plaintext
Commodity breakdown for Nation State records with emissions > 1000 MtCO2e:
commodity
Bituminous Coal       52
Oil & NGL             17
Cement                13
Natural Gas           10
Metallurgical Coal     1
```
![9](https://github.com/user-attachments/assets/6409d741-da8f-4a1c-8f44-6ad41bbd45ef)

### Emission Composition: Operational vs Product Use & Methane Focus

```python
# Violin Chart
plt.figure(figsize=(10, 6))
sns.violinplot(data=df_ratios.dropna(subset=['op_emissions_ratio']),
               x='parent_type',
               y='op_emissions_ratio',
               order=median_order_filtered, 
               palette='viridis',
               inner='quartiles') 
plt.title('Distribution of Operational Emissions Ratio by Parent Type')
plt.xlabel('Parent Type')
plt.ylabel('Operational Emissions / Total Emissions')
plt.ylim(0, max(1, df_ratios['op_emissions_ratio'].max() * 1.1)) 
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
```
![10](https://github.com/user-attachments/assets/46d681dc-083c-4a6d-8bda-84fb101f81cb)

```python
total_methane_by_commodity = df.groupby('commodity')['fugitive_methane_emissions_MtCO2e'].sum().sort_values(ascending=False)

# by top commodities
plt.figure(figsize=(12, 6))
total_methane_by_commodity.head(10).plot(kind='bar', color=sns.color_palette('YlGnBu', 10)) # Top 10
plt.title('Top 10 Commodities by Total Cumulative Fugitive Methane Emissions')
plt.ylabel('Total Fugitive Methane (MtCO2e)')
plt.xlabel('Commodity')
plt.xticks(rotation=45, ha='right')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()

total_methane_by_parent_type = df.groupby('parent_type')['fugitive_methane_emissions_MtCO2e'].sum().sort_values(ascending=False)

# by parent type
plt.figure(figsize=(8, 5))
total_methane_by_parent_type.plot(kind='bar', color=sns.color_palette('YlGnBu', 3))
plt.title('Total Cumulative Fugitive Methane Emissions by Parent Type')
plt.ylabel('Total Fugitive Methane (MtCO2e)')
plt.xlabel('Parent Type')
plt.xticks(rotation=0)
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
```
![11](https://github.com/user-attachments/assets/ea0845b6-d098-4410-8222-42dccacbc208)
![12](https://github.com/user-attachments/assets/6bce8c12-1fad-45a9-85f7-9dadd6391c75)

```python
component_cols = [
    'flaring_emissions_MtCO2', 'venting_emissions_MtCO2',
    'own_fuel_use_emissions_MtCO2', 'fugitive_methane_emissions_MtCO2e',
    'total_operational_emissions_MtCO2e'
]
commodity_totals = df.groupby('commodity')[component_cols].sum(numeric_only=True)
commodity_totals_valid = commodity_totals[commodity_totals['total_operational_emissions_MtCO2e'] > 0]

component_names = [
    'flaring_emissions_MtCO2', 'venting_emissions_MtCO2',
    'own_fuel_use_emissions_MtCO2', 'fugitive_methane_emissions_MtCO2e'
]
perc_cols = []
for col in component_names:
    perc_col_name = f'perc_{col.split("_")[0]}'
    commodity_totals_valid[perc_col_name] = (
        commodity_totals_valid[col] / commodity_totals_valid['total_operational_emissions_MtCO2e']
    ) * 100
    perc_cols.append(perc_col_name)

commodities_to_plot = ['Oil & NGL', 'Natural Gas', 'Bituminous Coal', 'Lignite Coal', 'Cement']

plot_data = commodity_totals_valid.loc[commodity_totals_valid.index.intersection(commodities_to_plot), perc_cols]

plt.figure(figsize=(12, 8))

plot_data.plot(kind='bar', stacked=True, width=0.8, legend=False, ax=plt.gca()) 

plt.title('Composition of Total Operational Emissions for Key Commodities')
plt.xlabel('Commodity')
plt.ylabel('Contribution to Total Operational Emissions (%)')
plt.xticks(rotation=0)
# Create legend with slightly cleaner labels
plt.legend(title='Emission Source', labels=[col.replace('perc_','').capitalize() for col in plot_data.columns], bbox_to_anchor=(1.02, 1), loc='upper left')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout(rect=[0, 0, 0.85, 1]) 
plt.show()
```
![13](https://github.com/user-attachments/assets/e9d21d35-61c3-4977-b8cb-14805d1dedf0)

### Trends of key Players Over Time

```python
total_emissions_by_entity = df.groupby('parent_entity')['total_emissions_MtCO2e'].sum()
top_5_entities = total_emissions_by_entity.nlargest(5).index.tolist()


df_top_entities = df[df['parent_entity'].isin(top_5_entities)].copy()

emissions_top_entities_yearly = df_top_entities.groupby(['year', 'parent_entity'])['total_emissions_MtCO2e'].sum().unstack(fill_value=0)


fig, ax = plt.subplots(figsize=(14, 8)) 
emissions_top_entities_yearly.plot(kind='line', ax=ax) 

ax.set_title('Annual Emissions Trend for Top 5 Parent Entities')
ax.set_xlabel('Year')
ax.set_ylabel('Total Emissions (MtCO2e)')
ax.legend(title='Parent Entity', bbox_to_anchor=(1.05, 1), loc='upper left') 
ax.grid(True, linestyle='--', alpha=0.6)
plt.tight_layout(rect=[0, 0, 0.85, 1]) 
plt.show()
```
![14](https://github.com/user-attachments/assets/b06804d7-4cfd-43e9-ae8c-c85cc7c2130c)

```python
top_3_commodities = ['Oil & NGL', 'Bituminous Coal', 'Natural Gas']

df_top_commodities = df[df['commodity'].isin(top_3_commodities)].copy()

emissions_top_commodities_yearly = df_top_commodities.groupby(['year', 'commodity'])['total_emissions_MtCO2e'].sum().unstack(fill_value=0)

fig, ax = plt.subplots(figsize=(14, 8)) 
emissions_top_commodities_yearly.plot(kind='area', stacked=True, ax=ax) 

ax.set_title('Annual Emissions Trend by Top 3 Commodities (Stacked)')
ax.set_xlabel('Year')
ax.set_ylabel('Total Emissions (MtCO2e)')
ax.legend(title='Commodity', bbox_to_anchor=(1.05, 1), loc='upper left')
ax.grid(True, linestyle='--', alpha=0.6)
plt.tight_layout(rect=[0, 0, 0.85, 1])
plt.show()
```
![15](https://github.com/user-attachments/assets/9dd0098b-d50d-4300-b75f-35830cadb0fc)

