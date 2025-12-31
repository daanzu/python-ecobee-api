# python-ecobee-api

A Python library for controlling Ecobee smart thermostats via the official Ecobee API.

## Table of Contents

- [Overview](#overview)
- [Supported Devices](#supported-devices)
- [Installation](#installation)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Authentication](#authentication)
  - [PIN-Based Authorization](#pin-based-authorization)
  - [Web-Based Authorization](#web-based-authorization)
  - [Token Management](#token-management)
- [Configuration](#configuration)
  - [File-Based Configuration](#file-based-configuration)
  - [Dictionary-Based Configuration](#dictionary-based-configuration)
- [Usage Examples](#usage-examples)
  - [Getting Thermostat Data](#getting-thermostat-data)
  - [Temperature Control](#temperature-control)
  - [HVAC Mode Control](#hvac-mode-control)
  - [Fan Control](#fan-control)
  - [Climate/Comfort Settings](#climatecomfort-settings)
  - [Vacation Management](#vacation-management)
  - [Humidity Control](#humidity-control)
  - [Ventilator Control](#ventilator-control)
  - [Remote Sensors](#remote-sensors)
  - [Messages](#messages)
  - [Advanced Settings](#advanced-settings)
- [API Reference](#api-reference)
- [Error Handling](#error-handling)
- [Integration with Home Assistant](#integration-with-home-assistant)
- [License](#license)

---

## Overview

`python-ecobee-api` (pyecobee) provides a simple Python interface to interact with Ecobee smart thermostats. It wraps the official [Ecobee API](https://www.ecobee.com/home/developer/api/introduction/index.shtml) and handles authentication, token management, and all thermostat operations.

**Key Features:**
- PIN-based and web-based authentication flows
- Automatic token refresh handling
- Complete thermostat control (temperature, HVAC mode, fan, humidity, etc.)
- Vacation and climate hold management
- Remote sensor support
- File-based or dictionary-based configuration

---

## Supported Devices

This library supports all Ecobee smart thermostat models:

| Model ID | Device Name |
|----------|-------------|
| `idtSmart` | ecobee Smart Thermostat |
| `idtEms` | ecobee Smart EMS Thermostat |
| `siSmart` | ecobee Si Smart Thermostat |
| `siEms` | ecobee Si EMS Thermostat |
| `athenaSmart` | ecobee3 Smart Thermostat |
| `athenaEms` | ecobee3 EMS Thermostat |
| `corSmart` | Carrier/Bryant Cor Thermostat |
| `nikeSmart` | ecobee3 lite Smart Thermostat |
| `nikeEms` | ecobee3 lite EMS Thermostat |
| `apolloSmart` | ecobee4 Smart Thermostat |
| `vulcanSmart` | ecobee Smart Thermostat with Voice Control |

---

## Installation

### Using pip

```bash
pip install python-ecobee-api
```

### From source

```bash
git clone https://github.com/nkgilley/python-ecobee-api.git
cd python-ecobee-api
pip install .
```

### Dependencies

- Python 3.7+
- `requests>=2.25`

---

## Prerequisites

Before using this library, you need to obtain an **API Key** from Ecobee:

1. Log into your Ecobee account at [ecobee.com](https://www.ecobee.com)
2. Navigate to the **Developer** section in your account settings
3. Create a new application to receive your **API Key** (also called `client_id`)
4. Store this API key securely - you'll need it for authentication

> ⚠️ **Important**: Your API key is a secret. Never share it or commit it to version control.

---

## Quick Start

Here's a minimal example to get started:

```python
from pyecobee import Ecobee

# Method 1: Using a configuration file
ecobee = Ecobee(config_filename='ecobee.conf')
ecobee.read_config_from_file()

# Method 2: Using a dictionary (recommended for applications)
config = {
    'API_KEY': 'your-api-key-here',
}
ecobee = Ecobee(config=config)

# First-time setup: Request PIN for authorization
if ecobee.request_pin():
    print(f"Please authorize the app using PIN: {ecobee.pin}")
    print("Go to ecobee.com -> My Apps -> Add Application -> Enter PIN")
    input("Press Enter after authorizing...")

    # Exchange PIN for tokens
    if ecobee.request_tokens():
        print("Authorization successful!")

# Get thermostat data
if ecobee.get_thermostats():
    for i, thermostat in enumerate(ecobee.thermostats):
        print(f"Thermostat {i}: {thermostat['name']}")
        print(f"  Current temp: {thermostat['runtime']['actualTemperature'] / 10}°F")
```

---

## Authentication

The Ecobee API uses OAuth 2.0 for authentication. This library supports two authentication methods:

### PIN-Based Authorization

This is the recommended method for most applications. It uses a 4-character PIN that the user enters on the Ecobee portal.

```python
from pyecobee import Ecobee

# Initialize with your API key
config = {'API_KEY': 'your-api-key-here'}
ecobee = Ecobee(config=config)

# Step 1: Request a PIN
if ecobee.request_pin():
    print(f"Authorization PIN: {ecobee.pin}")
    print("Steps to authorize:")
    print("1. Go to https://www.ecobee.com/consumerportal/index.html")
    print("2. Log in to your account")
    print("3. Click 'My Apps' in the menu")
    print("4. Click 'Add Application'")
    print("5. Enter the PIN shown above")
    print("6. Click 'Authorize'")

    # Wait for user to complete authorization
    input("\nPress Enter after you've authorized the app...")

    # Step 2: Request access tokens
    if ecobee.request_tokens():
        print("Successfully obtained access tokens!")
        print(f"Access Token: {ecobee.access_token[:20]}...")
        print(f"Refresh Token: {ecobee.refresh_token[:20]}...")
    else:
        print("Failed to obtain tokens. Make sure you authorized the app.")
else:
    print("Failed to request PIN. Check your API key.")
```

**PIN Authorization Flow:**
1. Your app requests a PIN from Ecobee
2. Display the PIN to the user (valid for ~10 minutes)
3. User logs into ecobee.com and enters the PIN in "My Apps"
4. Your app exchanges the authorization code for access/refresh tokens
5. Store the tokens securely for future use

### Web-Based Authorization

For applications that have user credentials, you can use web-based authentication:

```python
from pyecobee import Ecobee

config = {
    'API_KEY': 'your-api-key-here',
    'USERNAME': 'user@example.com',
    'PASSWORD': 'user-password',
}
ecobee = Ecobee(config=config)

# This will perform web-based login
if ecobee.request_tokens():
    print("Successfully authenticated!")
```

> ⚠️ **Security Note**: Storing user credentials is not recommended for production applications. Use PIN-based authorization whenever possible.

### Token Management

Access tokens expire after approximately 1 hour. The library handles token refresh automatically, but you can also manually refresh:

```python
# Automatic refresh happens in _request_with_refresh()
# But you can manually refresh if needed:
if ecobee.refresh_tokens():
    print("Tokens refreshed successfully!")
    # Updated tokens are available in:
    # ecobee.access_token
    # ecobee.refresh_token
```

---

## Configuration

### File-Based Configuration

Create a JSON configuration file (e.g., `ecobee.conf`):

```json
{
    "API_KEY": "your-api-key-here",
    "ACCESS_TOKEN": "your-access-token",
    "REFRESH_TOKEN": "your-refresh-token",
    "AUTHORIZATION_CODE": "your-auth-code"
}
```

Use the file-based configuration:

```python
from pyecobee import Ecobee

ecobee = Ecobee(config_filename='ecobee.conf')
ecobee.read_config_from_file()

# The library automatically updates the file when tokens are refreshed
```

### Dictionary-Based Configuration

For programmatic control, use dictionary-based configuration:

```python
from pyecobee import (
    Ecobee,
    ECOBEE_API_KEY,
    ECOBEE_ACCESS_TOKEN,
    ECOBEE_REFRESH_TOKEN,
)

config = {
    ECOBEE_API_KEY: 'your-api-key',
    ECOBEE_ACCESS_TOKEN: 'your-access-token',
    ECOBEE_REFRESH_TOKEN: 'your-refresh-token',
}

ecobee = Ecobee(config=config)

# After operations, get updated config:
updated_config = ecobee.config
# Store updated_config in your database/storage
```

### Configuration Options

| Key | Description |
|-----|-------------|
| `API_KEY` | Your Ecobee developer API key (required) |
| `ACCESS_TOKEN` | OAuth access token (obtained after authorization) |
| `REFRESH_TOKEN` | OAuth refresh token (obtained after authorization) |
| `AUTHORIZATION_CODE` | Authorization code from PIN request |
| `USERNAME` | Ecobee account email (for web auth) |
| `PASSWORD` | Ecobee account password (for web auth) |
| `INCLUDE_NOTIFICATIONS` | Set to "true" to include notification settings |

---

## Usage Examples

### Getting Thermostat Data

```python
from pyecobee import Ecobee

ecobee = Ecobee(config=your_config)

# Fetch all thermostats
if ecobee.get_thermostats():
    for index, thermostat in enumerate(ecobee.thermostats):
        # Basic info
        print(f"Name: {thermostat['name']}")
        print(f"Identifier: {thermostat['identifier']}")
        print(f"Model: {thermostat['modelNumber']}")

        # Runtime data
        runtime = thermostat['runtime']
        print(f"Current Temperature: {runtime['actualTemperature'] / 10}°F")
        print(f"Current Humidity: {runtime['actualHumidity']}%")
        print(f"Desired Heat: {runtime['desiredHeat'] / 10}°F")
        print(f"Desired Cool: {runtime['desiredCool'] / 10}°F")

        # Settings
        settings = thermostat['settings']
        print(f"HVAC Mode: {settings['hvacMode']}")
        print(f"Fan Min On Time: {settings['fanMinOnTime']} min/hr")

        # Equipment status
        print(f"Equipment Running: {thermostat['equipmentStatus']}")

        # Weather
        weather = thermostat['weather']['forecasts'][0]
        print(f"Outdoor Temperature: {weather['temperature'] / 10}°F")

# Update thermostat data (alias for get_thermostats)
ecobee.update()
```

### Temperature Control

```python
# Set a temperature hold
# Temperatures are in Fahrenheit
ecobee.set_hold_temp(
    index=0,                    # Thermostat index
    cool_temp=75.0,             # Cooling setpoint
    heat_temp=68.0,             # Heating setpoint
    hold_type='nextTransition'  # Hold until next schedule change
)

# Hold types:
# - 'nextTransition': Until the next scheduled program change
# - 'indefinite': Until manually cancelled
# - 'holdHours': For a specific number of hours
# - 'dateTime': Until a specific date/time

# Hold for a specific number of hours
ecobee.set_hold_temp(
    index=0,
    cool_temp=72.0,
    heat_temp=70.0,
    hold_type='holdHours',
    hold_hours='4'  # Hold for 4 hours
)

# Resume the regular program schedule
ecobee.resume_program(index=0)

# Resume all events (clears all holds)
ecobee.resume_program(index=0, resume_all=True)
```

### HVAC Mode Control

```python
# Available HVAC modes: 'auto', 'auxHeatOnly', 'cool', 'heat', 'off'

# Set to cooling mode
ecobee.set_hvac_mode(index=0, hvac_mode='cool')

# Set to heating mode
ecobee.set_hvac_mode(index=0, hvac_mode='heat')

# Set to auto (heat/cool as needed)
ecobee.set_hvac_mode(index=0, hvac_mode='auto')

# Turn off HVAC
ecobee.set_hvac_mode(index=0, hvac_mode='off')

# Use auxiliary/emergency heat only
ecobee.set_hvac_mode(index=0, hvac_mode='auxHeatOnly')
```

### Fan Control

```python
# Set fan minimum on time (minutes per hour, 1-60)
ecobee.set_fan_min_on_time(index=0, fan_min_on_time=15)

# Set fan mode with hold
# Fan modes: 'auto', 'on'
ecobee.set_fan_mode(
    index=0,
    fan_mode='on',
    hold_type='nextTransition'
)

# Set fan mode with additional options
ecobee.set_fan_mode(
    index=0,
    fan_mode='on',
    hold_type='holdHours',
    holdHours=2,
    coolHoldTemp=75,  # Optional: set cool temp
    heatHoldTemp=68   # Optional: set heat temp
)
```

### Climate/Comfort Settings

```python
# Set a climate hold (uses predefined comfort settings)
# Common climates: 'home', 'away', 'sleep'

# Set to "Away" mode
ecobee.set_climate_hold(
    index=0,
    climate='away',
    hold_type='nextTransition'
)

# Set to "Home" mode indefinitely
ecobee.set_climate_hold(
    index=0,
    climate='home',
    hold_type='indefinite'
)

# Set to "Sleep" mode for 8 hours
ecobee.set_climate_hold(
    index=0,
    climate='sleep',
    hold_type='holdHours',
    hold_hours=8
)

# Update which sensors participate in a climate
# Using sensor names
ecobee.update_climate_sensors(
    index=0,
    climate_name='Home',
    sensor_names=['Living Room', 'Bedroom']
)

# Using sensor IDs
ecobee.update_climate_sensors(
    index=0,
    climate_name='Sleep',
    sensor_ids=['rs:100:1', 'rs:100:2']
)
```

### Vacation Management

```python
# Create a vacation
ecobee.create_vacation(
    index=0,
    vacation_name='Beach Trip',
    cool_temp=78.0,
    heat_temp=62.0,
    start_date='2024-07-01',
    start_time='08:00:00',
    end_date='2024-07-07',
    end_time='18:00:00',
    fan_mode='auto',        # 'auto' or 'on'
    fan_min_on_time='0'     # 0-60 minutes
)

# Delete a vacation
ecobee.delete_vacation(index=0, vacation='Beach Trip')
```

### Humidity Control

```python
# Set target humidity level (percentage)
ecobee.set_humidity(index=0, humidity='45')

# Humidifier modes: 'auto', 'manual', 'off'
ecobee.set_humidifier_mode(index=0, humidifier_mode='auto')

# Dehumidifier modes: 'on', 'off'
ecobee.set_dehumidifier_mode(index=0, dehumidifier_mode='on')

# Set dehumidification level (percentage)
ecobee.set_dehumidifier_level(index=0, dehumidifier_level=50)
```

### Ventilator Control

```python
# Ventilator modes: 'auto', 'minontime', 'on', 'off'
ecobee.set_vent_mode(index=0, vent_mode='auto')

# Set minimum ventilator run time (minutes per hour)
ecobee.set_ventilator_min_on_time(index=0, ventilator_min_on_time=20)

# Set ventilator run time when home
ecobee.set_ventilator_min_on_time_home(
    index=0,
    ventilator_min_on_time_home=20
)

# Set ventilator run time when away
ecobee.set_ventilator_min_on_time_away(
    index=0,
    ventilator_min_on_time_away=10
)

# Toggle ventilator timer
ecobee.set_ventilator_timer(index=0, ventilator_on=True)
```

### Remote Sensors

```python
# Get remote sensors for a thermostat
sensors = ecobee.get_remote_sensors(index=0)

for sensor in sensors:
    print(f"Sensor: {sensor['name']}")
    print(f"  ID: {sensor['id']}")
    print(f"  Type: {sensor['type']}")

    for capability in sensor['capability']:
        print(f"  {capability['type']}: {capability['value']}")
```

### Messages

```python
# Send a message to the thermostat display (max 500 characters)
ecobee.send_message(
    index=0,
    message="Hello from Python!"
)

# Default message if none provided
ecobee.send_message(index=0)  # Displays "Hello from pyecobee!"
```

### Advanced Settings

```python
# Enable/disable Smart Home/Away and Follow Me
ecobee.set_occupancy_modes(
    index=0,
    auto_away=True,     # Smart Home/Away
    follow_me=True      # Follow Me (uses sensor occupancy)
)

# Enable/disable daylight saving time
ecobee.set_dst_mode(index=0, enable_dst=True)

# Enable/disable microphone (ecobee4 only)
ecobee.set_mic_mode(index=0, mic_enabled=False)

# Set auxiliary heat cutover threshold (outdoor temp in °F)
# Below this temp, aux heat will be used
ecobee.set_aux_cutover_threshold(index=0, threshold=35)

# Set max outdoor temp for auxiliary heat
# Above this temp, aux heat won't be used
ecobee.set_aux_maxtemp_threshold(index=0, threshold=50)

# Get equipment notifications (requires INCLUDE_NOTIFICATIONS config)
notifications = ecobee.get_equipment_notifications(index=0)
```

---

## API Reference

### Ecobee Class

#### Constructor

```python
Ecobee(config_filename: str = None, config: dict = None)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `config_filename` | str | Path to JSON configuration file |
| `config` | dict | Dictionary containing configuration |

> Note: Provide either `config_filename` or `config`, not both.

#### Authentication Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `request_pin()` | bool | Request authorization PIN |
| `request_tokens()` | bool | Exchange authorization code for tokens |
| `refresh_tokens()` | bool | Refresh expired access token |

#### Configuration Methods

| Method | Description |
|--------|-------------|
| `read_config_from_file()` | Read configuration from the specified file |

#### Data Retrieval Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `get_thermostats()` | bool | Fetch all thermostats and cache in `self.thermostats` |
| `update()` | bool | Alias for `get_thermostats()` |
| `get_thermostat(index)` | dict | Get single thermostat by index |
| `get_remote_sensors(index)` | list | Get remote sensors for thermostat |
| `get_equipment_notifications(index)` | list | Get equipment notifications |

#### Control Methods

| Method | Description |
|--------|-------------|
| `set_hvac_mode(index, hvac_mode)` | Set HVAC mode |
| `set_hold_temp(index, cool_temp, heat_temp, ...)` | Set temperature hold |
| `set_climate_hold(index, climate, ...)` | Set climate/comfort hold |
| `set_fan_mode(index, fan_mode, hold_type, ...)` | Set fan mode |
| `set_fan_min_on_time(index, fan_min_on_time)` | Set fan min run time |
| `resume_program(index, resume_all)` | Resume scheduled program |
| `create_vacation(index, vacation_name, ...)` | Create vacation event |
| `delete_vacation(index, vacation)` | Delete vacation event |
| `set_humidity(index, humidity)` | Set humidity setpoint |
| `set_humidifier_mode(index, humidifier_mode)` | Set humidifier mode |
| `set_dehumidifier_mode(index, dehumidifier_mode)` | Set dehumidifier mode |
| `set_dehumidifier_level(index, dehumidifier_level)` | Set dehumidifier level |
| `set_vent_mode(index, vent_mode)` | Set ventilator mode |
| `set_ventilator_min_on_time(index, ...)` | Set ventilator min time |
| `set_ventilator_min_on_time_home(index, ...)` | Set ventilator min time when home |
| `set_ventilator_min_on_time_away(index, ...)` | Set ventilator min time when away |
| `set_ventilator_timer(index, ventilator_on)` | Toggle ventilator timer |
| `set_occupancy_modes(index, auto_away, follow_me)` | Set smart features |
| `set_dst_mode(index, enable_dst)` | Toggle daylight saving |
| `set_mic_mode(index, mic_enabled)` | Toggle microphone |
| `set_aux_cutover_threshold(index, threshold)` | Set aux heat cutover threshold |
| `set_aux_maxtemp_threshold(index, threshold)` | Set aux heat max temp threshold |
| `send_message(index, message)` | Send message to display |
| `update_climate_sensors(index, climate_name, ...)` | Update climate sensors |

---

## Error Handling

The library defines custom exceptions for common error conditions:

```python
from pyecobee import Ecobee, ExpiredTokenError, InvalidTokenError, InvalidSensorError

ecobee = Ecobee(config=your_config)

try:
    ecobee.set_hold_temp(0, 72.0, 68.0)
except ExpiredTokenError:
    # Access token expired - refresh and retry
    if ecobee.refresh_tokens():
        ecobee.set_hold_temp(0, 72.0, 68.0)
except InvalidTokenError:
    # Tokens are invalid - re-authorization required
    print("Please re-authorize the application")
except InvalidSensorError:
    # Sensor not found on thermostat
    print("The specified sensor was not found")
```

### Exception Types

| Exception | Description |
|-----------|-------------|
| `EcobeeError` | Base exception for all ecobee errors |
| `ExpiredTokenError` | Access token has expired |
| `InvalidTokenError` | Token is invalid, re-auth required |
| `InvalidSensorError` | Specified sensor not found |

---

## Integration with Home Assistant

This library is used by [Home Assistant](https://www.home-assistant.io/) for the official Ecobee integration. Here's a simplified example based on how Home Assistant uses it:

```python
from pyecobee import Ecobee, ECOBEE_API_KEY, ECOBEE_REFRESH_TOKEN, ExpiredTokenError

class EcobeeData:
    """Manage Ecobee data and token refresh."""

    def __init__(self, api_key: str, refresh_token: str):
        self.ecobee = Ecobee(config={
            ECOBEE_API_KEY: api_key,
            ECOBEE_REFRESH_TOKEN: refresh_token,
        })

    def update(self):
        """Update thermostat data with automatic token refresh."""
        try:
            self.ecobee.update()
        except ExpiredTokenError:
            self.refresh()
            self.ecobee.update()

    def refresh(self) -> bool:
        """Refresh tokens and return success status."""
        if self.ecobee.refresh_tokens():
            # Save updated tokens to persistent storage
            new_config = self.ecobee.config
            # save_to_database(new_config)
            return True
        return False

# Usage
data = EcobeeData(api_key='your-key', refresh_token='your-refresh-token')
data.update()

if data.ecobee.thermostats:
    for i, thermostat in enumerate(data.ecobee.thermostats):
        current_temp = thermostat['runtime']['actualTemperature'] / 10
        print(f"{thermostat['name']}: {current_temp}°F")

        # Control operations
        data.ecobee.set_hvac_mode(i, 'cool')
        data.ecobee.set_hold_temp(i, 72.0, 68.0)
```

---

## Temperature Notes

All temperatures in the Ecobee API are stored as integers representing tenths of a degree Fahrenheit:

- **Reading temperatures**: Divide by 10 to get the actual temperature
  ```python
  actual_temp = thermostat['runtime']['actualTemperature'] / 10  # e.g., 720 -> 72.0°F
  ```

- **Setting temperatures**: The library handles conversion automatically
  ```python
  ecobee.set_hold_temp(0, cool_temp=72.0, heat_temp=68.0)  # Pass as floats
  ```

---

## Logging

The library uses Python's standard logging module. To enable debug logging:

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger('pyecobee')
logger.setLevel(logging.DEBUG)
```

---

## License

MIT License

Copyright (c) 2017 Nolan Gilley

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

## Links

- [Ecobee Developer Portal](https://www.ecobee.com/home/developer/api/introduction/index.shtml)
- [Ecobee API Documentation](https://www.ecobee.com/home/developer/api/documentation/v1/index.shtml)
- [Home Assistant Ecobee Integration](https://www.home-assistant.io/integrations/ecobee/)
