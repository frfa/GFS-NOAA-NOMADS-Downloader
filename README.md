# XyGrib GFS GRIB2 downloader from NOAA NOMADS

This is a fork of DrCalambre's fantastic GFS NOAA NOMADS downloader for XyGrib.
I tried to make it more UNIXish by translating it to English, added various
command line flags that you would expect from a command line tool, and added
predefined regions and data sets. See `xygrib-noaa --help` ;)

Change and add regions and datasets to your liking in the script. The parameters
and levels though were modeled after the original NOMADS GRIG file downloader at
<https://nomads.ncep.noaa.gov/gribfilter.php?ds=gfs_0p25>.

Also a trap has been introduced to remove the temp working directory if the
script is interrupted or an error occurs during script execution.

And BTW - it works without Bash now, a standard POSIX-compatible shell should
suffice (like the standard `/bin/sh` which is `/bin/dash` on Ubuntu or the like).

# GFS NOAA NOMADS Downloader for XyGrib — GRIB2 Forecast

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Bash](https://img.shields.io/badge/Bash-4.0+-green.svg)](https://www.gnu.org/software/bash/)

> **Direct download from NOAA/NOMADS of a GRIB2 forecast, ready for XyGrib.**  
> Eliminates dependency on the OpenGribs intermediary server.  
> **Now with optional wave data (WW3)!** 🌊

---

## 📌 Background

Since **September 2026**, the OpenGribs server that generates and serves GRIB datasets for XyGrib has been down ([issue #326](https://github.com/opengribs/XyGrib/issues/326)).

This script **downloads directly from the official source (NOAA NOMADS)** and builds a **GRIB2** that XyGrib can open without intermediaries.

✅ **No OpenGribs dependency**  
✅ **No GUI required**  
✅ **Only `curl` and Bash**  
✅ **Configurable forecast horizon (0-384 hours)**  
✅ **Includes 0°C isotherm** (validated in Río Gallegos)  
✅ **Optional wave data (WW3)** — height, direction, period  
✅ **Robust and reliable** — automatic cycle detection, fallback, and retry

---

## 🚀 Features

| Feature | Detail |
|---|---|
| **Model** | GFS 0.25° (NOAA/NCEP) |
| **Cycle** | **Automatic detection** (18Z → 12Z → 06Z → 00Z) with smart fallback |
| **Horizon** | 0 – 384 hours (configurable, default 72h / 3 days) |
| **Interval** | 3 hours (0-240h) / 12 hours (240-384h) |
| **Region** | `-90°W` to `-30°W` / `-20°S` to `-60°S`<br>(South America and surrounding waters) *configurable* |
| **Variables** | Temperature, wind, gusts, pressure, humidity, cloud cover, precipitation, snow, CAPE, **0°C isotherm**, freezing rain, etc. |
| **Output** | Single GRIB2 in `~/.xygrib/grib/GFS_NOAA_YYYYMMDD_XXhs.grib2` |
| **Temporaries** | Stored in `/tmp/gfs-...` and **automatically cleaned up** after execution |
| **Validation** | Automatic GRIB format check using `file` command |
| **Error handling** | Smart retry with fallback cycles on 404 errors |
| **Fallback date** | Automatic retry with previous days if no cycles available |
| **Progress** | Compact output with per-file status (`[01/25] F000 → ✅ 12Z`) |
| **🌊 Wave data (WW3)** | **Optional** — height, direction, period, with intelligent file detection |

---

## 📦 Installation

### Option 1 — Clone the repository

```bash
cd ~/bin  # or any directory in your PATH
git clone https://github.com/DrCalambre/GFS_NOAA_NOMADS_Downloader_for_XyGrib-72h_GRIB2_Forecast
cd GFS_NOAA_NOMADS_Downloader_for_XyGrib-72h_GRIB2_Forecast
chmod +x xygrib-noaa.sh
```

### Option 2 — Direct download

```bash
wget https://raw.githubusercontent.com/DrCalambre/GFS_NOAA_NOMADS_Downloader_for_XyGrib-72h_GRIB2_Forecast/main/xygrib-noaa.sh
chmod +x xygrib-noaa.sh
```

### Option 3 — Manual copy

If you already have the script on your system, just make it executable:

```bash
chmod +x xygrib-noaa.sh
```

---

## ▶️ Usage

```bash
./xygrib-noaa.sh
```

### What it does

1. Downloads **filtered GRIB files** from NOAA NOMADS (number depends on `MAX_FORECAST`).
   - Default: 25 files for 72h (3 days) at 3-hour intervals.
2. Waits **8 seconds** between requests (respecting NOAA's recommendation).
3. Concatenates the files into a **single GRIB2**.
4. Saves it to `~/.xygrib/grib/GFS_NOAA_YYYYMMDD_XXhs.grib2`.
5. Optionally, downloads wave data (WW3) from NOAA NOMADS, saving it as `WW3_NOAA_YYYYMMDD_XXhs.grib2`.
6. Displays size and location information.

### Example output

```text
============================================================
 GFS NOAA NOMADS - v1.0.3
 72-hour forecast for XyGrib
 + Wave data (WW3)
============================================================

Date        : 20260908
Cycle       : 12Z
Horizon     : f000 → f072
Interval    : 3 hours (12h beyond 240h)
Time steps  : 25

...
[25/25] F072 →  ✅ 4.0M [12Z]

============================================================
 DOWNLOADING WAVE DATA (WW3)
============================================================

🔍 Detecting available WW3 cycle...
✅ Using WW3 cycle: 12Z (date: 20260908)

[01] WAVE F000 →  ✅ 772K
[02] WAVE F003 →  ✅ 772K
...
[25] WAVE F072 →  ✅ 824K

============================================================
 WAVE RESULTS
============================================================

✅ Successful : 25
❌ Failed    : 0
📊 Total     : 25
```

---

## ⚙️ Advanced configuration

You can edit the script to adjust these parameters:

| Variable | Description | Default |
|---|---|---|
| `MAX_FORECAST` | Forecast horizon in hours (0-384) | `72` |
| `MAX_DAYS_BACK` | Days to look back if current date has no cycles | `3` |
| `PAUSE` | Pause between downloads (seconds) | `8` |
| `WEST`, `EAST`, `NORTH`, `SOUTH` | Geographic region | `-90`, `-30`, `-20`, `-60` |

---

## 🌊 Wave data (WW3) configuration

The script can optionally download wave data from NOAA's WaveWatch III (WW3) model. This data includes:

- **Significant wave height** (HTSGW)
- **Primary wave direction** (DIRPW)
- **Primary wave period** (PERPW)
- **Wind direction and speed** at surface

### Enable/disable wave data

Edit the script and set:

```bash
# Set to true to download wave data, false to skip
DOWNLOAD_WAVES=true   # or false
```

### How it works

- The script **independently detects the best WW3 cycle** (18Z → 12Z → 06Z → 00Z), separate from GFS.
- It uses **intelligent file detection**: tries to download files until it finds a missing one, ensuring only existing files are fetched.
- If no WW3 cycle is available for the current date, it will retry with previous days (up to `MAX_DAYS_BACK`).

### Wave data output

- The wave data is saved as a separate GRIB2 file:
  ```
  ~/.xygrib/grib/WW3_NOAA_YYYYMMDD_XXhs.grib2
  ```
- You can open it in XyGrib together with the GFS file to overlay weather and wave information.

---

## 🗺️ Adjusting the geographic area

The script downloads a GRIB2 file for a specific region. By default, it covers:

```
West:  -90°  →  East:  -30°
North: -20°  →  South: -60°
```

This area includes the southern cone of South America and surrounding waters.

### How to change the region

Edit the script and modify these variables:

```bash
WEST="-80"      # Longitude limit (west)
EAST="-62"      # Longitude limit (east)
NORTH="-15"     # Latitude limit (north)
SOUTH="-74"     # Latitude limit (south)
```

**Important:** Longitudes are negative for the Western Hemisphere. Latitudes are negative for the Southern Hemisphere.

### Examples

#### 1. Focus on Patagonia

```bash
WEST="-75"
EAST="-65"
NORTH="-40"
SOUTH="-56"
```

#### 2. Cover all of South America

```bash
WEST="-85"
EAST="-35"
NORTH="10"
SOUTH="-60"
```

#### 3. Focus on a specific coastal area

```bash
WEST="-80"
EAST="-62"
NORTH="-15"
SOUTH="-74"
```

### How to find the right coordinates

You can get coordinates from:

- **Google Maps** → Right-click → "What's here?"
- **OpenStreetMap** → Click on a location → shows coordinates
- **GPS tools** → Any tool that shows latitude/longitude

**Remember:**
- Longitude: West = negative, East = positive
- Latitude: South = negative, North = positive

---

## 🧪 Validation

The script was tested on **XyGrib 1.2.6 / antiX Linux 26** with the following verified parameters:

| XyGrib Parameter | Value in Río Gallegos |
|---|---|
| Temperature (2 m) | 7.0 °C |
| Wind (10 m) | 251° / 32.1 km/h |
| Wind gusts | 39.4 km/h |
| CAPE | 0 J/kg |
| Relative humidity | 59 % |
| Dew point | -0.5 °C |
| Cloud cover | 4.3 % |
| Precipitation | 0.00 mm/h |
| Snow possible | 0 |
| Freezing rain | 0 |
| Snow depth | 0.0 cm |
| **0°C isotherm** | **1147 m** ✅ |
| **Significant wave height** | ✅ Available (WW3) |

---

## 🗂️ File structure

### Temporary (during download)

```text
/tmp/gfs-YYYYMMDD-$$
├── gfs_000.grib2
├── gfs_003.grib2
├── gfs_006.grib2
...
└── gfs_072.grib2

/tmp/wave-YYYYMMDD-$$
├── wave_000.grib2
├── wave_003.grib2
├── wave_006.grib2
...
└── wave_072.grib2
```

### Final (permanent)

```text
~/.xygrib/grib/
├── GFS_NOAA_YYYYMMDD_XXhs.grib2
└── WW3_NOAA_YYYYMMDD_XXhs.grib2  (if DOWNLOAD_WAVES=true)
```

---

## ⚠️ Important notes

1. **Pause between requests:** NOAA recommends spacing requests to avoid overloading NOMADS. The script waits 8 seconds between each download.
2. **Download failures:** If any individual file fails, the script stops and shows which `fXXX` failed.
3. **Temporary files:** They are kept in `/tmp/gfs-...` and `/tmp/wave-...` for debugging if needed.
4. **CDO compatibility:** If you run `cdo showname` and get `Unsupported file structure`, **don't worry** — XyGrib can still open the file. This happens with some GRIB structures that CDO can't interpret but XyGrib handles fine.
5. **Wave data:** WW3 data is optional and can be enabled/disabled with `DOWNLOAD_WAVES`. It uses `all_var=on` and `all_lev=on` for reliability, which means the file includes all available wave variables.

---

## 🔧 Requirements

- `curl` (installed by default on most systems)
- Bash 4.0+
- Internet access to reach `nomads.ncep.noaa.gov`

If `curl` is not installed:

```bash
# Debian/Ubuntu/MX Linux/antiX
sudo apt install curl

# Fedora
sudo dnf install curl

# Arch
sudo pacman -S curl
```

---

## ⏰ Automating with anacron (Linux)

To make the most of this script, you can automate it to download the latest forecast **once a day** without having to remember to run it manually.

**anacron** is the perfect tool for this. Unlike `cron`, it is designed for **laptops and desktops that are not running 24/7**. It will execute the task the next time you turn on your computer, ensuring you always get your daily update.

### 📝 Step-by-step: Add the task to anacron

Follow these simple steps to automate the download:

1.  **Open your personal anacrontab file** in a text editor:
    ```bash
    nano ~/.anacron/anacrontab
    ```

2.  **Add the following line** at the end of the file:
    ```text
    1       10      descargar_gfs_xygrib   /home/your_user/xygrib-noaa.sh > /home/your_user/xygrib-forecast.log 2>&1 && echo "---- $(date) ----" >> /home/your_user/xygrib-forecast.log
    ```

    **Important:** Replace `/home/your_user/` with the actual path to your script and log file.

3.  **Explanation of the line:**
    | Part | Meaning |
    | :--- | :--- |
    | `1` | Run the job **once a day**. |
    | `10` | Wait **10 minutes** after booting before running the command. |
    | `descargar_gfs_xygrib` | A unique identifier for this job. |
    | `/home/your_user/xygrib-noaa.sh` | The full path to your script. |
    | `> /home/your_user/xygrib-forecast.log 2>&1` | Redirects all output (including errors) to a log file in your home directory. |
    | `&& echo "---- $(date) ----" >> /home/your_user/xygrib-forecast.log` | Appends a timestamp to the log after the script finishes. |

4.  **Save and close** the file (`Ctrl+O`, `Enter`, `Ctrl+X`).

That's it! Starting tomorrow, your system will automatically download a fresh GRIB forecast for you.

### 🔍 Check the log

To verify that everything is working, you can check the log file:

```bash
tail -f /home/your_user/xygrib-forecast.log
```

---

## 🐛 Troubleshooting

### Error: `curl: (22) The requested URL returned error: 500`

- Verify that the UTC date is correct (`date -u +%Y%m%d`).
- Check that the chosen cycle (`CYCLE`) is available on NOAA for that date.
- Try switching to another cycle (e.g., `00` or `06`).

### XyGrib doesn't show the 0°C isotherm

- Make sure the file was generated with **V9** or later (includes `lev_0C_isotherm`).
- Verify that the `HGT` variable at the `0C isotherm` level is included in the URL.

### The final file doesn't appear in XyGrib

- Confirm that the `~/.xygrib/grib` directory exists.
- XyGrib may need to be restarted to see new files in that folder.
- You can also open the file manually from **XyGrib → File → Open GRIB...**

### Wave data (WW3) fails to download

- Check that `DOWNLOAD_WAVES=true` is set in the script.
- The script will automatically retry with previous days if no WW3 cycle is available.
- If the issue persists, check your internet connection and NOAA's service status.

---

## 🤝 Contributing

If you find an issue, have an improvement, or want to add support for other models (DWD ICON, etc.), please open an **issue** or **pull request**.

---

## 📸 Screenshots

### Selecting the GRIB file in XyGrib

![Selecting GRIB file](screenshots/select-grib.jpg)

*The generated GRIB2 file ready to be opened in XyGrib.*

---

### 72-hour forecast displayed in XyGrib

![XyGrib forecast](screenshots/xygrib-forecast.jpg)

*72-hour GFS forecast loaded in XyGrib, showing wind, pressure, temperature, and the 0°C isotherm.*

---

### 72-hour forecast displayed in XyGrib

![XyGrib forecast](screenshots/xygrib-forecast_02.jpg)

---

### Wave data (WW3) displayed in XyGrib

![WW3 wave data in XyGrib](screenshots/ww3-xygrib.jpg)

*Wave data from NOAA's WaveWatch III (WW3) loaded in XyGrib, showing significant wave height, direction, and period.*

---

### Wave data (WW3) displayed in XyGrib

![WW3 wave data in XyGrib](screenshots/ww3-xygrib_02.jpg)

*Wave data from NOAA's WaveWatch III (WW3) loaded in XyGrib. The screenshot shows significant wave height forecasts for a point in the South Atlantic (48.78°S 044.73°W) with values ranging from 2.5 m to 3.8 m over the forecast period. The data includes primary wave direction, period, and wind wave information, providing a complete picture of sea state conditions for maritime and coastal planning.*

---

### The xygrib forecast log file

![XyGrib log](screenshots/xygrib-forecast-log.jpg)

![XyGrib forecast](screenshots/xygrib-forecast-log_02.jpg)

*The log file showing a successful 336-hour (14-day) GFS and WW3 forecast download. The script downloaded 89 wave files (F000 to F336) with zero failures, demonstrating its robustness for extended horizons. The final GRIB files are 98 MB (GFS) and 70 MB (WW3).*

---

### GFS-NOAA File information

![XyGrib File information](screenshots/GFS-NOAA_file-info.jpg)

*Information from the grib2 file downloaded using this script*

---

## 📋 Changelog

### v1.0.3 — 2026-09-08
**Critical bug fix and reliability improvements**

- **Fixed:** Silent file skipping when `MAX_FORECAST > 240h` (STEP mutation bug)
- Generate `HOURS` array once, use it everywhere (download, concatenation, WW3)
- More reliable cycle detection using `curl --range 0-0` instead of `HEAD`
- `test_cycle()` now uses actual region coordinates instead of hardcoded values
- Renamed local `date` variables to `d` to avoid shadowing the `date` command
- Updated temporary directory naming to reflect current version
- `EXPECTED_FILES` calculated dynamically from `HOURS` array

---

### v1.0.2 — 2026-09-08
**Wave data (WW3) integration**

- **New** optional wave data download (WW3) with intelligent file detection
- **New** `DOWNLOAD_WAVES` variable to enable/disable wave data
- Independent cycle detection for WW3 (separate from GFS)
- Wave data saved as `WW3_NOAA_YYYYMMDD_XXhs.grib2`
- Automatic retry with previous days if no WW3 cycle available
- Uses `all_var=on` and `all_lev=on` for reliable wave data download

---

### v1.0.1 — 2026-09-08
**Improved stability release**

- Automatic retry with previous days if no GFS cycles available for current date
- New `MAX_DAYS_BACK` variable (default 3 days)
- Informative messages when using data from a previous day
- Region adjusted to northern Argentina (`NORTH="-20"`)
- Always finds data, even when run early in the day (00:00-04:00 UTC)

---

### v1.0.0 — 2026-09-07
**First stable release**

- Automatic detection of available GFS cycles (18Z → 12Z → 06Z → 00Z)
- Smart fallback to alternative cycles when a file returns 404
- Clean progress output with compact messages (`[01/25] F000 → ✅ 12Z`)
- Automatic cleanup of temporary files (configurable)
- GRIB validation using `file` command
- Region optimized for South America
- Tested on antiX Linux 26 / XyGrib 1.2.6

---

## 📄 License

**MIT** — free use, no warranty.

---

## 🌐 Useful links

- [NOAA NOMADS GRIB Filter — GFS (weather)](https://nomads.ncep.noaa.gov/gribfilter.php?ds=gfs_0p25)
- [NOAA NOMADS GRIB Filter — WW3 (waves)](https://nomads.ncep.noaa.gov/gribfilter.php?ds=gfswave)
- [XyGrib — GitHub](https://github.com/opengribs/XyGrib)
- [OpenGribs issue #326 — GRIB server down](https://github.com/opengribs/XyGrib/issues/326)
- [GFS 0.25° documentation](https://www.nco.ncep.noaa.gov/pmb/products/gfs/)
- [WaveWatch III (WW3) documentation](https://polar.ncep.noaa.gov/waves/index2.shtml)

---

## 🧠 Credits

Developed from tests conducted with **XyGrib 1.2.6** on **antiX Linux 26**.

Thanks to the XyGrib community and NOAA/NCEP for keeping the data open.

---
## 📸 Why this project exists

"Piedra del Fraile" 🏔️❄️🇦🇷

This photograph was taken on July 11, 2026, while following the trail toward Piedra del Fraile, before reaching the refuge.

In the depths of winter, the landscape takes on an almost otherworldly appearance. The forest is covered in a delicate layer of frost, turning every branch and shrub into a pale silhouette. Below, the cold waters of the Río Eléctrico flow quietly through the valley, surrounded by rounded stones and scattered boulders.

Beyond the river, the mountains rise dramatically on both sides, creating a natural corridor that leads the eye toward the distant, snow-covered peaks of the Andes. The contrast between the dark rock, the frozen vegetation, the icy blue water and the small patch of blue sky makes this scene feel both wild and incredibly peaceful.

This is one of those places where the immensity of Patagonia becomes truly apparent. Far from the crowds and deep into the wilderness, the trail follows the Río Eléctrico through a landscape shaped by ice, water and mountains.

A quiet winter moment in one of the most beautiful corners of Argentine Patagonia. 🏔️❄️🇦🇷

![Piedra del Fraile, Patagonia](screenshots/PiedraDelFraile.jpg)

— The landscape that inspired this project. Weather in Patagonia can change in minutes, and having reliable forecast data is essential for anyone venturing into these mountains.

**Fair winds!** 🌬️
