# 📊 Data Preprocessing Documentation

## Overview

This document explains how the anime dashboard data was preprocessed from the raw Kaggle CSV into an optimized JSON format for web deployment.

---

## Source Data

**Dataset**: [Top 15,000 Ranked Anime Dataset (Update to 3/2025)](https://www.kaggle.com/datasets/quanthan/top-15000-ranked-anime-dataset-update-to-32025)

**Source**: Kaggle  
**Original Size**: ~13 MB CSV  
**Total Entries**: 15,000 anime  

---

## Finished Site: https://tpaden1.github.io/anime-dashboard/

## Preprocessing Script

### File: `preprocess_data.py`

This Python script performs 11 documented steps to transform raw data into an optimized web-ready format.

### Requirements:
```bash
pip install pandas --break-system-packages
```

### Usage:
```bash
python3 preprocess_data.py
```

### Input:
- `top_15000_anime.csv` (raw Kaggle data)

### Output:
- `anime_data_optimized.json` (173.5 KB compressed JSON)

---

## Preprocessing Steps

### STEP 1: Load Raw Data
- Reads CSV file using pandas
- Verifies 15,000 entries loaded
- Displays all available columns

**Code**:
```python
df = pd.read_csv('top_15000_anime.csv')
```

---

### STEP 2: Data Cleaning
- Removes entries without scores or genres (critical fields)
- Filters out invalid scores (score <= 0)
- Handles missing episode counts (fill with 0)
- Handles missing member counts (fill with 0)

**Result**: 14,382 valid entries after cleaning

**Code**:
```python
df = df.dropna(subset=['score', 'genres'])
df = df[df['score'] > 0]
df['episodes'] = df['episodes'].fillna(0)
df['members'] = df['members'].fillna(0)
```

---

### STEP 3: Extract Primary Genre
- Anime can have multiple genres (comma-separated)
- Extracts first genre as "primary genre" for analysis
- Identifies 21 unique genres

**Top Genres**:
1. Action (4,403 anime)
2. Comedy (3,518 anime)
3. Adventure (1,380 anime)

**Code**:
```python
df['primary_genre'] = df['genres'].str.split(',').str[0].str.strip()
```

---

### STEP 4: Categorize Episode Counts
- Creates meaningful episode ranges for analysis
- 6 categories plus "Unknown"

**Episode Ranges**:
- 1-12 episodes
- 13-26 episodes  
- 27-52 episodes
- 53-100 episodes
- 101-200 episodes
- 200+ episodes

**Code**:
```python
def categorize_episodes(episodes):
    if pd.isna(episodes) or episodes == 0:
        return "Unknown"
    elif episodes <= 12:
        return "1-12"
    # ... etc
```

---

### STEP 5: Select Top 2,000 Anime
- Sorts by score (descending)
- Takes top 2,000 highest-rated anime
- Ensures quality dataset for dashboard

**Score Range**: 7.51 - 9.29  
**Highest Rated**: Sousou no Frieren (9.29)  
**Most Popular**: Shingeki no Kyojin (Attack on Titan)

**Code**:
```python
top_anime = df.nlargest(2000, 'score')
```

---

### STEP 6: Create Optimized Anime List
- Uses shortened field names to reduce file size
- Converts data types for JSON serialization
- Each anime has 6 essential fields

**Field Mapping**:
| Full Name | Short | Type | Description |
|-----------|-------|------|-------------|
| `name` | `n` | string | Anime title |
| `genre` | `g` | string | Primary genre |
| `score` | `s` | float | Rating (0-10) |
| `episodes` | `e` | int | Episode count |
| `members` | `m` | int | Community members |
| `range` | `r` | string | Episode range |

**Space Savings**: ~30% file size reduction

**Code**:
```python
anime_entry = {
    'n': str(row['name']),
    'g': str(row['primary_genre']),
    's': float(round(row['score'], 2)),
    'e': int(row['episodes']),
    'm': int(row['members']),
    'r': str(row['episode_range'])
}
```

---

### STEP 7: Pre-Calculate Genre Statistics
- Calculates average score per genre
- Counts anime per genre
- Sorts by average rating

**Why Pre-Calculate?**
- Browser doesn't need to compute statistics
- Charts render instantly
- Better performance for users

**Output Structure**:
```json
{
  "labels": ["Award Winning", "Adventure", "Action", ...],
  "scores": [7.17, 7.12, 7.08, ...],
  "counts": [245, 1380, 4403, ...]
}
```

**Code**:
```python
genre_stats = df.groupby('primary_genre').agg({
    'score': 'mean',
    'anime_id': 'count'
}).round(2)
```

---

### STEP 8: Pre-Calculate Episode Statistics
- Calculates average score per episode range
- Counts anime per range
- Maintains consistent ordering

**Findings**:
- Best rated range: 101-200 episodes (6.99 avg)
- Most common: 1-12 episodes (10,674 anime)

**Output Structure**:
```json
{
  "labels": ["1-12", "13-26", "27-52", "53-100", "101-200", "200+"],
  "scores": [6.85, 7.12, 7.05, 7.15, 6.99, 6.95],
  "counts": [10674, 2295, 921, 205, 123, 56]
}
```

---

### STEP 9: Package Data Structure
- Combines all components into single object
- Adds metadata for documentation
- Creates final data package

**Structure**:
```json
{
  "anime": [ /* 2000 anime entries */ ],
  "genreStats": { /* pre-calculated */ },
  "episodeStats": { /* pre-calculated */ },
  "metadata": {
    "totalAnime": 2000,
    "totalGenres": 21,
    "generatedAt": "2025-12-09 02:03:29",
    "sourceDataset": "Kaggle - Top 15,000 Ranked Anime Dataset"
  }
}
```

---

### STEP 10: Save Compressed JSON
- Writes JSON with minimal whitespace
- Uses compact separators (`,` and `:`)
- No indentation or extra spaces
- Ensures UTF-8 encoding

**File Size**: 173.5 KB (vs 13 MB original = 74x smaller!)

**Code**:
```python
with open('anime_data_optimized.json', 'w', encoding='utf-8') as f:
    json.dump(data_package, f, separators=(',', ':'), ensure_ascii=False)
```

---

## Optimization Techniques

### 1. Field Name Compression
**Before**:
```json
{
  "name": "Fullmetal Alchemist: Brotherhood",
  "genre": "Action",
  "score": 9.1,
  "episodes": 64,
  "members": 3577489
}
```

**After**:
```json
{
  "n":"Fullmetal Alchemist: Brotherhood",
  "g":"Action",
  "s":9.1,
  "e":64,
  "m":3577489
}
```

**Savings**: 30% smaller per entry

---

### 2. Pre-Calculated Statistics
**Without Preprocessing**:
```javascript
// Browser must calculate every time:
const avgScore = anime
  .filter(a => a.genre === 'Action')
  .reduce((sum, a) => sum + a.score, 0) / filtered.length;
```

**With Preprocessing**:
```javascript
// Already calculated:
const avgScore = DATA.genreStats.scores[0]; // Instant!
```

---

### 3. JSON Compression
**Standard JSON** (with formatting):
```json
{
  "name": "Anime Title",
  "score": 9.1
}
```

**Compressed JSON** (no whitespace):
```json
{"n":"Anime Title","s":9.1}
```

**Savings**: ~20% smaller file

---

## Results

### Performance Comparison

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **File Size** | 13 MB | 173.5 KB | **74x smaller** |
| **Entries** | 1000 | 2000 | **2x more data** |
| **Load Time** | 3-5 sec | <1 sec | **5x faster** |
| **Memory** | High | Low | **Efficient** |
| **Charts** | Calculated | Pre-rendered | **Instant** |

---

### Final Output Statistics

📊 **Data Summary**:
- Total anime: 2,000
- Score range: 7.51 - 9.29
- Average score: 8.25
- Total genres: 21
- Episode ranges: 6

💾 **File Optimization**:
- Original CSV: ~13 MB
- Optimized JSON: 173.5 KB
- Compression ratio: 74x smaller

⚡ **Performance Benefits**:
- Instant browser loading (<1 second)
- Pre-calculated statistics (no runtime computation)
- Optimized field names (30% size reduction)
- Ready for web deployment

---

## How to Update Data

### Step 1: Download New Data
```bash
# Download latest CSV from Kaggle
# Place in same directory as preprocess_data.py
```

### Step 2: Run Preprocessing
```bash
python3 preprocess_data.py
```

### Step 3: Verify Output
```bash
ls -lh anime_data_optimized.json
# Should be ~170-180 KB
```

### Step 4: Update Dashboard
- Replace data in `index.html`
- Or keep as separate JSON file and load it

---

## Technical Details

### Dependencies
```python
import pandas as pd      # Data manipulation
import json             # JSON serialization
from datetime import datetime  # Timestamps
import numpy as np      # Type handling
```

### Python Version
- Python 3.12+
- Works with Python 3.8+

### Data Types
- All integers converted from numpy.int64 to Python int
- All floats converted from numpy.float64 to Python float
- All strings explicitly cast to str
- Ensures JSON serialization compatibility

---

## Integration with Dashboard

### Embedded in HTML
The optimized JSON is embedded directly in `index.html`:

```html
<script>
    const DATA = {"anime":[...],"genreStats":{...}};
    // Data immediately available, no loading required
</script>
```

### Benefits:
- ✅ No external file loading
- ✅ No CORS issues
- ✅ Works offline
- ✅ Single file deployment
- ✅ Instant availability

---

## Quality Assurance

### Data Validation
- ✅ All entries have valid scores (> 0)
- ✅ All entries have genres
- ✅ Episode counts handled (0 for unknown)
- ✅ Member counts handled (0 for unknown)
- ✅ No null values in critical fields

### Statistical Integrity
- ✅ Calculations verified against source data
- ✅ Averages rounded to 2 decimal places
- ✅ Counts match source dataset
- ✅ Rankings preserved from Kaggle

---

## Files Included

1. **preprocess_data.py** (10 KB)
   - Documented preprocessing script
   - 11 steps with detailed comments
   - Console output for verification

2. **anime_data_optimized.json** (173.5 KB)
   - Processed dataset
   - 2,000 anime entries
   - Pre-calculated statistics
   - Ready for web deployment

3. **index.html** (191 KB)
   - Complete dashboard
   - Embedded data
   - Interactive charts
   - Single-file deployment

---

## Conclusion

The preprocessing script transforms 13 MB of raw CSV data into a 173.5 KB optimized JSON file—a **74x reduction** in size while **doubling** the number of anime entries included.

This optimization enables:
- Instant loading in web browsers
- Smooth user experience
- Professional visualizations
- Mobile-friendly performance
- Single-file deployment

**The preprocessing makes the dashboard fast, efficient, and production-ready!** 🚀
## Acknowledgements

Some of the wording, section organization, and formatting of this README were refined with the help of ChatGPT, which I used as a writing assistant to make the documentation clearer and more readable.
