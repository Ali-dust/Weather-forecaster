<script setup>
import { computed } from 'vue';

const props = defineProps({
  forecast: Object // This is now the 'daily' object
});

// Helper function to interpret Open-Meteo's weather codes
// This is a simplified version. A real app would have more codes.
const getWeatherDisplay = (code) => {
  const codeMap = {
    0: { text: 'Clear Sky', icon: '☀️' },
    1: { text: 'Mainly Clear', icon: '🌤️' },
    2: { text: 'Partly Cloudy', icon: '🌥️' },
    3: { text: 'Overcast', icon: '☁️' },
    45: { text: 'Fog', icon: '🌫️' },
    51: { text: 'Light Drizzle', icon: '🌦️' },
    61: { text: 'Light Rain', icon: '🌦️' },
    63: { text: 'Rain', icon: '🌧️' },
    65: { text: 'Heavy Rain', icon: '🌧️' },
    80: { text: 'Light Showers', icon: '🌧️' },
    95: { text: 'Thunderstorm', icon: '⛈️' },
  };
  return codeMap[code] || { text: 'Unknown', icon: '🤷' };
};

const getDayOfWeek = (dateString) => {
  const options = { weekday: 'short' };
  const date = new Date(dateString + 'T00:00:00');
  return date.toLocaleDateString('en-US', options);
};

// This is the professional way to handle this data structure.
// We use a 'computed' property to transform the prop data.
const formattedDays = computed(() => {
  const { 
    time, 
    weather_code, 
    temperature_2m_max, 
    temperature_2m_min 
  } = props.forecast;

  // "Zip" the arrays together into an array of objects
  return time.map((date, index) => {
    return {
      date: date,
      dayName: getDayOfWeek(date),
      weather: getWeatherDisplay(weather_code[index]),
      high: Math.round(temperature_2m_max[index]),
      low: Math.round(temperature_2m_min[index]),
    };
  });
});
</script>

<template>
  <div class="forecast-container">
    <h2>7-Day Forecast</h2>
    <ul class="forecast-list">
      <li 
        v-for="day in formattedDays" 
        :key="day.date" 
        class="forecast-item"
      >
        <h3>{{ day.dayName }}</h3>
        
        <span class="icon">{{ day.weather.icon }}</span>
        
        <p class="day-condition">{{ day.weather.text }}</p>
        
        <div class="temps">
          <span class="high">High: {{ day.high }}°C</span>
          <span class="low">Low: {{ day.low }}°C</span>
        </div>
      </li>
    </ul>
  </div>
</template>

<style scoped>
.forecast-container h2 {
  text-align: center;
  margin-bottom: 1.5rem;
}

.forecast-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(130px, 1fr));
  gap: 1rem;
  padding: 0;
  list-style: none;
}

.forecast-item {
  background: #fff;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 1rem;
  text-align: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.03);
}

.forecast-item h3 {
  margin: 0 0 0.5rem 0;
  font-size: 1rem;
  color: #333;
}

.icon {
  font-size: 2.5rem; /* Emoji icon */
  line-height: 1;
  margin-bottom: 0.5rem;
}

.day-condition {
  font-size: 0.85rem;
  color: #666;
  min-height: 2.5em;
  margin-bottom: 0.5rem;
}

.temps {
  font-size: 0.9rem;
}

.high {
  font-weight: bold;
  margin-right: 0.5rem;
}

.low {
  color: #555;
}

@media (max-width: 600px) {
  .forecast-list {
    /* Change from a flexible grid to a single column */
    grid-template-columns: 1fr;
    gap: 0.75rem;
  }
  
  .forecast-item {
    /* Bonus: Let's make the items horizontal to save space */
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0.5rem 1rem;
  }

  .forecast-item h3 { /* Day name */
    margin: 0;
    flex: 1; /* Give it space */
  }

  .icon {
    font-size: 2rem; /* Smaller icon */
    margin: 0 1rem;
  }

  .day-condition {
     display: none; /* Hide text, icon is enough */
  }

  .temps {
    text-align: right;
  }
}
</style>