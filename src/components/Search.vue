<script setup>
    import {ref} from 'vue';
    import {useRoute} from 'vue-router';

    const route = useRoute();
    const query = ref('');

    const handleSearch = () => {
      if (!query.value.trim()) return;

      route.push({
        path: '/jobs',
        query: { q: query.value.trim() }
      });

      query.value = '';
};
</script>

<template>
    <form @submit.prevent="handleSearch" class="ml-auto">
        <input
          type="text"
          v-model="query"
          placeholder="Search jobs..."
          class="px-3 py-2 rounded-md border border-gray-300 focus:outline-none focus:ring-2 focus:ring-green-500"
        />
        <button
          type="submit"
          class="ml-2 px-3 py-2 bg-green-500 text-white rounded-md hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-500"
        >
          Search
        </button>
    </form>
</template>