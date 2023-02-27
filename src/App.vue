<script setup>

import jsonData from "./assets/data.json";
import {ref} from "vue";
import Linq from "linq";

const week = ref();
const dayOfTheWeek = ref();
const displayData = ref();

const weekSwitchingDisplay = ref(0);
const timeSwitchDisplay = ref(0);

const nowTime = ref();

screening();
getNowFormatTime();
setInterval(getNowFormatTime, 1000);

//更改周数
function setWeek() {
  if (week.value <= 0) {
    week.value = '';
  } else if (week.value > jsonData.totalWeeks) {
    week.value = jsonData.totalWeeks;
  }
  screening();
}

//更改星期
function setDayOfTheWeek() {
  if (dayOfTheWeek.value <= 0) {
    dayOfTheWeek.value = '';
  } else if (dayOfTheWeek.value > 7) {
    dayOfTheWeek.value = 7;
  }
  screening();
}

//获取课程时间
function getCourseDuration(numberOfSections) {
  let courseDuration = "";
  for (let i = 0; i < numberOfSections.length; i++) {
    courseDuration += jsonData.courseDuration[numberOfSections[i] - 1];
    courseDuration += "  "
  }
  return courseDuration;
}

//筛选
function screening() {
  displayData.value = Linq.from(jsonData.courses)
      .where(item => {
        if (week.value !== undefined && week.value !== '') {
          return Linq.from(item.week).contains(parseInt(week.value))
        } else {
          return true
        }
      })
      .where(item => {
        if (dayOfTheWeek.value !== undefined && dayOfTheWeek.value !== '') {
          return item.dayOfTheWeek == dayOfTheWeek.value
        } else {
          return true
        }
      })
      .toArray();
  // console.log(displayData.value);
}

//获取当前日期 2000-01-01
// function getNowFormatDate() {
//   let date = new Date();
//   let seperator1 = "-";
//   let year = date.getFullYear();
//   let month = date.getMonth() + 1;
//   let strDate = date.getDate();
//   if (month >= 1 && month <= 9) {
//     month = "0" + month;
//   }
//   if (strDate >= 0 && strDate <= 9) {
//     strDate = "0" + strDate;
//   }
//   let currentdate = year + seperator1 + month + seperator1 + strDate;
//   return currentdate;
// }

//获取当前时间 2000-01-01 00:00:00
function getNowFormatTime() {
  let date = new Date();
  let seperator1 = "-";
  let seperator2 = ":";
  let year = date.getFullYear();
  let month = date.getMonth() + 1;
  let strDate = date.getDate();
  let hours = date.getHours();
  let minutes = date.getMinutes();
  let seconds = date.getSeconds();
  if (month >= 1 && month <= 9) {
    month = "0" + month;
  }
  if (strDate >= 0 && strDate <= 9) {
    strDate = "0" + strDate;
  }
  if (hours >= 0 && hours <= 9) {
    hours = "0" + hours;
  }
  if (minutes >= 0 && minutes <= 9) {
    minutes = "0" + minutes;
  }
  if (seconds >= 0 && seconds <= 9) {
    seconds = "0" + seconds;
  }
  nowTime.value = year + seperator1 + month + seperator1 + strDate + " " + hours + seperator2 + minutes + seperator2 + seconds;
}

//获取今天星期几
function getDayOfTheWeek() {
  let date = new Date();
  let day = date.getDay();
  if (day === 0) {
    day = 7;
  }
  return day;
}

function getWeek(i) {
  switch (i) {
    case 1:
      return "一";
    case 2:
      return "二";
    case 3:
      return "三";
    case 4:
      return "四";
    case 5:
      return "五";
    case 6:
      return "六";
    case 7:
      return "日";
  }
}

//获取当前周数
function getTheCurrentWeekNumber() {

  let oneWeek = 7 * 24 * 60 * 60 * 1000; // 一周的毫秒数
  let startDate = new Date(jsonData.startData); // 起始日期
  let currentDate = new Date(); // 当前日期
  let timeDiff = currentDate.getTime() - startDate.getTime(); // 时间差（毫秒）
  let weeks = Math.floor(timeDiff / oneWeek) + 1; // 差的周数

  return weeks;
}

//本日课表
function setTodaySClassSchedule() {
  week.value = getTheCurrentWeekNumber();
  dayOfTheWeek.value = getDayOfTheWeek();
  screening();
}

//次日课表
function setTomorrowClassSchedule() {
  dayOfTheWeek.value = getDayOfTheWeek() >= 7 ? 1 : getDayOfTheWeek() + 1;
  week.value = getTheCurrentWeekNumber() + (dayOfTheWeek.value === 1 ? 1 : 0);
  screening();
}

//本周课表
function setThisWeeksClassSchedule() {
  week.value = getTheCurrentWeekNumber();
  dayOfTheWeek.value = '';
  screening();
}

