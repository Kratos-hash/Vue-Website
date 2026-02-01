<script setup>
import { RouterLink } from 'vue-router';
import JobListing from './JobListing.vue';
import { reactive, ref, defineProps, onMounted } from 'vue';
import PulseLoader from 'vue-spinner/src/PulseLoader.vue';
import axios from 'axios';

defineProps({
  limit: Number,
  showButton: {
    type: Boolean,
    default: false
  }
});

const state = reactive({
  jobs: [],
  isLoading: true,
  message: ''
});

const email = ref('');

/* Fetch jobs */
onMounted(async () => {
  try {
    const res = await axios.get('/api/jobs');
    state.jobs = res.data;
  } catch (error) {
    console.error('Error fetching jobs', error);
  } finally {
    state.isLoading = false;
  }
});

/* Subscribe function */
const subscribe = async () => {
  if (!email.value) return;

  try {
    await axios.post('/api/subscribe', {
      email: email.value
    });
    state.message = 'Subscription successful!';
    email.value = '';
  } catch (error) {
    console.error('Error subscribing', error);
    state.message = 'Subscription failed. Please try again.';
  }
};
</script>
<template>
  <section class="bg-blue-50 px-4 py-10">
    <div class="container-xl lg:container m-auto">
      <h2 class="text-3xl font-bold text-green-500 mb-6 text-center">
        Browse Jobs
      </h2>

      <div v-if="state.isLoading" class="text-center py-6">
        <PulseLoader />
      </div>

      <div v-else class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <JobListing
          v-for="job in state.jobs.slice(0, limit || state.jobs.length)"
          :key="job.id"
          :job="job"
        />
      </div>
    </div>
  </section>

  <section v-if="showButton" class="m-auto max-w-lg my-10 px-6">
    <RouterLink
      to="/jobs"
      class="block bg-black text-white text-center py-4 px-6 rounded-xl hover:bg-gray-700"
    >
      View All Jobs
    </RouterLink>
  </section>

  <!-- SUBSCRIBE FORM -->
  <section class="m-auto max-w-lg my-10 px-6">
    <form-card bg="bg-green-100">
      <h2 class="text-2xl font-bold">Subscribe to our Newsletter</h2>
      <p class="mt-2 mb-4">
        Stay updated with the latest Vue job postings
      </p>

      <form @submit.prevent="subscribe" class="flex gap-2">
        <input
          v-model="email"
          type="email"
          required
          placeholder="Enter your email"
          class="flex-grow px-3 py-2 rounded-md border focus:ring-2 focus:ring-green-500"
        />

        <button
          type="submit"
          class="bg-green-500 text-white rounded-md px-4 py-2 hover:bg-green-600"
        >
          Subscribe
        </button>
      </form>

      <p v-if="state.message" class="mt-3 text-sm text-green-700">
        {{ state.message }}
      </p>
    </form-card>
  </section>
</template>
