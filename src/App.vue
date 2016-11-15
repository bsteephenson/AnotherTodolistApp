<template lang="jade">
.main-container
    div.day(v-for="(date, index) in sortedDates")
        div.header
            .date {{ date }}
            button(@click="taskAdd(date)") +
        div.boxes
            div.box(v-for="(task, taskIndex) in allTasks[date]", :class="{done: task.done, highlighted: task.highlighted && !(task.done) && (index == 0)}")
                .text
                    textarea(contenteditable="true", v-model="task.name")
                .buttons
                    .button(@click="task.done = !(task.done)") Done
                    .button(v-if= "index == 0" @click="task.highlighted = !(task.highlighted)") Select
                    .button(@click="allTasks[date].splice(taskIndex, 1)") Delete
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
                }
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
            this.allTasks[date].push({name: "", done: false})
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
                        if (!(lastDateTasks[i].done)) {
                            var task = {}
                            task.name = lastDateTasks[i].name
                            task.done = false
                            notDone.push(task)
                        }
                    }
                    allTasks[dateString] = notDone
                }
            }

            // Legacy: old tasks did not have a highlighted property
            for (var key in allTasks) {
                for (var i in allTasks[key]) {
                    if (!('highlighted' in allTasks[key][i])) {
                        allTasks[key][i].highlighted = false
                    }
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
        font-size: 12px
        display: flex
        flex-direction: row
        justify-content: space-between
        
        .button
            display: none
            cursor: pointer
            &:hover
                text-decoration: underline

    &:hover .buttons .button
            display: block      

.box.done
    background-color: alpha(#77E25C, .75)

.box.highlighted
    background-color: alpha(#00BFFF, .5)


.boxes
    display: flex
    flex-wrap: wrap


</style>
