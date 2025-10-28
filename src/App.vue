<script setup>
import { ref, onMounted } from 'vue';
import TheSearch from './components/TheSearch.vue';
import CurrentWeather from './components/CurrentWeather.vue';
import WeatherForecast from './components/WeatherForecast.vue';
import AppSpinner from './components/AppSpinner.vue';

// --- State Management ---
// We need to store location and weather data separately now
const locationData = ref(null);
const weatherData = ref(null);
const loading = ref(false);
const error = ref(null);

// --- API Logic ---

/**
 * This is the main function that gets weather from Open-Meteo.
 * It's reusable for both IP lookup and search.
 */
const fetchWeather = async (lat, lon) => {
  loading.value = true;
  error.value = null;
  weatherData.value = null;

  try {
    // Open-Meteo URL. We ask for specific daily and current weather data.
    // 'current' for current, 'daily' for the 7-day forecast.
    const apiUrl = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,weather_code,wind_speed_10m&daily=weather_code,temperature_2m_max,temperature_2m_min&timezone=auto`;

    const response = await fetch(apiUrl);
    
    if (!response.ok) {
      throw new Error('Could not fetch weather data.');
    }

    weatherData.value = await response.json();
    
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

/**
 * 1. This function runs on load to get location from IP.
 */
const fetchWeatherForIP = async () => {
  loading.value = true;
  error.value = null;
  locationData.value = null;

  try {
    // 1a. Call the keyless IP API
    // IMPORTANT: ip-api.com's free endpoint is HTTP. 
    // If your GitHub page is HTTPS, this may be blocked.
    // For a real project, you'd use their paid HTTPS endpoint.
    // For this learning project, we use their free HTTPS proxy.
    const locResponse = await fetch(`https://freeipapi.com/api/json/`);
    
    if (!locResponse.ok) {
      throw new Error('Could not fetch IP location.');
    }
    
    const locData = await locResponse.json();
    
    // Store location for display
    locationData.value = {
      city: locData.cityName,
      country: locData.countryName,
    };
    
    // 1b. Now, use the location to call the weather API
    await fetchWeather(locData.latitude, locData.longitude);

  } catch (err) {
    error.value = err.message;
    loading.value = false;
  }
};

/**
 * 2. This function handles the search event.
 * It must first convert a city name to coordinates.
 */
const handleSearch = async (city) => {
  loading.value = true;
  error.value = null;
  weatherData.value = null;
  locationData.value = null;
  
  try {
    // 2a. Use Open-Meteo's *geocoding* API to find the city
    const geoApiUrl = `https://geocoding-api.open-meteo.com/v1/search?name=${city}&count=1&language=en&format=json`;
    
    const geoResponse = await fetch(geoApiUrl);
    if (!geoResponse.ok) {
      throw new Error('Could not find location.');
    }
    
    const geoData = await geoResponse.json();
    
    if (!geoData.results || geoData.results.length === 0) {
      throw new Error(`City not found: ${city}`);
    }
    
    const loc = geoData.results[0];
    
    // Store location for display
    locationData.value = {
      city: loc.name,
      country: loc.country,
    };

    // 2b. Now, use the coordinates to call the weather API
    await fetchWeather(loc.latitude, loc.longitude);

    localStorage.setItem('lastCity', loc.name);
    
  } catch (err) {
    error.value = err.message;
    loading.value = false;
  }
};

// --- Lifecycle Hook ---
onMounted(() => {
  // 4. Check for a saved city first
  const savedCity = localStorage.getItem('lastCity'); // <-- NEW
  
  if (savedCity) { // <-- NEW
    // If we have a saved city, just search for it
    handleSearch(savedCity); // <-- NEW
  } else {
    // 5. If no saved city, use geolocation as the default
    if (!("geolocation" in navigator)) {
      error.value = "Geolocation is not supported. Please use the search bar.";
      return;
    }

    loading.value = true;
    
    navigator.geolocation.getCurrentPosition(
      // 5a. SUCCESS callback
      async (position) => {
        try {
          const { latitude, longitude } = position.coords;
          
          await Promise.all([
            fetchWeather(latitude, longitude),
            fetchLocationName(latitude, longitude)
          ]);
          
        } catch (err) {
          error.value = err.message;
        } finally {
          loading.value = false;
        }
      },
      // 5b. ERROR callback
      (err) => {
        loading.value = false;
        if (err.code === err.PERMISSION_DENIED) {
          error.value = "Location access denied. Please use the search bar to find a city.";
        } else {
          error.value = "Could not get your location. Please use the search bar.";
        }
        console.error("Geolocation error:", err);
      }
    );
  }
});
</script>

<template>
  <div id="app-container">
    <header>
      <h1>Vue Weather Dashboard</h1>
    </header>
    
    <main>
      <TheSearch @search-city="handleSearch" />

      <AppSpinner v-if="loading" />

      <div v-if="error" class="error-message">
        <p>Sorry, an error occurred: {{ error }}</p>
      </div>
      
      <div v-if="weatherData && locationData" class="weather-results">
        <CurrentWeather 
          :location="locationData" 
          :current="weatherData.current"
        />
        
        <WeatherForecast 
          :forecast="weatherData.daily"
        />
      </div>
    </main>
  </div>
</template>

<style scoped>
/* Styles remain exactly the same as before */
#app-container {
  max-width: 900px;
  margin: 0 auto;
  padding: 1.5rem;
}

header {
  text-align: center;
  margin-bottom: 2rem;
  color: #2c3e50;
}

.error-message {
  background: #ffebee;
  color: #c62828;
  border: 1px solid #c62828;
  border-radius: 4px;
  padding: 1rem;
  text-align: center;
}

@media (max-width: 600px) {
  #app-container {
    padding: 0.75rem; /* Less padding on small screens */
  }

  header h1 {
    font-size: 1.75rem; /* Smaller title */
  }
}

</style>