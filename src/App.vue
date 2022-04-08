<script setup>
import { ref, computed } from 'vue'
import axios from "axios"
const history = ref([])
const prefix = ref("8801886")
const simNumber = ref("")
const state = ref({
  searchingType: ''
})

const option = ref({
  continueSearch: false,
  incrementBy: 1,
  searchLimited: false,
  searchLimit: 100,
})

const fullNumber = computed(() => prefix.value.concat(simNumber.value))


function startSearch() {
  if (fullNumber.value.length == 13) {
    const serverApiUrl = "https://da-api.robi.com.bd/da-nll/free-msisdn/get-msisdn-status"
    const postData = {
      "msisdn": fullNumber.value
    }

    state.value.searchingType = "static"
    axios.post(serverApiUrl, postData)
      .then(res => {
        state.value.searchingType = ''
        // console.log(res.data)
        if (res.data.status == "SUCCESSFUL") {
          history.value.push({
            simNumber: fullNumber.value,
            available: res.data.data.available
          })
          if (option.value.continueSearch && history.value.length < option.value.searchLimit) {
            setTimeout(() => {
              simNumber.value = parseInt(parseInt(simNumber.value) + parseInt(option.incrementBy ?? 1))
              if (option.value.searchLimit) {
                if (history.value.length < option.value.searchLimit) {
                  startSearch()
                }
              } else {
                startSearch()
              }
            }, 2000)

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

  state.value.searchingType = 'random'
  axios.post(serverApiUrl, postData)
    .then(res => {
      state.value.searchingType = ''
      // console.log(res.data)
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
              <h3 class="card-title border-bottom mb-3 pb-2 text-center">Find Available Numbers</h3>
              <form @submit.prevent="startSearch()" class="mb-5">
                <div class="row mb-2">
                  <div class="col-5">
                    <input
                      class="mb-2 form-control"
                      v-model="prefix"
                      type="text"
                      placeholder="01886"
                    />
                    <button
                      type="button"
                      @click="getRandomAvailableNumber"
                      class="btn w-100 btn-primary"
                    >
                      Random
                      Numbers
                    </button>
                  </div>
                  <div class="col-7">
                    <input
                      class="mb-2 form-control"
                      v-model="simNumber"
                      type="text"
                      placeholder="Enter last 6 digit of your choice"
                    />
                    <button
                      type="submit"
                      class="btn w-100 btn-primary"
                      :class="[simNumber.length == 6 ? 'btn-success' : 'disabled']"
                    >Search Number</button>
                  </div>
                </div>
              </form>

              <p>Options</p>
              <div class="form-check form-switch d-flex justify-content-between align-items-center">
                <div>
                  <input
                    class="form-check-input"
                    v-model="option.continueSearch"
                    type="checkbox"
                    id="continueSearch"
                  />
                  <label class="form-check-label" for="continueSearch">Continue search by :</label>
                </div>
                <input
                  type="text"
                  v-model="option.incrementBy"
                  class="form-control w-50"
                  :disabled="!option.continueSearch"
                />
              </div>
              <div class="form-check form-switch d-flex justify-content-between align-items-center">
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

              <hr />

              <p class="alert alert-info text-center" v-if="state.searchingType == 'static'">
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
                Checking Number: {{ fullNumber }}
              </p>
              <p class="alert alert-info text-center" v-if="state.searchingType == 'random'">
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
                Searching Random Avaialble Numbers
              </p>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <div class="card">
            <div class="card-body">
              <div class="d-flex justify-content-between align-items-center border-bottom mb-3">
                <h3 class="card-title">History</h3>
                <button
                  class="btn p-2 btn-close"
                  data-tooltip="Clear history"
                  @click="history = []"
                ></button>
              </div>
              <ul class="list-group" v-if="history.length > 0">
                <li
                  v-for="record in history"
                  :key="record.simNumber"
                  class="list-group-item d-flex justify-content-between align-items-center"
                  :class="[record.available ? 'list-group-item-info' : 'list-group-item-secondary']"
                >
                  <span>
                    {{
                      record.simNumber
                    }}
                    <span
                      :class="[record.available ? 'text-primary' : 'text-danger']"
                    >({{ record.available ? 'Available' : 'Not Available' }})</span>
                  </span>
                  <a
                    v-if="record.available"
                    target="_blank"
                    class="btn btn-primary btn-sm"
                    href="https://onlinesim.robi.com.bd/robi/newconnection/1/2227/registration"
                  >Buy Now</a>
                </li>
              </ul>
              <h5 v-else class="text-center">no number searched yet</h5>
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