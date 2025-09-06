## Code
```python
# --- Subject Interface ---
class Subject:
    def register_observer(self, observer):
        raise NotImplementedError

    def remove_observer(self, observer):
        raise NotImplementedError

    def notify_observers(self):
        raise NotImplementedError


# --- Observer Interface ---
class Observer:
    def update(self, temperature, humidity, pressure):
        raise NotImplementedError


# --- Concrete Subject (WeatherData) ---
class WeatherData(Subject):
    def __init__(self):
        self.observers = []
        self.temperature = 0
        self.humidity = 0
        self.pressure = 0

    def register_observer(self, observer):
        self.observers.append(observer)

    def remove_observer(self, observer):
        self.observers.remove(observer)

    def notify_observers(self):
        for observer in self.observers:
            observer.update(self.temperature, self.humidity, self.pressure)

    def set_measurements(self, temperature, humidity, pressure):
        self.temperature = temperature
        self.humidity = humidity
        self.pressure = pressure
        self.notify_observers()


# --- Concrete Observers ---
class CurrentConditionsDisplay(Observer):
    def __init__(self, weather_data):
        self.temperature = 0
        self.humidity = 0
        self.weather_data = weather_data
        weather_data.register_observer(self)

    def update(self, temperature, humidity, pressure):
        self.temperature = temperature
        self.humidity = humidity
        self.display()

    def display(self):
        print(f"Current conditions: {self.temperature}°C and {self.humidity}% humidity")


class StatisticsDisplay(Observer):
    def __init__(self, weather_data):
        self.temperatures = []
        self.weather_data = weather_data
        weather_data.register_observer(self)

    def update(self, temperature, humidity, pressure):
        self.temperatures.append(temperature)
        self.display()

    def display(self):
        avg_temp = sum(self.temperatures) / len(self.temperatures)
        print(f"Avg/Max/Min temperature: {avg_temp:.1f}/{max(self.temperatures)}/{min(self.temperatures)}")


class ForecastDisplay(Observer):
    def __init__(self, weather_data):
        self.last_pressure = None
        self.current_pressure = 0
        self.weather_data = weather_data
        weather_data.register_observer(self)

    def update(self, temperature, humidity, pressure):
        self.last_pressure = self.current_pressure
        self.current_pressure = pressure
        self.display()

    def display(self):
        if self.last_pressure is None:
            print("Forecast: More data needed...")
        elif self.current_pressure > self.last_pressure:
            print("Forecast: Improving weather on the way!")
        elif self.current_pressure == self.last_pressure:
            print("Forecast: More of the same")
        else:
            print("Forecast: Watch out for cooler, rainy weather!")


# --- Test Drive ---
if __name__ == "__main__":
    weather_data = WeatherData()

    current_display = CurrentConditionsDisplay(weather_data)
    statistics_display = StatisticsDisplay(weather_data)
    forecast_display = ForecastDisplay(weather_data)

    print("---- First Update ----")
    weather_data.set_measurements(26, 65, 1013)

    print("\n---- Second Update ----")
    weather_data.set_measurements(28, 70, 1012)

    print("\n---- Third Update ----")
    weather_data.set_measurements(22, 90, 1014)

```

## Output
```
---- First Update ----
Current conditions: 26°C and 65% humidity
Avg/Max/Min temperature: 26.0/26/26
Forecast: More data needed...

---- Second Update ----
Current conditions: 28°C and 70% humidity
Avg/Max/Min temperature: 27.0/28/26
Forecast: Watch out for cooler, rainy weather!

---- Third Update ----
Current conditions: 22°C and 90% humidity
Avg/Max/Min temperature: 25.3/28/22
Forecast: Improving weather on the way!

```

## Example
1. Websocket
2. Redux
