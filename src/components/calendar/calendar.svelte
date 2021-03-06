<script lang="ts">
  // Port and modification from vue-sweet-calendar
  // https://github.com/maryayi/vue-sweet-calendar/blob/master/src/components/Calendar.vue

  // svelte
  import { onMount, onDestroy } from "svelte";
  import { createEventDispatcher } from "svelte";

  // Local
  import DateTime from "./date-time.js";
  import math from "../../utils/math/math";

  // vendors
  import dayjs from "dayjs";
  import type { Dayjs } from "dayjs";

  // Utils & modules
  import Logger from "../../utils/log/log";
  import calcTrackerScore from "../../modules/scoring/score-tracker";
  // import { TrackerStore } from "../../store/tracker-store";
  import { UserStore } from "../../store/user-store";
  import { Lang } from "../../store/lang";

  // Components
  import NIcon from "../../components/icon/icon.svelte";
  import Text from "../../components/text/text.svelte";
  import NPositivityBar from "../../components/positivity-bar/positivity-bar.svelte";

  import NextPrevCal from "../../components/next-prev-cal/next-prev-cal.svelte";

  const console = new Logger("📅 calendar/calendar");
  const dispatch = createEventDispatcher();

  // Props
  export let initialDate = dayjs();
  export let size = "md";
  export let className = "";

  let firstDayOfWeek;

  // Updating to be react...
  $: firstDayOfWeek = $UserStore.meta.firstDayOfWeek || 1; // 1: Sunday, 2: Monday, etc.

  // export let eventCategories = [];
  export let events = [];
  export let offDays = [[1, 7]];
  export let showHeader = true;
  export let tracker = null;

  // Data
  export let state = {
    date: initialDate.format ? initialDate : dayjs(initialDate),
    today: new Date(),
    weekdays: null,
    percentage: null,
    totals: {
      positive: 0,
      negative: 0,
      neutral: 0,
    },
  };

  let mounted = false;

  onMount(() => {
    mounted = true;
  });

  onDestroy(() => {
    mounted = false;
  });

  let days: Array<any>;
  let day = null;

  let startWeekDayOfMonth = state.date.startOf("month").toDate().getDay() + 1;
  let numberOfDays = state.date.daysInMonth();
  let selectedMonth = state.date.month();
  let selectedMonthName = state.date.format("MMMM");
  let selectedYear = state.date.format("YYYY");
  let monthStartDate = dayjs(state.date).startOf("month");
  let refreshing = false;

  let positiveCount = 0;
  let negativeCount = 0;
  let neutralCount = 0;

  // If the initial date is set, convert to dayjs date
  $: if (initialDate) {
    state.date = dayjs(initialDate);
    state.weekdays = methods.generateWeekdayNames(firstDayOfWeek);
    let positiveCount = 0;
    let negativeCount = 0;
    let neutralCount = 0;
  }

  function init() {
    lastDate = state.date;

    state.totals.neutral = 0;
    state.totals.positive = 0;
    state.totals.negative = 0;

    startWeekDayOfMonth = state.date.startOf("month").toDate().getDay() + 1;

    numberOfDays = state.date.daysInMonth();
    selectedMonth = state.date.month();
    selectedMonthName = state.date.format("MMMM");
    selectedYear = state.date.format("YYYY");
    monthStartDate = dayjs(state.date).startOf("month");

    // Create array of empty days previous month calendar bleed over
    let emptyDays = Array((startWeekDayOfMonth - firstDayOfWeek + 7) % 7).fill(null);
    // Create array of days for this month
    let nonEmptyDays = Array(numberOfDays)
      .fill(0)
      .map((item, index) => dayjs(monthStartDate).add(index, "day"));
    // Merge the arrays
    days = emptyDays.concat(nonEmptyDays);
    state.percentage = nonEmptyDays.length / (emptyDays.length + nonEmptyDays.length);
  }

  // If date change - do the magic.
  let lastDate = null;
  $: if (state.date && state.date != lastDate) {
    init();
  }

  // Methods
  const methods = {
    prevMonth() {
      state.date = state.date.subtract(1, "month");
    },
    nextMonth() {
      state.date = state.date.add(1, "month");
    },
    generateWeekdayNames(firstDayOfWeek = 1) {
      let weekdays = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
      for (let i = 2; i <= firstDayOfWeek; i++) {
        let first = weekdays.shift();
        weekdays.push(first);
      }
      return weekdays;
    },
    // Refresh the view
    refresh() {
      refreshing = true;
      setTimeout(() => {
        refreshing = false;
      }, 1);
    },
    // Go to Today
    goToday(withClick: boolean = true) {
      state.date = dayjs();
      if (withClick) {
        dispatch("dayClick", state.date);
      }
    },
    /**
     * Generate the Style for the Provided Day
     */
    getDayStyle(day) {
      let total = 0;
      let score = undefined;
      // Filter values, get only this days
      let values = events
        .filter((row) => {
          return day.toDate().toDateString() === new Date(row.end).toDateString();
        })
        // Map the tracker - and return the sum or avg values
        .map((row) => {
          if (!row.trackers) {
            row.getMeta();
          }
          if (tracker.math == "sum") {
            return math.sum(row.getTrackerValues(tracker.tag));
          } else {
            return math.average(row.getTrackerValues(tracker.tag));
          }
        });
      // If we have values
      if (values.length) {
        // If it's a sum
        if (tracker.math == "sum") {
          total = math.sum(values);
        } else {
          total = math.average(values);
        }
      }
      // If we have a total - figure the score
      if (total) {
        score = calcTrackerScore(total, tracker);
      }
      // Lets extract the score for this tracker
      if (values.length) {
        // Did we pass in a tracker?
        if (score == 0) {
          state.totals.neutral = state.totals.neutral + 1;
        } else if (score > 0) {
          state.totals.positive = state.totals.positive + 1;
        } else if (score < 0) {
          state.totals.negative = state.totals.negative + 1;
        }
        return methods.getDayBorder(score);
      } else {
        return ``;
      }
    },
    getDayBorder(score) {
      if (score) {
        if (score > 0) {
          return `font-weight:bold; border:solid 2px var(--color-green); color:#FFF; background-color:var(--color-green)`;
        } else {
          return `font-weight:bold; border:solid 2px var(--color-red); color:#FFF; background-color:var(--color-red)`;
        }
      } else {
        return `font-weight:bold; border:solid 2px var(--color-primary-bright); color:#FFF; background-color:var(--color-primary-bright);`;
      }
    },
    getDayClass(day) {
      try {
        const dayFormat = day.format("YYYY-MM-DD");
        const stateDateFormat = state.date.format("YYYY-MM-DD");
        const todayFormat = dayjs(state.today).format("YYYY-MM-DD");
        const activeToday = events.find((row) => {
          return dayFormat === stateDateFormat;
        });
        let classes = [
          "day",
          `day-${day.format("D")}`,
          `weekday-${day.toDate().getDay()}`,
          activeToday ? "selected" : "not-selected",
          offDays.includes(day.toDate().getDay()) ? "off-day" : null,
          dayFormat === stateDateFormat ? "active" : "inactive",
          dayFormat === todayFormat ? "today" : null,
        ];
        return classes.join(" ");
      } catch (e) {
        console.error(e);
        return "error-generating-classes";
      }
    },
  };
