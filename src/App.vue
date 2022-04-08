<script setup>
import { ref, computed } from 'vue'
import axios from "axios"

const history = ref([])
const prefix = ref("8801886")
const simNumber = ref("")

const state = ref({
  searching: false
})

const option = ref({
  continueSearch: false,
  searchingType: '',
  incrementBy: 1,
  searchLimit: 100,
  onlyAvailableNumber: true,
})

const fullNumber = computed(() => prefix.value.concat(simNumber.value))

function playSound(type = '') {
  switch (type) {
    case 'gotNumber':
      new Audio('http://soundbible.com/mp3/Air Plane Ding-SoundBible.com-496729130.mp3').play()
      break
    case 'completeSearching':
      new Audio('https://assets.mixkit.co/sfx/download/mixkit-unlock-game-notification-253.wav').play()
      break

    default:
      new Audio('http://soundbible.com/mp3/Air Plane Ding-SoundBible.com-496729130.mp3').play()
      break
  }

}

function addToHistory(obj) {

  if (obj.available) {
    playSound('gotNumber')
  }

  if (option.value.onlyAvailableNumber) {
    if (res.data.data.available) {
      history.value.unshift(obj)
    }
  } else {
    history.value.unshift(obj)
  }
}

function initAutoSearch() {
  if (option.value.continueSearch) {
    if (option.value.searchLimit > 0) {
      if (history.value.length < option.value.searchLimit) {
        simNumber.value = (parseInt(simNumber.value) + parseInt(option.value.incrementBy)).toString().padStart(simNumber.value.length, "0")
        startSearch()
      } else {
        option.value.searchingType = ""
        state.value.searching = false
        playSound('completeSearching')
      }

    } else {
      simNumber.value = (parseInt(simNumber.value) + parseInt(option.value.incrementBy)).toString().padStart(simNumber.value.length, "0")
      startSearch()
    }

  }
}

function startSearch() {
  if (fullNumber.value.length == 13) {
    const serverApiUrl = "https://da-api.robi.com.bd/da-nll/free-msisdn/get-msisdn-status"
    const postData = {
      "msisdn": fullNumber.value
    }

    option.value.searchingType = "static"
    state.value.searching = true

    axios.post(serverApiUrl, postData)
      .then(res => {
        option.value.searchingType = ''
        if (res.data.status == "SUCCESSFUL") {
          addToHistory({
            simNumber: fullNumber.value,
            available: res.data.data.available
          })

          if (state.value.searching) {
            initAutoSearch()
          }
        } else {
          console.log('server error')
        }
      })
      .catch(e => {
        alert('Something went wrong please the console')
        console.error(e)
      })


  } else {
    alert('Please enter valid number')
  }
}

function getRandomAvailableNumber() {
  const serverApiUrl = "https://da-api.robi.com.bd/da-nll/free-msisdn/get-msisdn-list"
  const postData = {
    "brand": "ROBI",
    "simCategory": "PREPAID"
  }

  option.value.searchingType = 'random'
  state.value.searching = true
  axios.post(serverApiUrl, postData)
    .then(res => {
      option.value.searchingType = ''
      state.value.searching = false

      res.data.freeMsisdnList.forEach(number => {
        history.value.push({
          simNumber: number,
          available: true
        })
      })

    })
    .catch(e => {
      alert('Something went wrong please the console')
      console.error(e)
    })
}

</script>

