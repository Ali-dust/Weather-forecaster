<script setup>
import { ref } from 'vue';

// 1. defineEmits: Declares the event this component can send.
const emit = defineEmits(['search-city']);

// 2. ref: Holds the internal state of the input field.
const city = ref('');

// 3. submitHandler: This function is called on form submission.
const submitHandler = () => {
  // 3a. Basic validation
  if (!city.value.trim()) {
    alert('Please enter a city name.');
    return;
  }
  
  // 3b. Emits the event up to the parent.
  // The parent will be listening for '@search-city'.
  emit('search-city', city.value);
  
  // 3c. Clear the input after search
  city.value = '';
};
</script>

<template>
  <form class="search-form" @submit.prevent="submitHandler">
    <input 
      type="text" 
      v-model="city" 
      placeholder="Search for a city..."
    />
    <button type="submit">Search</button>
  </form>
</template>

<style scoped>
/* 'scoped' means these styles ONLY apply to this component */
.search-form {
  display: flex;
  margin-bottom: 2rem;
}

input[type="text"] {
  flex-grow: 1;
  padding: 0.75rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px 0 0 4px;
}

button {
  padding: 0.75rem 1.5rem;
  font-size: 1rem;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
  transition: background-color 0.2s;
}

button:hover {
  background-color: #0056b3;
}
</style>