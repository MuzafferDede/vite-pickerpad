<template>
  <div class="border rounded-lg p-2 space-y-2" v-if="show">
    <div class="grid grid-cols-3 items-center">
      <p>{{ currentDate }}</p>
      <p class="space-x-2 flex justify-center">
        <strong>{{ MONTH_NAMES[current.month] }}</strong>
        <span>{{ current.year }}</span>
      </p>
      <div class="flex space-x-2 justify-end">
        <button class="px-3 border rounded" @click="setMonth(-1)">&lt;</button>
        <button class="px-3 border rounded" @click="setMonth(+1)">&gt;</button>
      </div>
    </div>
    <div class="grid grid-cols-7 gap-2 border-t">
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
        @click="setDate(cell.day)"
        v-for="cell in days"
        :key="cell.day"
      >
        {{ cell.day }}
      </span>
    </div>
  </div>
</template>

<script setup>
import { computed, reactive, defineProps } from "vue";

const props = defineProps({
  show: Boolean,
})

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
          "bg-blue-500 text-white": focusedDay(day),
          "text-center rounded border p-2 hover:bg-blue-500 hover:text-white cursor-pointer": day,
        },
      };
});

const days = computed(() => {
  const list = [];
  for (var i = 1; i <= new Date(current.year, current.month).getDay(); i++) {
    list.push({ day: null });
  }

  for (
    var i = 1;
    i <= new Date(current.year, current.month + 1, 0).getDate();
    i++
  ) {
    list.push({ day: i });
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

const setDate = (day) => {
  current.day = day;
};

const focusedDay = (day) => {
  return current.day === day && !isToday(day);
};

const setMonth = (operation) => {
  current.month = current.month + operation;

  if (current.month === 0) {
    current.year--;
    current.month = 11;
    return;
  }

  if (current.month > 11) {
    current.year++;
    current.month = 0;
    return;
  }
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
</script>
