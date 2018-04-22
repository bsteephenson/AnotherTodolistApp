<template lang="pug">
.main-container
    div
        button.toggleHiddenButton(@click="showHidden = !showHidden") Show hidden 
            i(class = "fa", :class="{'fa-toggle-on': showHidden, 'fa-toggle-off': !showHidden}")
    div.day(v-for="(date, index) in sortedDates", :class="{today: (index == 0)}")
        div.header
            .date {{ date }}
            button(@click="taskAdd(date)") +
        div.boxes
            div.box(v-for="(task, taskIndex) in allTasks[date]", :class="{done: task.done, highlighted: task.highlighted && !(task.done)}" v-if="!(task.hidden) || showHidden")
                .text
                    textarea(contenteditable="true", v-model="task.name")
                .buttons
                    i.button(@click="task.done = !(task.done)", class="fa fa-check", :class="{selected: task.done}")
                    i.button(@click="task.highlighted = !(task.highlighted)", class="fa fa-star", :class="{selected: task.highlighted}")
                    i.button(@click="task.repeat = !(task.repeat)", class="fa fa-repeat", :class="{selected: task.repeat}")
                    i.button(@click="task.hidden = !(task.hidden)", class="fa fa-trash", :class="{selected: task.hidden}")
    hr
    div
        p Purpose: keep a long running todo list, keep track of what you worked on or completed over time. 
        p Every not "completed" task that you add today will carry over to tomorrow.
        p Only green tasks are "completed".
        p If a task is recurring (the recurring symbol is on), then it will carry over to the next day even if it is completed today.
        p Starring a task (making it blue) is just a visual aid. This can be used for keeping track of things worked on but not completed.

</template>

<script>

import moment from 'moment'

var STORAGE_KEY = "allTasks"
var DATE_FORMAT = "MMMM Do, YYYY"

export default {
    name: 'app',
    data: function () {
        return {
            allTasks: {
                "December 3rd, 2003": [
                    {name: "hello", done: false, highlighted: false}
                    ]
                },
            showHidden: false
            }
    },
    created: function() {
        console.log("Created")
        window.app = this
        window.moment = moment
        this.load(moment().format(DATE_FORMAT))
        self = this
        window.addEventListener('storage', function() {
            self.load(moment().format(DATE_FORMAT))
        })
    },
    computed: {
        sortedDates: function() {
            return this.sortDates(this.allTasks)
        }
    },
    watch: {
        allTasks: {
            handler: function(all) {
                this.save()
            },
            deep: true
        }
    },
    methods: {
        sortDates: function (allTasks) {
            var dates = Object.keys(allTasks).map(function(dateString) {
                var it = moment(dateString, DATE_FORMAT).valueOf()
                return it
            })
            dates.sort()
            dates.reverse()
            dates = dates.map(function(date) {
                var m = moment(date)
                var f = m.format(DATE_FORMAT)
                return f
            })
            var filteredDates = []
            var first = true
            for (var i in dates) {
                if (first) {
                    first = false
                    filteredDates.push(dates[i])
                }
                else {
                    var date = dates[i]
                    if (allTasks[date].length > 0) {
                        filteredDates.push(date)
                    }
                }
            }
            return filteredDates
        },

        taskAdd: function(date) {
            this.allTasks[date].push({name: "", done: false, highlighted: false, repeat: false, hidden: false})
        },
        toggleTaskState: function(task) {
            if (task.done) {
                task.done = false
                task.highlighted = false
            }
            else if (task.highlighted) {
                task.done = true
            }
            else {
                task.highlighted = true
            }
        },
        toggleTaskRepeat : function(task) {
            if (task.repeat) {
                task.repeat = false
            }
            else {
                task.repeat = true
            }
            console.log(task)
        },
        save: function () {
            var stuff = JSON.stringify(this.allTasks)
            localStorage.setItem(STORAGE_KEY, stuff)
        },
        load: function(dateString) {
            var date = moment(dateString, DATE_FORMAT)
            var stuff = localStorage.getItem(STORAGE_KEY) || "null"
            var allTasks = JSON.parse(stuff)
            if (!allTasks) {
                // Just add today's thing
                allTasks = {}
                allTasks[dateString] = []
            }
            else {
                if (!(allTasks[dateString])) {
                    var notDone = []
                    var lastday = this.sortDates(allTasks)[0]
                    var lastDateTasks = allTasks[lastday]
                    for (var i in lastDateTasks) {
                        if ((!(lastDateTasks[i].done) || (lastDateTasks[i].repeat)) && !(lastDateTasks[i].hidden)) {
                            var task = {}
                            task.name = lastDateTasks[i].name
                            task.done = false
                            task.highlighted = false
                            task.repeat = lastDateTasks[i].repeat || false
                            notDone.push(task)
                        }
                    }
                    allTasks[dateString] = notDone
                }
            }

            // Legacy: old tasks did not have a highlighted or a repeat property
            for (var key in allTasks) {
                for (var i in allTasks[key]) {
                    if (!('highlighted' in allTasks[key][i])) {
                        allTasks[key][i].highlighted = false
                    }
                    allTasks[key][i].repeat = allTasks[key][i].repeat || false
                    allTasks[key][i].hidden = allTasks[key][i].hidden || false
                }
            }

            this.allTasks = allTasks
        },
        clear: function() {
            localStorage.removeItem(STORAGE_KEY)
        }
    }
}


</script>




<style lang = "stylus">
    
body
    display: flex
    flex-direction: row
    justify-content: center

.main-container
    width: 75%
    font-family: "Gill Sans", "Gill Sans MT", sans-serif !important
    font-weight: 200 !important
.header
    font-size: 42px
    margin-top: 10px
    margin-bottom: 10px
    display: flex
    flex-direction: row
    width: 100%
    button
        margin-left: 10px
        background-color: alpha(white, 0)
        border: none
        text-decoration: none
        font-size: 20px
        cursor: pointer
        &:focus
            outline: 0px solid transparent
        

.box
    background-color: alpha(grey, .15)
    width: 100px
    height: 100px
    padding: 10px
    margin: 1px
    display: flex
    flex-direction: column
    .text
        height: 90%
        textarea
            width: 100%
            height: 100%
            background-color: alpha(white, 0)
            border: none
            resize: none
            font-size: 16px
            font-weight: inherit
            &:focus
                outline: 0px solid transparent
    
    .buttons
        height: 10%
        font-size: 14px
        display: flex
        flex-direction: row
        flex-wrap: wrap
        justify-content: space-between
        
        .button
            display: none
            cursor: pointer
            color: alpha(black, .25)
            &:hover
                color: alpha(black, 1)
            &.selected
                color: alpha(black, 1)

    &:hover .buttons .button
            display: block      

.box.done
    background-color: alpha(#77E25C, .75)

.box.highlighted
    background-color: alpha(#00BFFF, .5)


.boxes
    display: flex
    flex-wrap: wrap

.day
    opacity: .5
.day.today
    opacity: 1



.toggleHiddenButton
    padding-left: 0px
    opacity: .1
    &:hover
        opacity: 1
    margin-left: 0px
    background-color: alpha(white, 0)
    border: none
    text-decoration: none
    font-size: 12px
    cursor: pointer
    &:focus
        outline: 0px solid transparent

</style>
