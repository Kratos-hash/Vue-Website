<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import { useRoute } from 'vue-router';

// Example job data (replace with API fetch if needed)
const jobs = ref([]);
const loading = ref(false);
const error = ref(null);

const route = useRoute();

const fetchJobs = async (query = '') => {
  loading.value = true;
  error.value = null;

  try {
    const url = query
      ? `/api/jobs?search=${encodeURIComponent(query)}`
      : `/api/jobs`;

    const response = await fetch(url);

    if (!response.ok) {
      throw new Error('Failed to fetch jobs');
    }

    jobs.value = await response.json();
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

// Initial fetch
onMounted(() => {
  fetchJobs(route.query.q || '');
});

// Re-fetch when search query changes
watch(
  () => route.query.q,
  (newQuery) => {
    fetchJobs(newQuery || '');
  }
);

const searchQuery = ref(route.query.q || '');

// Watch route changes (important when searching multiple times)
watch(
  () => route.query.q,
  (newQuery) => {
    searchQuery.value = newQuery || '';
  }
);

// Filter logic
const filteredJobs = computed(() => {
  if (!searchQuery.value) return jobs.value;

  return jobs.value.filter(job =>
    job.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
    job.company.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});
</script>

<template>
  <div class="max-w-7xl mx-auto px-4 py-6">
    <h1 class="text-2xl font-bold mb-4">Job Listings</h1>

    <p v-if="route.query.q" class="mb-4 text-gray-600">
      Showing results for "<strong>{{ route.query.q }}</strong>"
    </p>

    <div v-if="loading" class="text-gray-500">
      Loading jobs...
    </div>

    <div v-else-if="error" class="text-red-600">
      {{ error }}
    </div>

    <div v-else-if="jobs.length">
      <div
        v-for="job in jobs"
        :key="job.id"
        class="border p-4 rounded mb-3"
      >
        <h2 class="text-lg font-semibold">{{ job.title }}</h2>
        <p class="text-gray-500">{{ job.company }}</p>
      </div>
    </div>

    <p v-else class="text-gray-500">
      No jobs found.
    </p>
  </div>
</template>

