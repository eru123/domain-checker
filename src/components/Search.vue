<script setup>
import { ref, watch } from 'vue'
import { vendors, domainChecker as check } from '@skiddph/domain-checker'

const emit = defineEmits([ 'loading', 'state' ])

// primary
const search = ref("")
const vendor = ref(vendors[ 0 ])

// check vars
const loading = ref(false)
const error = ref(false)
const success = ref(false)
const message = ref("")
const website = ref("")
const title = ref("")
const result = ref([])

watch(loading, (l) => emit('loading', l))

function clearQuery() {
  search.value = ""
  emit('state', {
    error: false
  })
}

watch([ error, success ], () => {
  emit('state', {
    error,
    success,
    search,
    vendor,
    result,
    website,
    title,
    message
  })
})

const sbH = async () => {
  if (search.value.trim().length < 1) {
    success.value = false
    error.value = true
    message.value = "Search query must not be empty"
    return
  }

  try {
    loading.value = true
    return await check(search.value)[ vendor.value ]({ cors: "https://cors-anywhere.herokuapp.com/" })
      .then(e => {
        if (e.success) {
          success.value = true
          error.value = false
          result.value = e.result
          website.value = e.website
          title.value = e.title
        } else if (e.error) {
          error.value = true
          success.value = false
          message.value = e.message
        } else throw new Error()
      })
      .catch(() => {
        error.value = true
        success.value = false
        message.value = "Unknown error occurred"
      })
      .finally(() => loading.value = false)
  } catch (e) {
    error.value = true
    success.value = false
    message.value = "Unknown error occurred"
    loading.value = false
  }
}

</script>
<template>
  <div class="w">
    <div action="#" class="c">
      <div class="s">
        <input type="text" placeholder="example.com or example" v-model="search" title="Search query text input" />
        <icon icon="x" @click="clearQuery" />
      </div>
      <div class="a">
        <select v-model="vendor" title="vendors">
          <option
            v-for="v in vendors"
            :value="v"
            :key="v"
          >{{ v.charAt(0).toUpperCase() + v.slice(1) }}</option>
        </select>
        <button @click="sbH" title="Search button">
          <span>Search</span>
          <icon icon="magnifying-glass" />
        </button>
      </div>
    </div>
  </div>
</template>
<style lang="scss" scoped>
@import "@/styles/variables.scss";
.w {
  @apply w-full flex flex-row items-center justify-center;
  .c {
    @apply w-full flex flex-col md:flex-row  rounded-lg;
    @apply md:rounded-full transition-all duration-200 mb-4;
    max-width: $max-width;

    .s {
      @apply flex-1 flex flex-row items-center justify-center mb-4 md:mb-0 md:mr-2 py-1 md:py-0 pl-6 pr-2;
      @apply border border-gray-300 rounded-full;
      input[type="text"] {
        @apply flex-1 bg-inherit outline-none text-base p-0 m-0;
      }
      svg {
        @apply py-2 px-4 text-gray-400 cursor-pointer hover:text-gray-700 transition-all duration-200;
      }
    }

    .a {
      @apply md:w-full flex flex-row items-center justify-center;
      flex: 0;

      select {
        @apply bg-inherit rounded-lg md:rounded-full px-2 py-2 mr-2 text-base;
        @apply border border-gray-300;
        @apply text-gray-400;
        @apply hover:text-gray-700 focus:text-gray-700 active:text-gray-700;
        @apply transition-all duration-200 outline-none cursor-pointer;
      }

      button {
        @apply flex flex-row items-center justify-center;
        @apply bg-blue-500 text-blue-100 py-2 px-4 text-base rounded-lg md:rounded-full;
        @apply hover:bg-blue-700 transition-all duration-200 outline-none hover:text-blue-200;
        span {
          @apply pr-2 py-0 text-sm;
        }

        svg {
          @apply text-sm;
        }
      }
    }
  }
}

.dark {
  .w {
    .c {
      .s {
        @apply border-blue-400 focus-within:border-blue-200;
        input[type="text"] {
          @apply text-blue-400;

          &:focus {
            @apply text-blue-200
          }
        }
        svg {
          @apply text-blue-400 hover:text-blue-200;
        }
      }

      .a {
        select {
          @apply border-blue-400 text-blue-400 hover:text-blue-200;
          @apply focus:text-blue-200 active:text-blue-200 bg-gray-900;
          @apply hover:border-blue-200 active:border-blue-200 focus:border-blue-200;
        }

        button {
          @apply bg-blue-400 text-gray-900;
          @apply hover:bg-blue-200;
          @apply focus:bg-blue-200;
        }
      }
    }
  }
}
</style>
