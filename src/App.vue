<template>
    <div id="app">
        <div v-if="messageError" class="notification is-danger">{{ messageError }}</div>
        <div v-else-if="message" class="notification is-success">{{ message }}</div>
        <div class="calendar-wrapper">
            <div class="calendar-controls">
                <div class="box" v-if="isAddNewEvent">
                    <div class="field">
                        <label class="label">Title</label>
                        <div class="control">
                            <input v-model="itemTitle" class="input" type="text" required />
                        </div>
                    </div>

                    <div class="field">
                        <label class="label">Event date</label>
                        <div class="control">
                            <input v-model="itemEventDate" class="input" type="date" required />
                        </div>
                    </div>

                    <div class="field">
                        <label class="label">Time start</label>
                        <div class="control">
                            <input v-model="itemTimeStart" class="input" type="time" required />
                        </div>
                    </div>

                    <button class="button is-info" @click="clickAddItem">
                        Add Item
                    </button>
                </div>
            </div>
            <div class="calendar-parent">
                <calendar-view
                        :items="items"
                        :show-date="showDate"
                        :show-times="true"
                        :time-format-options="{ hour: 'numeric', minute: '2-digit' }"
                        :class="themeClasses"
                        :enable-date-selection="false"
                        :enable-drag-drop="true"
                        @click-date="onClickDay"
                        @drop-on-date="onDrop"
                        @click-item="onClickItem"
                >
                    <calendar-view-header
                            slot="header"
                            slot-scope="{ headerProps }"
                            :header-props="headerProps"
                    />
                </calendar-view>
            </div>
        </div>
    </div>
</template>

<script>
    require("vue-simple-calendar/static/css/default.css");
    require("vue-simple-calendar/static/css/holidays-us.css");

    import {
        CalendarView,
        CalendarViewHeader,
        CalendarMathMixin,
    } from "vue-simple-calendar" // published version
    //} from "../../vue-simple-calendar/src/components/bundle.js" // local repo

    const COUNT_WORKOUTS_TOTAL = 9;
    const COUNT_WORKOUTS_WEEK = 3;

    export default {
        name: "App",
        components: {
            CalendarView,
            CalendarViewHeader,
        },
        mixins: [CalendarMathMixin],
        data: function () {
            return {
                showDate: new Date(),
                useDefaultTheme: true,
                itemTitle: "",
                itemEventDate: "",
                itemTimeStart: "",
                message: "",
                messageError: "",
                isAddNewEvent: false,
                addedWorkouts: 1, // todo replace with 0
                items: [
                    {
                        id: "e1",
                        startDate: this.thisMonth(15, 18, 30),
                        title: "title",
                    }
                ],
            }
        },
        computed: {
            themeClasses() {
                return {
                    "theme-default": this.useDefaultTheme,
                }
            },
        },
        methods: {
            thisMonth(d, h, m) {
                const t = new Date();
                return new Date(t.getFullYear(), t.getMonth(), d, h || 0, m || 0);
            },
            onClickDay(d) {
                this.isAddNewEvent = true;
                this.itemEventDate = this.isoYearMonthDay(d);
            },
            clickAddItem() {
                if (COUNT_WORKOUTS_TOTAL >= this.addedWorkouts) {
                    let newEventDate = new Date(this.itemEventDate);
                    // count added event on the week
                    let countAddedWorkoutsForWeek = 0;
                    let weekDateStart = 10; // todo calculate start of the week
                    let weekDateEnd = weekDateStart + 6;

                    while (weekDateStart <= weekDateEnd) {
                        this.items.forEach((item) => {
                            let itemDate = new Date(item.startDate);
                            if (newEventDate.getFullYear() === itemDate.getFullYear()
                                && newEventDate.getMonth() === itemDate.getMonth()
                                && weekDateStart === itemDate.getDate()) {
                                countAddedWorkoutsForWeek++;
                            }
                        });
                        weekDateStart++;
                    }

                    if (COUNT_WORKOUTS_WEEK > countAddedWorkoutsForWeek) {
                        let hasEvent = false;
                        let newEventDate = new Date(this.itemEventDate);
                        this.items.forEach((item) => {
                            let itemDate = new Date(item.startDate);

                            if (newEventDate.getFullYear() === itemDate.getFullYear()
                                && newEventDate.getMonth() === itemDate.getMonth()
                                && newEventDate.getDate() === itemDate.getDate()) {
                                hasEvent = true;
                            }
                        });

                        if (!hasEvent) {
                            let date = new Date(this.itemEventDate + " " + this.itemTimeStart);

                            this.items.push({
                                startDate: date,
                                title: this.itemTitle,
                                id: "e" + Math.random().toString(36).substr(2, 10),
                            });

                            this.message = "You added a calendar item!";
                            this.messageError = "";
                            this.addedWorkouts++;
                        } else {
                            this.messageError = "The event has already added here. Please choose another date";
                        }
                    }
                    else {
                        this.messageError = "The week available workouts is exceeded";
                    }
                }
                else {
                    this.messageError = "The total available workouts is exceeded";
                }
            },
            onDrop(item, date) {
                let hasEvent = false;
                this.items.forEach((item) => {
                    let itemDate = new Date(item.startDate);

                    if (date.getFullYear() === itemDate.getFullYear()
                        && date.getMonth() === itemDate.getMonth()
                        && date.getDate() === itemDate.getDate()) {

                        hasEvent = true;
                    }
                });

                if (!hasEvent) {
                    // Determine the delta between the old start date and the date chosen,
                    // and apply that delta to both the start and end date to move the item.
                    const eLength = this.dayDiff(item.startDate, date);
                    item.originalItem.startDate = this.addDays(item.startDate, eLength);
                    item.originalItem.endDate = this.addDays(item.endDate, eLength);

                    // todo it doesn't necessary or come up with something different
                    this.message = `You dropped ${item.id} on ${date.toLocaleDateString()}`;
                    this.messageError = "";
                }
                else {
                    this.messageError = "The same date cannot contain more than one event";
                }
            },
            onClickItem(e) {
                this.items =  this.items.filter(function(item) {
                    return item.id !== e.id;
                });

                this.message = `You removed: ${e.id}`;
                this.messageError = "";
            },
        },
    }
</script>

<style scoped>
    .calendar-wrapper {
        display: flex;
        flex-direction: row;
        font-family: Calibri, sans-serif;
        width: 95vw;
        min-width: 30rem;
        max-width: 100rem;
        min-height: 40rem;
        margin-left: auto;
        margin-right: auto;
    }
    .calendar-controls {
        margin-right: 1rem;
        min-width: 14rem;
        max-width: 14rem;
    }

    .calendar-parent {
        display: flex;
        flex-direction: column;
        flex-grow: 1;
        overflow-x: hidden;
        overflow-y: hidden;
        max-height: 80vh;
        background-color: white;
    }
</style>