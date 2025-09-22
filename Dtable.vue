<script setup>
import { computed, ref, nextTick, onMounted, onBeforeUnmount } from 'vue';
import sorter from './icons/sorter.vue';
import toggleBtn from './icons/switch.vue';

const props = defineProps({
  columns: { type: Array, required: true },
  data: { type: Array, required: true, default: () => [] },
  rowsPerPage: { type: Number, default: 10 },
  showAddBtn: { type: Boolean, default: false },
  btnLabel: { type: String },
});

const openMenuIndex = ref(null);

const toggleMenu = async (i) => {
  openMenuIndex.value = openMenuIndex.value === i ? null : i;
  await nextTick();
};

const menuRefs = ref({});

const setMenuRef = (el, index) => {
  if (el) {
    menuRefs.value[index] = el;
  } else {
    delete menuRefs.value[index];
  }
};

const handleClickOutside = (event) => {
  if (openMenuIndex.value === null) return;

  const currentMenu = menuRefs.value[openMenuIndex.value];
  if (currentMenu && !currentMenu.contains(event.target)) {
    openMenuIndex.value = null;
  }
};

onMounted(() => {
  document.addEventListener('click', handleClickOutside);
});

onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside);
});

//select Functionallity:
const selectedRows = ref([]);

const allSelected = computed({
  get() {
    return (
      paginatedData.value.length > 0 &&
      selectedRows.value.length === paginatedData.value.length
    );
  },
  set(value) {
    if (value) {
      selectedRows.value = paginatedData.value.map((row, index) => index);
    } else {
      selectedRows.value = [];
    }
  },
});

//SEARCH FUNCTIONALITY

const searchQuery = ref('');

const filteredData = computed(() => {
  if (!searchQuery.value) return props.data;

  return props.data.filter((row) =>
    props.columns.some((col) => {
      const value = row[col.key];
      return value
        ?.toString()
        .toLowerCase()
        .includes(searchQuery.value.toLowerCase());
    })
  );
});
//Paginated Data:

const currentPage = ref(1);

const totalPages = computed(() =>
  Math.ceil(filteredData.value.length / props.rowsPerPage)
);

const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * props.rowsPerPage;
  return filteredData.value.slice(start, start + props.rowsPerPage);
});

const goToPage = (p) => {
  if (p >= 1 && p <= totalPages.value) {
    currentPage.value = p;
  }
};

const emit = defineEmits([
  'row-hover',
  'row-leave',
  'row-select',
  'open-dialog',
]);
</script>

