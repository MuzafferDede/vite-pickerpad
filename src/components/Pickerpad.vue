<template>
  <div class="relative p-4">
    <div class="w-full relative">
      <input
        aria-live="assertive"
        aria-atomic="true"
        v-model="currentDate"
        type="text"
        name="date"
        id="date"
        class="appearance-none w-full p-2 border border-gray-300 rounded-lg"
      />
      <button
        class="grid grid-cols-3 absolute right-0 inset-y-0 p-3"
        @click="toggle"
      >
        <span
          class="bg-gray-300 w-1 h-1 m-px"
          v-for="item in 9"
          :key="item"
        ></span>
      </button>
    </div>
    <div
      class="border rounded-lg p-4 space-y-2 absolute top-full"
      v-show="show"
      ref="pickerpad"
    >
      <div class="w-full space-y-4">
        <div class="flex justify-between items-center">
          <div class="space-x-2 flex justify-center">
            <select class="p-1" v-model="current.month">
              <option
                v-for="(month, value) in MONTHS"
                :key="value"
                :value="value"
              >
                {{ month }}
              </option>
            </select>
            <select class="p-1" v-model="current.year">
              <option
                v-for="yearOption in 20"
                :key="yearOption"
                :value="today.getFullYear() - 1 + yearOption"
              >
                {{ today.getFullYear() - 1 + yearOption }}
              </option>
            </select>
          </div>
          <div class="flex items-center justify-between space-x-4">
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
        <div class="grid grid-cols-7 gap-2">
          <span class="text-center p-2" v-for="day in DAYS" :key="day">{{
            day
          }}</span>
        </div>
        <div
          class="grid grid-cols-7 grid-rows-6 gap-2 focus:outline-none"
          v-swipe="`yes`"
          tabindex="0"
          :aria-label="`Current selected date ${currentDate}`"
          @focus="active = true"
          @blur="active = false"
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
    </div>
  </div>
</template>

<script setup>
import { computed, reactive, ref, defineProps, onMounted } from "vue";

// const props = defineProps({
//   show: Boolean,
// });

// const variables

const MONTHS = [
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
// reactive
const active = ref(false);
const show = ref(false);
// computed
const currentDate = computed(() => {
  return new Date(current.year, current.month, current.day).toDateString();
});

const attributes = computed(() => (day) => {
  return !day
    ? { class: "bg-gray-50 border rounded" }
    : {
        class: {
          "bg-yellow-600 text-white": isToday(day),
          "bg-blue-500 text-white ring-2": focusedDay(day),
          "text-center rounded border p-2 hover:bg-blue-500 hover:text-white cursor-pointer": day,
        },
      };
});

const days = computed(() => {
  return Array.from(Array(42).keys()).map((day) => {
    day = day - new Date(current.year, current.month).getDay() + 1;
    return {
      day:
        day > 0 && day <= new Date(current.year, current.month + 1, 0).getDate()
          ? day
          : undefined,
    };
  });
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
  return current.day === day && active.value;
};

const setMonth = (operation) => {
  const currentMonth = current.month;

  current.month = circular(current.month + operation, 11, 0);

  if (currentMonth + operation < 0) {
    current.year--;
  }

  if (currentMonth + operation > 11) {
    current.year++;
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

const makeDate = (date) => {
  const newDate = {
    year: date.year || current.year,
    month: date.month || current.month,
    day: date.day || current.day,
  };
};

const focusTrap = (element, trigger) => {
  const focusableEls = element.querySelectorAll(
    '[tabindex="0"], button:not([disabled]), select:not([disabled])'
  );
  const firstFocusableEl = focusableEls[0];
  const lastFocusableEl = focusableEls[focusableEls.length - 1];
  const KEYCODE_TAB = 9;

  setTimeout(() => {
    firstFocusableEl.focus();
  }, 10);

  element.addEventListener("keydown", function (e) {
    if (e.key === "Escape") {
      show.value = false;
      trigger.focus();
    }

    var isTabPressed = e.key === "Tab" || e.keyCode === KEYCODE_TAB;

    if (!isTabPressed) {
      return;
    }

    if (e.shiftKey) {
      /* shift + tab */ if (document.activeElement === firstFocusableEl) {
        lastFocusableEl.focus();
        e.preventDefault();
      }
    } /* tab */ else {
      if (document.activeElement === lastFocusableEl) {
        firstFocusableEl.focus();
        e.preventDefault();
      }
    }
  });
};

const pickerpad = ref(null);

const toggle = (trigger) => {
  show.value = !show.value;

  show.value && focusTrap(pickerpad.value, trigger.target);
};

const unify = (e) => {
  return e.changedTouches ? e.changedTouches[0] : e;
};

const checkSwipe = (position) => {
  
} 

const swipe = {
  beforeMount(el, binding, vnode, prevVnode) {
    let position = {
      current: 0,
      before: 0
    };

    el.addEventListener("touchstart", (event) => {
      position.current = unify(event).clientX;
    });
    
    el.addEventListener("touchend", (event) => {
      position.before =  unify(event).clientX;
      checkSwipe(position)
    });
  },
};
</script>
