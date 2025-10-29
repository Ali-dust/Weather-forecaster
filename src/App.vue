<script setup>
import { ref, onMounted } from 'vue';
import TheSearch from './components/TheSearch.vue';
import CurrentWeather from './components/CurrentWeather.vue';
import WeatherForecast from './components/WeatherForecast.vue';
import AppSpinner from './components/AppSpinner.vue';

// --- State Management ---
const locationData = ref(null);
const weatherData = ref(null);
const loading = ref(false);
const error = ref(null);

// --- API Logic ---

/**
 * 1. Fetches weather from Open-Meteo.
 * (This function is correct)
 */
const fetchWeather = async (lat, lon) => {
  loading.value = true; // Set loading true *inside* this function
  error.value = null;
  weatherData.value = null;

  try {
    const apiUrl = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,weather_code,wind_speed_10m&daily=weather_code,temperature_2m_max,temperature_2m_min&timezone=auto`;
    const response = await fetch(apiUrl);
    
    if (!response.ok) {
      throw new Error('Could not fetch weather data.');
    }
    weatherData.value = await response.json();
  } catch (err) {
    // Let the calling function know an error happened
    throw new Error(`Weather API error: ${err.message}`);
  }
  // Note: We don't set loading.value = false here
  // We let the function *calling* this one decide when loading is fully done
};

/**
 * 2. NEW: Fetches a location name from lat/lon.
 * (THIS WAS THE MISSING FUNCTION)
 */
const fetchLocationName = async (lat, lon) => {
  try {
    const geoApiUrl = `https://geocoding-api.open-meteo.com/v1/reverse?latitude=${lat}&longitude=${lon}&format=json`;
    const response = await fetch(geoApiUrl);
    if (!response.ok) {
      throw new Error('Could not fetch location name.');
    }
    const data = await response.json();
    
    locationData.value = {
      city: data.city || data.name || 'Unknown Location',
      country: data.country || '',
    };
  } catch (err) {
    throw new Error(`Geocoding API error: ${err.message}`);
  }
};


/**
 * 3. This function handles the search event.
 * (This function is correct)
 */
const handleSearch = async (city) => {
  loading.value = true;
  error.value = null;
  weatherData.value = null;
  locationData.value = null;
  
  try {
    // 3a. Use Open-Meteo's *geocoding* API to find the city
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

    // 3b. Now, use the coordinates to call the weather API
    await fetchWeather(loc.latitude, loc.longitude);

    // 3c. Save to local storage
    localStorage.setItem('lastCity', loc.name);
    
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

// --- Lifecycle Hook ---
// (This is correct, and will now work)
onMounted(() => {
  const savedCity = localStorage.getItem('lastCity');
  
  if (savedCity) {
    handleSearch(savedCity);
  } else {
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
          
          // This Promise.all will now work
          await Promise.all([
            fetchWeather(latitude, longitude),
            fetchLocationName(latitude, longitude) // <-- This function now exists!
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
D      }
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
