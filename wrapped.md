---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.17.2
  kernelspec:
    display_name: light
    language: python
    name: python3
---

# Mine spotify wrapped

When You obtain your data from Spotify, put them into data folder. Then run the following code to explore your data.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import os
import seaborn as sns
import json
```

```python
# load json files from a given path
def load_json(file_path):
    json_data = []

    for filename in os.listdir(file_path):
        if filename.endswith(".json"):
            path = os.path.join(file_path, filename)
            print(f"Loading JSON file: {path}")
            with open(path, "r") as file:
                data = json.load(file)
                json_data.append(data)
    return json_data


json_data = load_json("data")
```

```python
# data are stored as netsed list
flattened = []
for entry in json_data:
    if isinstance(entry, list):
        flattened.extend(entry)
    else:
        flattened.append(entry)

# Convert to DataFrame
df = pd.DataFrame(flattened)

# Convert timestamp to datetime
df["ts"] = pd.to_datetime(df["ts"], errors="coerce")

# drop rows with NaN values in 'ms_played' or 'ts'
df = df.dropna(subset=["ms_played"])

df.head()
```

## Start explorations

```python
df.info()
```

# Music

```python
total_plays = len(df)
print("Total tracks/episodes played:", total_plays)
```

```python
df_music = df[df["episode_name"].isna()]
```

```python
print("Total hours:", round(df_music["ms_played"].sum() / (1000 * 60 * 60)))
```

```python
print("Top 5 tracks:", df_music["master_metadata_track_name"].value_counts().head(5))
```

```python
print(
    "Top 5 artists:",
    df_music["master_metadata_album_artist_name"].value_counts().head(5),
)
```

```python
# top  5 artist in 23 24 25
df_music["year"] = df_music["ts"].dt.year

top_artists = (
    df_music[df_music["year"].isin([2023, 2024, 2025])]
    .groupby("master_metadata_album_artist_name")["ms_played"]
    .sum()
    .reset_index()
)
top_artists = top_artists.sort_values(by="ms_played", ascending=False).head(5)
print("Top 5 artists in 2023, 2024, 2025:")
print(top_artists)
```

```python
# Sum ms_played per year and total
listening_per_year = df_music.groupby("year")["ms_played"].sum().reset_index()

# Convert ms_played to hours
listening_per_year["hours"] = listening_per_year["ms_played"] / (1000 * 60 * 60)


plt.figure(figsize=(10, 6))
sns.barplot(data=listening_per_year, x="year", y="hours", palette="viridis")
plt.title("Listening Time per Year", fontsize=16)
plt.xlabel("Year", fontsize=12)
plt.ylabel("Listening Time (hours)", fontsize=12)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

### show on a map the countries where you listened to music


```python
listening_by_country = df_music.groupby("conn_country")["ms_played"].sum().reset_index()

import plotly.express as px
import pycountry

# Convert ms_played to hours
listening_by_country["hours"] = listening_by_country["ms_played"] / (1000 * 60 * 60)


# Convert Alpha-2 to Alpha-3 country codes
def alpha2_to_alpha3(alpha2):
    country = pycountry.countries.get(alpha_2=alpha2)
    return country.alpha_3 if country else None


listening_by_country["alpha3"] = listening_by_country["conn_country"].apply(
    alpha2_to_alpha3
)

# Drop rows with invalid or missing alpha3 codes
cleaned = listening_by_country.dropna(subset=["alpha3"])

fig = px.choropleth(
    cleaned,
    locations="alpha3",
    color="hours",
    color_continuous_scale="viridis",
    title="Spotify Listening Time by Country",
    labels={"hours": "Listening Time (hours)"},
    projection="natural earth",
)

fig.update_geos(showcountries=True)
fig.update_layout(margin={"r": 0, "t": 40, "l": 0, "b": 0})
fig.show()
```

# Podcasts

```python
df_podcasts = df[df["episode_name"].notna()]

```

```python
print("Total hours:", round(df_podcasts["ms_played"].sum() / 3600000))
```

```python
print("Top 5 tracks:", df_podcasts["episode_show_name"].value_counts().head(5))

```

```python
# Sum ms_played per year and total
listening_per_year = df_podcasts.groupby("year")["ms_played"].sum().reset_index()

# Convert ms_played to hours
listening_per_year["hours"] = listening_per_year["ms_played"] / (1000 * 60 * 60)


plt.figure(figsize=(10, 6))
sns.barplot(data=listening_per_year, x="year", y="hours", palette="viridis")
plt.title("Listening Time per Year", fontsize=16)
plt.xlabel("Year", fontsize=12)
plt.ylabel("Listening Time (hours)", fontsize=12)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

```
