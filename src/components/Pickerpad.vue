<template>
  <div class="border rounded-lg p-2 space-y-2" v-if="show">
    <div class="grid grid-cols-3 items-center">
      <p>{{ currentDate }}</p>
    </div>

    <div class="rounded-lg bg-wite py-10 px-6 flex items-center space-x-4">
      <button class="p-3" @click="setMonth(-1)">
        <svg
          class="h-4 w-4"
          viewBox="0 0 16 16"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            d="M15 8L1 8M1 8L8 15M1 8L8 0.999999"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
      </button>
      <div class="w-full space-y-8">
        <p class="space-x-2 flex justify-center">
          <strong>{{ MONTH_NAMES[current.month] }}</strong>
          <span>{{ current.year }}</span>
        </p>
        <div class="grid grid-cols-7 gap-2">
          <span class="text-center p-2" v-for="day in DAYS" :key="day">{{
            day
          }}</span>
        </div>
        <div
          class="grid grid-cols-7 grid-rows-6 gap-2 focus:outline-none"
          tabindex="0"
          ref="grid"
          @keydown.up.down="handleUpDown"
          @keydown.left.right="handleLeftRight"
        >
          <span
            v-bind="attributes(cell.day)"
            @click="cell.day && setDate(cell.day)"
            v-for="cell in days"
            :key="cell.day"
          >
            {{ cell.day }}
          </span>
        </div>
      </div>
      <button class="p-3" @click="setMonth(+1)">
        <svg
          class="h-4 w-4"
          viewBox="0 0 16 16"
          fill="none"
          xmlns="http://www.w3.org/2000/svg"
        >
          <path
            d="M1 8H15M15 8L8 1M15 8L8 15"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { computed, reactive, defineProps } from "vue";

const props = defineProps({
  show: Boolean,
});

// const variables

const MONTH_NAMES = [
  "January",
  "February",
  "March",
  "April",
  "May",
  "June",
  "July",
  "August",
  "September",
  "October",
  "November",
  "December",
];

const DAYS = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];

const today = new Date();

// reactive
const current = reactive({
  month: today.getMonth(),
  year: today.getFullYear(),
  day: today.getDate(),
});

// computed
const currentDate = computed(() => {
  return new Date(current.year, current.month, current.day).toDateString();
});

const attributes = computed(() => (day) => {
  return !day
    ? undefined
    : {
        class: {
          "bg-yellow-600 text-white": isToday(day),
          "bg-blue-500 text-white ring-2": focusedDay(day),
          "text-center rounded border p-2 hover:bg-blue-500 hover:text-white cursor-pointer": day,
        },
      };
});

const days = computed(() => {
  const list = [];
  for (
    var i = -(new Date(current.year, current.month).getDay() - 1);
    i <= new Date(current.year, current.month + 1, 0).getDate();
    i++
  ) {
    list.push({ day: i > 0 ? i : undefined });
  }

  return list;
});

// methods
const isToday = (day) => {
  return (
    today.toDateString() ==
    new Date(current.year, current.month, day).toDateString()
  );
};

const circular = (value, max, min = 1) => {
  max = max - min + 1;
  return ((value - min + max) % max) + min;
};

const setDate = (day) => {
  current.day = day;
};

const focusedDay = (day) => {
  return current.day === day;
};

const setMonth = (operation) => {
 current.month =  circular(current.month + operation, 11, 0)
};

const handleUpDown = ($event) => {
  const operation = $event.key === "ArrowUp" ? -7 : +7;

  const date = new Date(current.year, current.month, current.day);

  date.setDate(date.getDate() + operation);

  current.year = date.getFullYear();
  current.month = date.getMonth();
  current.day = date.getDate();
};

const handleLeftRight = ($event) => {
  $event.key === "ArrowLeft" ? current.day-- : current.day++;

  if (current.day > new Date(current.year, current.month + 1, 0).getDate()) {
    current.day = 1;
    setMonth(+1);
  }

  if (current.day < 1) {
    current.day = new Date(current.year, current.month, 0).getDate();
    setMonth(-1);
  }
};

const makeDate = (date) => {
  const newDate = {
    year: date.year || current.year,
    month: date.month || current.month,
    day: date.day || current.day,
  };
};

</script>