<template>
  <div
    class="py-5 px-5 bg-white mt-10 rounded-[8px] overflow-x-auto text-[14px] max-h-screen"
  >
    <div class="flex flex-row items-center justify-end pb-[24px] gap-[30px]">
      <div
        class="flex items-center gap-2 border bg-white border-gray-500 rounded-full px-4 h-[32px] w-[360px] relative"
      >
        <svg
          width="16"
          height="16"
          viewBox="0 0 16 16"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            d="M15.8898 15.0493L11.8588 11.0182C11.7869 10.9463 11.6932 10.9088 11.5932 10.9088H11.2713C12.3431 9.74952 12.9994 8.20272 12.9994 6.49968C12.9994 2.90923 10.0901 0 6.49968 0C2.90923 0 0 2.90923 0 6.49968C0 10.0901 2.90923 12.9994 6.49968 12.9994C8.20272 12.9994 9.74952 12.3431 10.9088 11.2744V11.5932C10.9088 11.6932 10.9495 11.7869 11.0182 11.8588L15.0493 15.8898C15.1961 16.0367 15.4336 16.0367 15.5805 15.8898L15.8898 15.5805C16.0367 15.4336 16.0367 15.1961 15.8898 15.0493ZM6.49968 11.9994C3.45921 11.9994 0.999951 9.54016 0.999951 6.49968C0.999951 3.45921 3.45921 0.999951 6.49968 0.999951C9.54016 0.999951 11.9994 3.45921 11.9994 6.49968C11.9994 9.54016 9.54016 11.9994 6.49968 11.9994Z"
            fill="#515150"
          />
        </svg>

        <input
          type="text"
          v-model="searchQuery"
          class="outline-none text-sm flex-1"
          placeholder="Search..."
        />
      </div>
      <svg
        width="14"
        height="16"
        viewBox="0 0 14 16"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          d="M13.75 2H10.5L9.45 0.6C9.31028 0.413706 9.1291 0.262501 8.92082 0.158359C8.71254 0.0542176 8.48287 0 8.25 0L5.75 0C5.51713 0 5.28746 0.0542176 5.07918 0.158359C4.8709 0.262501 4.68972 0.413706 4.55 0.6L3.5 2H0.25C0.183696 2 0.120107 2.02634 0.0732233 2.07322C0.0263392 2.12011 0 2.1837 0 2.25L0 2.75C0 2.8163 0.0263392 2.87989 0.0732233 2.92678C0.120107 2.97366 0.183696 3 0.25 3H0.840625L1.87812 14.6344C1.91168 15.0074 2.08356 15.3544 2.35998 15.6071C2.63639 15.8598 2.99735 15.9999 3.37188 16H10.6281C11.0027 15.9999 11.3636 15.8598 11.64 15.6071C11.9164 15.3544 12.0883 15.0074 12.1219 14.6344L13.1594 3H13.75C13.8163 3 13.8799 2.97366 13.9268 2.92678C13.9737 2.87989 14 2.8163 14 2.75V2.25C14 2.1837 13.9737 2.12011 13.9268 2.07322C13.8799 2.02634 13.8163 2 13.75 2ZM5.35 1.2C5.39674 1.13808 5.45717 1.08781 5.52656 1.05311C5.59595 1.01842 5.67242 1.00024 5.75 1H8.25C8.32758 1.00024 8.40405 1.01842 8.47344 1.05311C8.54283 1.08781 8.60326 1.13808 8.65 1.2L9.25 2H4.75L5.35 1.2ZM11.125 14.5437C11.1147 14.6683 11.0579 14.7845 10.9658 14.869C10.8737 14.9536 10.7531 15.0003 10.6281 15H3.37188C3.24686 15.0003 3.12631 14.9536 3.03422 14.869C2.94214 14.7845 2.88529 14.6683 2.875 14.5437L1.84375 3H12.1562L11.125 14.5437Z"
          fill="#515150"
        />
      </svg>

      <svg
        width="16"
        height="16"
        viewBox="0 0 16 16"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          d="M14.999 0.949219H1.00119C0.112905 0.949219 -0.335282 1.89899 0.294061 2.45364L6 7.48321V12.6262C5.99999 12.7518 6.03046 12.8759 6.08936 12.9903C6.14826 13.1047 6.23423 13.2067 6.3415 13.2894L8.3415 14.8311C8.98019 15.3236 10 14.9325 10 14.1679V7.48321L15.7061 2.45364C16.3342 1.90012 15.8891 0.949219 14.999 0.949219ZM9 7.11817V14.1684L7 12.6262V7.11817L0.999999 1.8305H15L9 7.11817Z"
          fill="#515150"
        />
      </svg>
      <button
        v-if="showAddBtn"
        class="btn p-3 bg-[#14A892] text-white rounded-sm duration-300 hover:bg-[#0f7566]"
        @click="$emit('open-dialog')"
      >
        {{ btnLabel }}
      </button>
    </div>

    <table class="min-w-full">
      <thead>
        <tr>
          <th class="px-[24px] py-[12px] w-[20px] text-center">
            <input
              class="w-4 h-6 accent-[#14A892]"
              type="checkbox"
              v-model="allSelected"
            />
          </th>
          <th v-for="col in props.columns" :key="col.key" class="px-4 py-2">
            <div class="flex items-center gap-2">
              {{ col.label }}
              <sorter size="15" />
            </div>
          </th>
          <th class="px-4 py-2">Actions</th>
        </tr>
      </thead>

      <tbody>
        <tr
          v-for="(row, index) in paginatedData"
          :key="index"
          :class="[
            'cursor-pointer',
            selectedRows.includes(index) ? 'bg-teal-50' : 'hover:bg-gray-100',
          ]"
          @mouseenter="$emit('row-hover', row)"
          @mouseleave="$emit('row-leave')"
          @click="$emit('row-select', row)"
        >
          <td class="px-[24px] py-[12px]">
            <input
              class="w-4 h-6 accent-[#14A892]"
              type="checkbox"
              :value="index"
              v-model="selectedRows"
            />
          </td>
          <td
            v-for="col in props.columns"
            :key="col.key"
            class="px-4 py-1 hover:bg-gray-100"
          >
            <template v-if="col.formatter">
              <span class="flex items-center gap-2">
                <img
                  v-if="col.formatter(row).icon"
                  :src="col.formatter(row).icon"
                  alt="flag"
                  class="w-6 h-6 rounded"
                />

                <span v-else>{{ col.formatter(row).icon }}</span>

                <span>{{ col.formatter(row).text }}</span>
              </span>
            </template>

            <template v-else-if="typeof row[col.key] === 'boolean'">
              <toggleBtn v-model="row[col.key]" />
            </template>

            <template v-else-if="col.key === 'icons'">
              <svg
                width="20"
                height="20"
                viewBox="0 0 20 20"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
              >
                <circle
                  cx="9.99984"
                  cy="9.99967"
                  r="8.33333"
                  transform="rotate(-90 9.99984 9.99967)"
                  fill="#515150"
                />
              </svg>
            </template>

            <template v-else>
              {{ row[col.key] }}
            </template>
          </td>
          <td class="px-[24px] py-[12px] w-[50px] text-center relative">
            <button @click.stop="toggleMenu(index)" class="cursor-pointer">
              <svg
                viewBox="0 0 4 14"
                fill="none"
                xmlns="http://www.w3.org/2000/svg"
                class="w-4 h-4"
              >
                <path
                  d="M2.00019 5.59584C2.7765 5.59584 3.4037 6.22303 3.4037 6.99935C3.4037 7.77566 2.7765 8.40286 2.00019 8.40286C1.22387 8.40286 0.59668 7.77566 0.59668 6.99935C0.59668 6.22303 1.22387 5.59584 2.00019 5.59584ZM0.59668 1.73619C0.59668 2.51251 1.22387 3.1397 2.00019 3.1397C2.7765 3.1397 3.4037 2.51251 3.4037 1.73619C3.4037 0.959875 2.7765 0.332683 2.00019 0.332683C1.22387 0.332683 0.59668 0.959875 0.59668 1.73619ZM0.59668 12.2625C0.59668 13.0388 1.22387 13.666 2.00019 13.666C2.7765 13.666 3.4037 13.0388 3.4037 12.2625C3.4037 11.4862 2.7765 10.859 2.00019 10.859C1.22387 10.859 0.59668 11.4862 0.59668 12.2625Z"
                  fill="#515150"
                />
              </svg>
            </button>
            <div
              v-if="openMenuIndex === index"
              :ref="(el) => setMenuRef(el, index)"
              class="absolute right-0 mt-2 w-32 bg-white rounded shadow-lg z-10"
            >
              <button
                class="flex items-center gap-2 w-full text-left px-4 py-2 hover:bg-gray-100"
              >
                <svg
                  width="16"
                  height="16"
                  viewBox="0 0 16 16"
                  fill="none"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path
                    d="M15.9033 5.5938C14.397 2.65463 11.4147 0.666016 8 0.666016C4.58527 0.666016 1.60221 2.65602 0.0966517 5.59408C0.0331076 5.71978 0 5.85865 0 5.9995C0 6.14034 0.0331076 6.27922 0.0966517 6.40492C1.60304 9.34409 4.58527 11.3327 8 11.3327C11.4147 11.3327 14.3978 9.3427 15.9033 6.40464C15.9669 6.27894 16 6.14007 16 5.99922C16 5.85837 15.9669 5.7195 15.9033 5.5938ZM8 9.99937C7.20887 9.99937 6.43551 9.76477 5.77771 9.32524C5.11992 8.88572 4.60723 8.261 4.30447 7.5301C4.00172 6.79919 3.92251 5.99492 4.07685 5.219C4.23119 4.44307 4.61216 3.73034 5.17157 3.17093C5.73098 2.61152 6.44371 2.23055 7.21964 2.07621C7.99556 1.92187 8.79983 2.00108 9.53074 2.30383C10.2616 2.60659 10.8864 3.11928 11.3259 3.77707C11.7654 4.43487 12 5.20823 12 5.99936C12.0003 6.52472 11.897 7.04498 11.696 7.5304C11.4951 8.01582 11.2005 8.45688 10.829 8.82836C10.4575 9.19985 10.0165 9.49448 9.53104 9.69541C9.04562 9.89633 8.52536 9.99962 8 9.99937ZM8 3.33269C7.76198 3.33601 7.5255 3.37143 7.29694 3.43797C7.48534 3.69399 7.57574 4.00905 7.55177 4.32601C7.52779 4.64297 7.39101 4.94084 7.16625 5.16561C6.94148 5.39037 6.64361 5.52715 6.32665 5.55113C6.00969 5.5751 5.69463 5.4847 5.43861 5.2963C5.29282 5.83341 5.31914 6.40271 5.51385 6.92408C5.70857 7.44545 6.06188 7.89263 6.52406 8.20268C6.98623 8.51274 7.534 8.67005 8.09027 8.65247C8.64653 8.6349 9.18328 8.44333 9.62497 8.10473C10.0667 7.76612 10.391 7.29753 10.5525 6.76491C10.7139 6.23229 10.7042 5.66246 10.5248 5.13563C10.3454 4.60879 10.0053 4.15148 9.5524 3.82805C9.09948 3.50463 8.55654 3.33138 8 3.33269Z"
                    fill="#515150"
                  />
                </svg>
                View
              </button>
              <button
                class="flex items-center gap-2 w-full text-left px-4 py-2 hover:bg-gray-100"
              >
                <svg
                  width="16"
                  height="16"
                  viewBox="0 0 16 16"
                  fill="none"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path
                    d="M9.08573 2.91344L13.0862 6.91396L4.39929 15.6009L0.832505 15.9947C0.355018 16.0475 -0.0484094 15.6437 0.00471426 15.1662L0.401579 11.597L9.08573 2.91344ZM15.5606 2.31783L13.6822 0.439442C13.0962 -0.146481 12.146 -0.146481 11.56 0.439442L9.7929 2.20658L13.7934 6.2071L15.5606 4.43996C16.1465 3.85373 16.1465 2.90375 15.5606 2.31783Z"
                    fill="#515150"
                  />
                </svg>

                Edit
              </button>
              <button
                class="flex items-center gap-2 w-full text-left px-4 py-2 hover:bg-gray-100"
              >
                <svg
                  width="16"
                  height="16"
                  viewBox="0 0 16 16"
                  fill="none"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path
                    d="M14.5 0C15.3284 0 16 0.671562 16 1.5V10.5C16 11.3284 15.3284 12 14.5 12H5.5C4.67156 12 4 11.3284 4 10.5V1.5C4 0.671562 4.67156 0 5.5 0H14.5ZM5.5 13C4.1215 13 3 11.8785 3 10.5V4H1.5C0.671562 4 0 4.67156 0 5.5V14.5C0 15.3284 0.671562 16 1.5 16H10.5C11.3284 16 12 15.3284 12 14.5V13H5.5Z"
                    fill="#515150"
                  />
                </svg>
                Duplicate
              </button>
              <button
                class="flex items-center gap-2 w-full text-left px-4 py-2 hover:bg-gray-100"
              >
                <svg
                  width="14"
                  height="16"
                  viewBox="0 0 14 16"
                  fill="none"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path
                    d="M13.5 1.00001H9.75L9.45625 0.41563C9.39402 0.290697 9.29817 0.185606 9.17947 0.11218C9.06077 0.0387537 8.92394 -9.46239e-05 8.78437 5.47897e-06H5.2125C5.07324 -0.00052985 4.93665 0.0381736 4.81837 0.111682C4.7001 0.18519 4.60492 0.290529 4.54375 0.41563L4.25 1.00001H0.5C0.367392 1.00001 0.240215 1.05268 0.146447 1.14645C0.0526784 1.24022 0 1.3674 0 1.5L0 2.5C0 2.63261 0.0526784 2.75979 0.146447 2.85356C0.240215 2.94733 0.367392 3 0.5 3H13.5C13.6326 3 13.7598 2.94733 13.8535 2.85356C13.9473 2.75979 14 2.63261 14 2.5V1.5C14 1.3674 13.9473 1.24022 13.8535 1.14645C13.7598 1.05268 13.6326 1.00001 13.5 1.00001ZM1.6625 14.5938C1.68635 14.9746 1.85442 15.332 2.13252 15.5932C2.41061 15.8545 2.77781 16 3.15937 16H10.8406C11.2222 16 11.5894 15.8545 11.8675 15.5932C12.1456 15.332 12.3136 14.9746 12.3375 14.5938L13 4H1L1.6625 14.5938Z"
                    fill="#515150"
                  />
                </svg>

                Delete
              </button>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="flex items-center justify-end gap-3 mt-6 text-gray-500">
      <div>
        Showing
        {{ Math.min(currentPage * props.rowsPerPage, filteredData.length) }} of
        {{ filteredData.length }}
      </div>
      <div class="flex">
        <button
          @click="goToPage(1)"
          :disabled="currentPage === 1"
          class="px-2 border rounded-l-md border-gray-300"
        >
          «
        </button>
        <button
          @click="goToPage(currentPage - 1)"
          :disabled="currentPage === 1"
          class="px-2 border border-gray-300"
        >
          ‹
        </button>
        <button
          v-for="page in totalPages"
          :key="page"
          @click="goToPage(page)"
          :class="
            page === currentPage
              ? 'bg-teal-500 text-white'
              : 'hover:bg-gray-100'
          "
          class="px-3 py-1 border border-gray-300"
        >
          {{ page }}
        </button>
        <button
          @click="goToPage(currentPage + 1)"
          :disabled="currentPage === totalPages"
          class="px-2 py-1 border-y border-gray-300"
        >
          ›
        </button>
        <button
          @click="goToPage(totalPages)"
          :disabled="currentPage === totalPages"
          class="px-2 py-1 border border-gray-300 rounded-r-md"
        >
          »
        </button>
      </div>
    </div>
  </div>
</template>

<style></style>