//全部课表
function setAllClassSchedule() {
  week.value = '';
  dayOfTheWeek.value = '';
  screening();
}

//判断数组数据是否连续 若连续则输出1-3 若不连续则输出1-2，4-5
function getNumberOfSections(numberOfSections) {
  let result = "";
  let temp = [];
  for (let i = 0; i < numberOfSections.length; i++) {
    if (i === 0) {
      temp.push(numberOfSections[i]);
    } else {
      if (numberOfSections[i] - numberOfSections[i - 1] === 1) {
        temp.push(numberOfSections[i]);
      } else {
        if (temp.length === 1) {
          result += temp[0] + ", ";
        } else {
          result += temp[0] + "-" + temp[temp.length - 1] + ", ";
        }
        temp = [];
        temp.push(numberOfSections[i]);
      }
    }
  }
  if (temp.length === 1) {
    result += temp[0];
  } else {
    result += temp[0] + "-" + temp[temp.length - 1];
  }
  return result;
}

//切换周数显示
function switchingWeekDisplay() {
  weekSwitchingDisplay.value = weekSwitchingDisplay.value === 0 ? 1 : 0;
}

//切换节数和时间显示
function switchTimeDisplay() {
  timeSwitchDisplay.value = timeSwitchDisplay.value === 0 ? 1 : 0;
}


</script>


<template>

  <div class="title">
    <a href="#" data-title="Awesome Button"></a>
    <text>{{ jsonData.title }}</text>
    <label>{{ nowTime }} 星期{{ getWeek(getDayOfTheWeek()) }} 第{{ getTheCurrentWeekNumber() }}教学周
      {{ jsonData.semester }}</label>
  </div>

  <div class="input">
    <input v-model="week" @input="setWeek" placeholder="第几周">
    <input v-model="dayOfTheWeek" @input="setDayOfTheWeek" placeholder="星期几">
    <br>
    <button @click="setTodaySClassSchedule()">今日课表</button>
    <button @click="setTomorrowClassSchedule()">次日课表</button>
    <button @click="setThisWeeksClassSchedule()">本周课表</button>
    <button @click="setAllClassSchedule()">全部课表</button>
  </div>

  <table>
    <thead>
    <tr>
      <th>星期</th>
      <th @click="switchTimeDisplay()" v-if="timeSwitchDisplay===0">上课时间*</th>
      <th @click="switchTimeDisplay()" v-if="timeSwitchDisplay===1">节数*</th>
      <th>课程</th>
      <th>老师</th>
      <th>教室</th>
      <th @click="switchingWeekDisplay()">周数*</th>
    </tr>
    </thead>

    <tbody>
    <tr v-for="item in displayData" :key="item.id">
      <td>{{ item.dayOfTheWeek }}</td>
      <td v-if="timeSwitchDisplay===0">{{ getCourseDuration(item.numberOfSections) }}</td>
      <td v-if="timeSwitchDisplay===1">{{ item.numberOfSections }}</td>
      <td>{{ item.name }}</td>
      <td>{{ item.teacher }}</td>
      <td>{{ item.position }}</td>
      <td v-if="weekSwitchingDisplay===0">{{ getNumberOfSections(item.week) }}</td>
      <td v-if="weekSwitchingDisplay===1">{{ item.week }}</td>
    </tr>
    </tbody>
  </table>

</template>


<style>

label {
  display: block;
  text-align: center;
}

.title,
.input {
  text-align: center;
}

text {
  font-size: 50px;
  font-weight: bold;
}

table {
  /*border: 2px solid #66666650;*/
  border-radius: 3px;
  background-color: #fff;
  margin: auto;
}

th {
  background-color: #666666;
  color: rgba(255, 255, 255, 0.88);
  cursor: pointer;
  user-select: none;
}

td {
  background-color: #f9f9f9;
}

th,
td {
  min-width: 30px;
  padding: 10px 10px;
  text-align: center;
}

input {
  outline-style: none;
  border: 1px solid #ccc;
  border-radius: 3px;
  padding: 1% 1%;
  /*width: 200px;*/
  /*font-size: 14px;*/
  /*font-weight: 700;*/
  text-align: center;
  margin: 10px 10px;
}

input:focus {
  border-color: #66afe9;
  outline: 0;
  -webkit-box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075), 0 0 8px rgba(102, 175, 233, .6);
  box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075), 0 0 8px rgba(102, 175, 233, .6)
}

button {
  background-color: #e7e7e7;
  color: black;
  border: none;
  padding: 1% 1%;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  border-radius: 3px;

  margin: 10px 10px;

  -webkit-transition-duration: 0.2s; /* Safari */
  transition-duration: 0.2s;
}

button:hover {
  box-shadow: 0 5px 5px 0 rgba(0, 0, 0, 0.24), 0 17px 50px 0 rgba(0, 0, 0, 0.19);
}

</style>