</script>

<style lang="scss">
  // Partially based on https://www.npmjs.com/package/vue-sweet-calendar
  @import "../../scss/utils/_utils";

  .sweet-calendar.sm {
    --cal-font-size: 0.5rem;
    --cal-day-size: 16px;
  }
  .sweet-calendar.md {
    --cal-font-size: 0.8rem;
    --cal-day-size: 20px;
  }
  .sweet-calendar.lg {
    --cal-font-size: 0.8rem;
    --cal-day-size: 24px;
  }

  .sweet-calendar {
    $off-day-background-color: #e5e5e5;
    $selected-color: #994242;
    $selection-hover-color: rgba($selected-color, 0.5);
    $today-color: #008290;
    $day-color: #232323;
    $week-day-name-color: rgba($day-color, 0.7);

    .active {
      position: relative;
      &:before {
        content: "✔";
        font-size: 10px;
        color: var(--color-green);
        top: -4px;
        right: -4px;
        position: absolute;
      }
    }
    .today {
      position: relative;
      &:after {
        content: "";
        height: 4px;
        width: 4px;
        border-radius: 2px;
        background-color: var(--color-primary-bright);
        top: -3px;
        position: absolute;
      }
    }
    .sweet-container {
      //   display: grid;
      //   grid-template-rows: 40px 1fr;
      .header {
        align-items: center;
        display: grid;
        grid-column-gap: 4px;
        grid-template-columns: 1fr 24px 50px 24px;
        margin: 6px 0;
        margin-left: 16px;
        .month {
          flex-grow: 1;
        }
        .left-arrow {
          justify-self: center;
          span {
            cursor: pointer;
          }
        }
        .right-arrow {
          justify-self: center;
          span {
            cursor: pointer;
          }
        }
      }
      .body {
        align-items: center;
        display: grid;
        grid-row-gap: 4px;
        grid-template-columns: repeat(7, 1fr);
        grid-template-rows: repeat(7, 1fr);
        justify-items: center;
        .day-name {
          color: var(--color-inverse);
          font-weight: bold;
          font-size: 0.6rem;
          height: 18px;
        }
        .day-container {
          display: flex;
          justify-content: center;
          align-items: center;
          height: 28px;
          position: relative;
          width: 100%;
          .day {
            align-content: center;
            border-radius: 32%;
            box-sizing: content-box;
            color: var(--color-inverse-2);
            display: flex;
            height: var(--cal-day-size);
            justify-content: center;
            min-width: var(--cal-day-size);
            &.selected {
              border: 3px solid var(--color-solid-2);
            }
            &.off-day {
              background-color: #e5e5e5;
              font-weight: bold;
            }
            span {
              font-size: var(--cal-font-size);
              margin: 0;
              padding: 0;
              align-self: center;
            }
          }
        }
      }
    }
    .calendar {
      background-color: inherit;
      .body .day-container {
        .before,
        .after {
          height: 28px;
        }
      }
    }
    .date-picker {
      background-color: inherit;
      .body .day-container .day {
        &:hover {
          background-color: rgba(153, 66, 66, 0.5);
          border: 2px solid #994242;
          cursor: pointer;
        }
        &.selected {
          border: 2px solid #994242;
          color: #994242;
        }
      }
    }

    .day.filled {
    }
    .day.unfilled {
      background-color: transparent;
    }
  }
</style>

{#if state.date && mounted}
  <div class="sweet-calendar {className} {size}">
    <div class="sweet-container calendar">
      {#if showHeader}
        <div class="header pr-3">
          <div class="month filler">
            <Text size="sm" bold>{selectedMonthName} {selectedYear}</Text>
          </div>
          <NextPrevCal
            on:previous={methods.prevMonth}
            on:next={methods.nextMonth}
            on:calendar={() => {
              methods.goToday(false);
            }} />
        </div>
      {/if}

      <div class="body">
        {#each state.weekdays || [] as day, index}
          <div class="day-name" aria-label={day}>{day[0]}</div>
        {/each}
        {#each days || [] as day}
          <div class="day-container clickable">
            {#if day}
              <div
                data-date={day.format('YYYY-MM-DD')}
                on:click={(event) => {
                  dispatch('dayClick', day);
                }}
                class={methods.getDayClass(day)}
                style={methods.getDayStyle(day)}>
                <span>{day.format('D')}</span>
              </div>
            {/if}
          </div>
        {/each}
      </div>
    </div>
  </div>
  <NPositivityBar positive={state.totals.positive} neutral={state.totals.neutral} negative={state.totals.negative} />
{:else}Loading{/if}
