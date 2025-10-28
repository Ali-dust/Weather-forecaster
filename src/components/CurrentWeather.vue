<script setup>
const props = defineProps({
  location: Object,
  current: Object
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

const weather = getWeatherDisplay(props.current.weather_code);
</script>

<template>
  <div class="current-weather-card">
    <h2>
      Current Weather in 
      {{ location.city }}, 
      {{ location.country }}
    </h2>
    
    <div class="details">
      <div class="temp-condition">
        <span class="icon">{{ weather.icon }}</span>
        
        <span class="temp">
          {{ Math.round(current.temperature_2m) }}°C
        </span>
      </div>
      
      <div class="meta">
        <p class="condition-text">{{ weather.text }}</p>
        
        <p>Wind: {{ current.wind_speed_10m }} km/h</p>
        
        <p>
          As of: 
          {{ new Date(current.time).toLocaleTimeString('en-US') }}
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Updated styles for emoji icon */
.current-weather-card {
  background: #f4f8ff;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 2rem;
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
}

h2 {
  margin-top: 0;
  color: #333;
}

.details {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 2rem;
}

.temp-condition {
  display: flex;
  align-items: center;
}

.icon {
  font-size: 3.5rem; /* Bigger emoji */
  margin-right: 1rem;
  line-height: 1;
}

.temp {
  font-size: 2.5rem;
  font-weight: bold;
  color: #007bff;
}

.meta {
  font-size: 0.95rem;
  color: #555;
}

.meta p {
  margin: 0.25rem 0;
}

.condition-text {
  font-weight: bold;
  font-size: 1.1rem;
  margin-bottom: 0.5rem;
}
@media (max-width: 600px) {
  .details {
    flex-direction: column; /* Force stacking */
    align-items: flex-start; /* Align to the start */
    gap: 1rem;
  }

  .icon {
    font-size: 3rem; /* Make icon a bit smaller */
  }

  .temp {
    font-size: 2.25rem; /* Make temp a bit smaller */
  }

  .meta {
    width: 100%; /* Ensure it takes full width */
  }
}
</style>