<template>
  <header class="bg-dark text-white p-3 mb-5 text-center">
    <h1>Robi Number Finder Tool</h1>
  </header>

  <div class="main">
    <div class="container">
      <div class="row">
        <div class="col-md-6 mb-5">
          <div class="card">
            <div class="card-body">
              <h3 class="card-title border-bottom pb-2 text-center">Find Available Numbers</h3>
              <form @submit.prevent="startSearch()">
                <div class="row mb-2">
                  <div class="col-5 pe-1">
                    <input
                      class="mb-2 form-control text-end"
                      v-model="prefix"
                      :disabled="state.searching"
                      type="text"
                      placeholder="01886"
                    />
                    <button
                      v-if="!state.searching"
                      type="button"
                      @click="getRandomAvailableNumber"
                      class="btn w-100 btn-secondary"
                    >
                      Random
                      Numbers
                    </button>
                  </div>
                  <div class="col-7 ps-1">
                    <input
                      class="mb-2 form-control"
                      v-model="simNumber"
                      type="text"
                      placeholder="Enter last 6 digit of your choice"
                      :disabled="state.searching"
                    />
                    <button
                      v-if="!state.searching"
                      type="submit"
                      class="btn w-100 btn-primary"
                      :disabled="simNumber.length != 6"
                    >Search</button>
                  </div>
                </div>
              </form>

              <div v-if="!state.searching">
                <p>Options</p>
                <div
                  class="form-check form-switch d-flex justify-content-between align-items-center"
                >
                  <div>
                    <input
                      class="form-check-input"
                      v-model="option.onlyAvailableNumber"
                      type="checkbox"
                      id="onlyAvailableNumber"
                    />
                    <label class="form-check-label" for="onlyAvailableNumber">Only Available Number</label>
                  </div>
                </div>

                <div
                  class="form-check form-switch d-flex justify-content-between align-items-center"
                >
                  <div>
                    <input
                      class="form-check-input"
                      v-model="option.continueSearch"
                      type="checkbox"
                      id="continueSearch"
                    />
                    <label class="form-check-label" for="continueSearch">Auto Search :</label>
                  </div>
                  <input
                    type="text"
                    v-model="option.incrementBy"
                    class="form-control w-50"
                    :disabled="!option.continueSearch"
                  />
                </div>
                <div
                  class="form-check form-switch d-flex justify-content-between align-items-center"
                >
                  <div>
                    <input
                      class="form-check-input"
                      v-model="option.searchLimited"
                      type="checkbox"
                      id="searchLimited"
                      :disabled="!option.continueSearch"
                    />
                    <label class="form-check-label" for="searchLimited">Search limit :</label>
                  </div>
                  <input
                    type="text"
                    v-model="option.searchLimit"
                    class="form-control w-50"
                    :disabled="!option.searchLimited"
                  />
                </div>
              </div>

              <div v-if="state.searching">
                <p class="alert alert-info text-center" v-if="option.searchingType == 'static'">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    xmlns:xlink="http://www.w3.org/1999/xlink"
                    style="margin:auto;background:transparent;display:block;"
                    width="50px"
                    height="50px"
                    viewBox="0 0 100 100"
                    preserveAspectRatio="xMidYMid"
                  >
                    <rect x="17.5" y="30" width="15" height="40" fill="#e15b64">
                      <animate
                        attributeName="y"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="18;30;30"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                        begin="-0.2s"
                      />
                      <animate
                        attributeName="height"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="64;40;40"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                        begin="-0.2s"
                      />
                    </rect>
                    <rect x="42.5" y="30" width="15" height="40" fill="#f8b26a">
                      <animate
                        attributeName="y"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="20.999999999999996;30;30"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                        begin="-0.1s"
                      />
                      <animate
                        attributeName="height"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="58.00000000000001;40;40"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                        begin="-0.1s"
                      />
                    </rect>
                    <rect x="67.5" y="30" width="15" height="40" fill="#abbd81">
                      <animate
                        attributeName="y"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="20.999999999999996;30;30"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                      />
                      <animate
                        attributeName="height"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="58.00000000000001;40;40"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                      />
                    </rect>
                  </svg>
                  Checking: {{ fullNumber }}
                </p>
                <p class="alert alert-info text-center" v-if="option.searchingType == 'random'">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    xmlns:xlink="http://www.w3.org/1999/xlink"
                    style="margin:auto;background:transparent;display:block;"
                    width="50px"
                    height="50px"
                    viewBox="0 0 100 100"
                    preserveAspectRatio="xMidYMid"
                  >
                    <rect x="17.5" y="30" width="15" height="40" fill="#e15b64">
                      <animate
                        attributeName="y"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="18;30;30"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                        begin="-0.2s"
                      />
                      <animate
                        attributeName="height"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="64;40;40"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                        begin="-0.2s"
                      />
                    </rect>
                    <rect x="42.5" y="30" width="15" height="40" fill="#f8b26a">
                      <animate
                        attributeName="y"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="20.999999999999996;30;30"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                        begin="-0.1s"
                      />
                      <animate
                        attributeName="height"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="58.00000000000001;40;40"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                        begin="-0.1s"
                      />
                    </rect>
                    <rect x="67.5" y="30" width="15" height="40" fill="#abbd81">
                      <animate
                        attributeName="y"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="20.999999999999996;30;30"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                      />
                      <animate
                        attributeName="height"
                        repeatCount="indefinite"
                        dur="1s"
                        calcMode="spline"
                        keyTimes="0;0.5;1"
                        values="58.00000000000001;40;40"
                        keySplines="0 0.5 0.5 1;0 0.5 0.5 1"
                      />
                    </rect>
                  </svg>
                  Searching Random Available Numbers
                </p>
              </div>

              <div class="col-12">
                <button
                  v-if="state.searching"
                  @click="state.searching = false"
                  type="button"
                  class="btn w-100 btn-danger"
                >Stop</button>
              </div>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <div class="card">
            <div class="card-body">
              <div class="d-flex justify-content-between align-items-center border-bottom mb-3">
                <h3 class="card-title">
                  History
                  <span v-if="history.length">({{ history.length }})</span>
                </h3>
                <button
                  class="btn p-2 btn-close"
                  data-tooltip="Clear history"
                  @click="history = []"
                ></button>
              </div>
              <ul class="list-group" v-if="history.length > 0">
                <li
                  v-for="(record, index) in history"
                  :key="index"
                  class="list-group-item d-flex justify-content-between align-items-center"
                  :class="[record.available ? 'list-group-item-info' : 'list-group-item-secondary']"
                >
                  <span>{{ record.simNumber }}</span>
                  <span :class="[record.available ? 'text-primary' : 'text-danger']">
                    ({{
                      record.available ? 'Available'
                        : 'Not Available'
                    }})
                  </span>

                  <a
                    v-if="record.available"
                    target="_blank"
                    class="btn btn-primary btn-sm"
                    href="https://onlinesim.robi.com.bd/robi/newconnection/1/2227/registration"
                  >Buy Now</a>
                </li>
              </ul>
              <h5 v-else class="text-center">no record found</h5>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <footer class="text-center bg-dark p-2 text-white mt-5">
    <p class="mb-2">@ Copyright 2022 idemonbd, All rights reserved.</p>
    <p class="mb-2 text-white-50">
      [ Made with
      <span style="color: red;">&hearts;</span> by
      <a class="text-decoration-none text-warning" href="https://m.me/idemonbd">Emon Khan</a> ]
    </p>
  </footer>
</template>


<style>
ul {
  height: 80vh;
  overflow: scroll;
}

body {
  min-width: 400px;
}
</style>