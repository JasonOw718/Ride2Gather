<template>
<RightNavbar />

  <div class="bg-white flex flex-col px-6 pt-[max(2.5rem,env(safe-area-inset-top))] pb-[max(env(safe-area-inset-bottom),2.5rem)]" style="max-width: 420px; margin: 0 auto;">
    <!-- Back Button -->
    <button
      class="absolute top-6 left-6 w-10 h-10 flex items-center justify-center rounded-lg bg-[#F5F5F5] z-20 border border-gray-200"
      @click="$router.back()"
      aria-label="Back"
    >
      <span class="w-6 h-6 text-primary">
        <font-awesome-icon icon="fa-arrow-left" class="text-[#C77DFF]" />
      </span>
    </button>
    
    <!-- Main Title -->
    <div class="text-2xl font-bold text-left mb-8 text-[#000000] mt-10">Find a Ride</div>

    <!-- Where are you going? -->
    <div class="mb-6">
      <div class="text-base font-semibold text-[#333333] mb-2 text-left">Where are you going?</div>
      <input
        v-model="from"
        type="text"
        placeholder="From"
        class="w-full border border-gray-200 bg-[#F5F5F5] px-4 py-3 mb-3 text-base rounded-lg text-black focus:outline-none focus:ring-2 focus:ring-[#C77DFF] text-left"
      />
      <input
        v-model="to"
        type="text"
        placeholder="To"
        class="w-full border border-gray-200 bg-[#F5F5F5] px-4 py-3 text-base rounded-lg text-black focus:outline-none focus:ring-2 focus:ring-[#C77DFF] text-left"
      />
    </div>

    <!-- Seat needed? (Custom Counter Input) -->
    <div class="mb-8">
      <div class="text-base font-semibold text-[#333333] mb-2 text-left">Seat needed?</div>
      <div class="flex items-center gap-4">
        <button @click="decrement" :disabled="seats <= 1 || loading" class="p-2 rounded-full  text-[#000000] disabled:opacity-30">
          <font-awesome-icon icon="fa-solid fa-minus" />
        </button>
        <span class="text-xl font-bold text-[#C77DFF] w-8 text-center">{{ seats }}</span>
        <button @click="increment" :disabled="seats >= 10 || loading" class="p-2 rounded-full  text-[#000000] disabled:opacity-50">
          <font-awesome-icon icon="fa-solid fa-plus" />
        </button>
      </div>
    </div>

    <!-- Search Button -->
    <button
      class="w-full py-3 px-4 rounded-full shadow-md text-base font-bold transition-all duration-300 mb-4 flex items-center justify-center gap-2"
      :class="{
        'bg-[#C77DFF] text-white hover:bg-opacity-90 cursor-pointer': isValid && !loading,
        'bg-gray-300 text-gray-400 cursor-not-allowed': !isValid || loading
      }"
      :disabled="!isValid || loading"
      @click="onSearch"
    >
      <span v-if="!loading">Search</span>
      <span v-else>
        <svg class="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4a4 4 0 00-4 4H4z"></path>
        </svg>
      </span>
    </button>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { usePassengerInputStore } from '../../../stores/passengerInput'
import { useUserStore } from '../../../stores/user'
import RightNavbar from '../rightnavbar.vue'

const userStore = useUserStore()
const from = ref('')
const to = ref('')
const seats = ref(1)
const loading = ref(false)
const router = useRouter()
const passengerInputStore = usePassengerInputStore()

const isValid = computed(() => {
  return (
    from.value.trim() !== '' &&
    to.value.trim() !== '' &&
    seats.value >= 1
  )
})

function increment() {
  if (seats.value < 10) seats.value++
}
function decrement() {
  if (seats.value > 1) seats.value--
}

async function onSearch() {
  if (!isValid.value) return
  loading.value = true
  
  try {
    // Store search parameters
    passengerInputStore.setInput({
      from: from.value,
      to: to.value,
      seats: seats.value
    })
    
    // Navigate to ride list with query parameters
    router.push({ 
      name: 'RideList', 
      query: {
        from: from.value,
        to: to.value,
        seats: seats.value
      }
    })
  } catch (error) {
    console.error('Search error:', error)
  } finally {
    loading.value = false
  }
}
</script>
