<script setup>
import {
  ref,
  reactive, onMounted, computed
} from "vue";
import VueDatePicker from '@vuepic/vue-datepicker';
import '@vuepic/vue-datepicker/dist/main.css'

const data = ref("")
const branchOptions = ref([])
const selectedBranchId = ref('429') //預設古亭店
const date = ref(new Date()) //預設今天日期
const emptySeats = ref(0)
const classCount = ref(1)
const searchOption = ref('searchOneWeek')

const formatDate = (date) => {
  if (date) {
    let day = date.getDate();
    if (day < 10) {
      day = `0${day}`
    }
    let month = date.getMonth() + 1;
    if (month < 10) {
      month = `0${month}`
    }
    const year = date.getFullYear();
    return `${year}-${month}-${day}`
  }

}





const params = reactive({
  branchId: computed(() => selectedBranchId.value),
  // tab: 'classes',
  page: 1,
  view: 'agenda',
  date: computed(() => formatDate(date.value))
})
const getAllBranch = async () => {
  const getAllBranchReq = new Request(`https://yogayogaapi-claras-projects-4138278a.vercel.app/getAllBranch`)
  await fetch(getAllBranchReq).then((response) => {
    if (!response.ok) {
      throw new Error('無法取得分店資訊')
    }
    return response.json()
  }).then((res) => {
    res = JSON.parse(res)
    branchOptions.value = res.data
    branchOptions.value.forEach((item) => {
      switch (item.branch_name) {

        case '好時光女生運動樂園 Café de GTFP':
          item.nickName = '南京復興'
          break;
        case '好時光女生運動樂園 The GTFP House':
          item.nickName = '古亭'
          break;
        case '好時光女生運動樂園 331 Boxing':
          item.nickName = '松江南京'
          break;
        case '好時光女生運動樂園 the GTFP Ohana':
          item.nickName = '大安'
          break;
        case '好時光女生運動樂園 The GTFP Lite':
          item.nickName = '江翠一館'
          break;
        case '好時光女生運動樂園 The GTFP Lucky7':
          item.nickName = '民權'
          break;
        case '好時光女生運動樂園 The GTFP Shaka':
          item.nickName = '西湖'
          break;
        case '好時光女生運動樂園 The GTFP Bingo':
          item.nickName = '中和'
          break;
        case ' 好時光女生運動樂園 The GTFP Hope':
          item.nickName = '新莊'
          break;
        case '好時光女生運動樂園The GTFP Plus':
          item.nickName = '江翠二館'
          break;
      }
    })

  })
}
const getList = async () => {
  const getFilterDatReq = new Request(`https://yogayogaapi-claras-projects-4138278a.vercel.app/getFilterData?branchId=${params.branchId}&page=${params.page}&view=${params.view}&date=${params.date}`, { method: 'GET' })
  await fetch(getFilterDatReq)
    .then((response) => {
      if (!response.ok) {
        throw new Error('Network response was not ok');
      }
      return response.json()
    })
    .then((res) => {
      res = JSON.parse(res)
      data.value = res.pageData
    })
    .catch((err) => {
      console.log('錯誤:', err);
    })
}


const getBranchName = (classFullName) => {
  const regex = /【(.*?)】/;
  const matched = classFullName.match(regex)
  if (matched && matched[1]) {
    return matched[0]
  } else {
    return classFullName
  }
}
const getClassName = (classFullName) => {
  const branchName = getBranchName(classFullName);
  return classFullName.replace(branchName, '')
}

getAllBranch()
getList()

const search = async () => {
  classCount.value = 0
  await getList()
  data.value.forEach((dailySchedule) => {
    if (dailySchedule.schedules.length > 0) {
      dailySchedule.schedules = dailySchedule.schedules?.filter((classInfo) => (classInfo.total_seats - classInfo.booked_seats) >= emptySeats.value)
      if (dailySchedule.schedules.length > 0) {
        classCount.value++
      }
    }
    if (searchOption.value == 'searchOneDay') {
      data.value = data.value.filter((item) => {
        return item.date == formatDate(date.value)
      })
    }
  })
}
onMounted(() => {
  const branchSelectorEl = document.getElementById("branchSelector");
  branchSelectorEl.addEventListener('change', ($event) => {
    selectedBranchId.value = $event.target.value
  })
})



</script>
<template>
  <div class="container text-center m-5">
    <div class="row mb-5">
      <!-- 分館 -->
      <div class="col-3">
        <div class="row justify-content-start align-items-center">
          <div class="col-3"><span>分館</span></div>
          <div class="col-9">
            <select class="form-select" aria-label="Default select example" id='branchSelector'>
              <option v-for="branch in branchOptions" :selected="branch.id == '429'" :key="branch.id"
                :value="branch.id">
                {{ branch.nickName }}</option>
            </select>
          </div>
        </div>
      </div>
      <div class="col-4">
        <div class="row justify-content-start align-items-center">
          <div class="col-3"><span>日期</span></div>
          <div class="col-9">
            <VueDatePicker v-model="date" :enable-time-picker="false"></VueDatePicker>
          </div>
        </div>
        <div class="row justify-content-start">
          <div class="col-3">

          </div>
          <div class="col-9 text-start">
            <div class="form-check form-check-inline">
              <input class="form-check-input" type="radio" name="inlineRadioOptions" id="searchOneWeek"
                value="searchOneWeek" v-model="searchOption">
              <label class="form-check-label" for="inlineRadio2">查詢一周內</label>
            </div>
            <div class="form-check form-check-inline">
              <input class="form-check-input" type="radio" name="inlineRadioOptions" id="searchOneDay"
                value="searchOneDay" v-model="searchOption">
              <label class="form-check-label" for="inlineRadio1">查詢當日</label>
            </div>

          </div>
        </div>
      </div>
      <div class="col-4">
        <div class="row justify-content-start align-items-center">
          <div class="col-4">剩餘人數</div>
          <div class="col-8">
            <input type="text" class="form-control" v-model='emptySeats'>
          </div>
        </div>
      </div>
      <div class="col-1 p-0">
        <!-- 搜尋 -->
        <button type="button" class=" btn btn-secondary " @click="search">搜尋</button>
      </div>
    </div>
    <div>
      <p v-if="classCount == 0">
        查無資料
      </p>
      <template v-else>
        <table class="table" v-for="(dailySchedule, index) in data" :key="index">
          <thead>

            <tr>
              <th scope="col">#</th>
              <th scope="col">分館</th>
              <th scope="col">課程日期</th>
              <th scope="col">課程時間</th>
              <th scope="col">課程名稱</th>
              <th scope="col">課程剩餘人數</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(classInfo, index) in dailySchedule.schedules" :key="index">
              <th scope="row">{{ index + 1 }}</th>
              <td>{{
                getBranchName(classInfo.name) }}</td>
              <td>{{ dailySchedule.date }}</td>
              <td>{{ classInfo.service_start_time }}~{{ classInfo.service_end_time }}</td>
              <td><a :href="`https://goodtime.17fit.com/myclass-confirm/${classInfo.id}`" target="_blank">{{
                getClassName(classInfo.name) }}</a></td>
              <td>{{ classInfo.total_seats - classInfo.booked_seats }}</td>
            </tr>

          </tbody>
        </table>
      </template>
    </div>
  </div>
</template>
<style>
.text {
  line-height: 100%;
}
</style>
