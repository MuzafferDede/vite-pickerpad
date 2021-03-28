<template>
  <div class="border rounded-lg p-2 space-y-2">
    <div class="grid grid-cols-3 items-center">
      <p>{{ datePicker.value }}</p>
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
    <div class="grid grid-cols-7 grid-rows-6 gap-2">
      <span v-for="day in init.blankDays" :key="day"></span>
      <button
        :class="{ 'bg-yellow-600 text-white': isToday(day) }"
        @click="setDate(day)"
        class="text-center rounded border p-2 focus:bg-blue-500 focus:text-white"
        v-for="day in init.days"
        :key="day"
        :ref="
          (el) => {
            if (el) refDays[day] = el;
          }
        "
        @keydown.up.down="handleUpDown(day, $event)"
        @keydown.left.right="handleLeftRight(day, $event)"
      >
        {{ day }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { computed, onBeforeUpdate, onMounted, reactive, ref } from "vue";

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

const current = reactive({
  month: today.getMonth(),
  year: today.getFullYear(),
});

const refDays = ref([]);
// make sure to reset the refs before each update
onBeforeUpdate(() => {
  refDays.value = [];
});

const datePicker = reactive({
  show: false,
  value: new Date(current.year, current.month, today.getDate()).toDateString(),
});

const init = reactive({ days: [], blankDays: [] });

const isToday = (day) => {
  return (
    today.toDateString() ==
    new Date(current.year, current.month, day).toDateString()
  );
};

const setLayout = computed(() => {
  init.blankDays = [];
  for (var i = 1; i <= new Date(current.year, current.month).getDay(); i++) {
    init.blankDays.push(i);
  }

  init.days = [];
  for (
    var i = 1;
    i <= new Date(current.year, current.month + 1, 0).getDate();
    i++
  ) {
    init.days.push(i);
  }
});

onMounted(setLayout);

const setDate = (day) => {
  datePicker.value = new Date(current.year, current.month, day).toDateString();
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
const handleUpDown = (day, $event) => {
  const operation = $event.key === "ArrowUp" ? -7 : +7;

  const date = new Date(current.year, current.month, day);

  date.setDate(date.getDate() + operation);
  current.year = date.getFullYear();
  current.month = date.getMonth();
  refDays.value[date.getDate()].focus();
};

const handleLeftRight = (day, $event) => {
  if ($event.key === "ArrowLeft") {
    if ($event.target.previousElementSibling && $event.target.previousElementSibling.tagName !== 'SPAN') {
      $event.target.previousElementSibling.focus();
      return;
    }
    setMonth(-1);
    refDays.value[new Date(current.year, current.month).getDate()].focus();
    return
  }
  if ($event.target.nextElementSibling) {
    $event.target.nextElementSibling.focus();
    return;
  }
  setMonth(+1);
  refDays.value[1].focus();
};
</script>
