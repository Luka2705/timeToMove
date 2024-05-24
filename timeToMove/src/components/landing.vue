<template>
    <section class="py-lg-7 py-5 bg-light-subtle" style="height: 100%;">
        <div class="container">
            <div class="row">
                <div class="col-lg-12">
                    <div class="mb-3">
                        <h1 class="mb-0 h3">Hey, User! Welcome Back.</h1>
                    </div>
                    <!--email-->
                    <div class="mb-5">
                        <label class="form-label" for="email">Upload your csv file</label>
                        <input type="file" id="email" class="form-control" @change="onFileChanged">
                    </div>
                    <div class="row mb-3" v-if="loadedFile">
                        <div class="col-2">
                            <button class="btn btn-primary" @click="togglePlay">{{ isPlaying ? 'Pause' : 'Play'
                                }}</button>
                        </div>
                        <div class="col-10">
                            <VueSlider class="form-range mt-1" v-model="sliderValue" :min="minValue" :max="maxValue" />
                        </div>
                    </div>
                    <h4 v-if="loadedFile">{{ new Date(sliderValue) }}</h4>

                    <div id="road">
                        <div class="station" :style="{ left: (fertigungsPunkte.indexOf(fp) * 10) + '%' }"
                            v-for="fp in fertigungsPunkte">{{ fp }}</div>
                        <div class="car mb-2" v-for="(item, id) in Object.entries(carList)" :key="id"
                            :style="{ left: item[1].leftPadding, top: item[1].topPadding }">
                            {{ item[0] }}
                        </div>
                    </div>

                    <!--
                        <div class="row mt-4 mb-5 g-4">
                        <div class="col-lg-2">
                            <div class="card border-0 shadow-sm">
                                <div class="card-body">
                                    <span>FP 1</span>
                                    <h3 class="mb-0 mt-4">$0.61</h3>
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-2">
                            <div class="card border-0 shadow-sm">
                                <div class="card-body">
                                    <span>FP 1</span>
                                    <h3 class="mb-0 mt-4">$0.61</h3>
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-2">
                            <div class="card border-0 shadow-sm">
                                <div class="card-body">
                                    <span>FP 1</span>
                                    <h3 class="mb-0 mt-4">$0.61</h3>
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-2">
                            <div class="card border-0 shadow-sm">
                                <div class="card-body">
                                    <span>FP 1</span>
                                    <h3 class="mb-0 mt-4">$0.61</h3>
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-2">
                            <div class="card border-0 shadow-sm">
                                <div class="card-body">
                                    <span>FP 1</span>
                                    <h3 class="mb-0 mt-4">$0.61</h3>
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-2">
                            <div class="card border-0 shadow-sm">
                                <div class="card-body">
                                    <span>Past 7 days</span>
                                    <h3 class="mb-0 mt-4">$0</h3>
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-2">
                            <div class="card border-0 shadow-sm">
                                <div class="card-body">
                                    <span>Total Earnings</span>
                                    <h3 class="mb-0 mt-4">$0.61</h3>
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-2">
                            <div class="card border-0 shadow-sm">
                                <div class="card-body">
                                    <span>FP 1</span>
                                    <h3 class="mb-0 mt-4">$0.61</h3>
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-2">
                            <div class="card border-0 shadow-sm">
                                <div class="card-body">
                                    <span>Past 7 days</span>
                                    <h3 class="mb-0 mt-4">$0</h3>
                                </div>
                            </div>
                        </div>
                        <div class="col-lg-2">
                            <div class="card border-0 shadow-sm">
                                <div class="card-body">
                                    <span>Total Earnings</span>
                                    <h3 class="mb-0 mt-4">$0.61</h3>
                                </div>
                            </div>
                        </div>
                    </div>
                    -->

                </div>
            </div>
        </div>
    </section>
</template>

<script setup>
import { ref, onMounted } from "vue";
import Papa from 'papaparse';
import VueSlider from 'vue-3-slider-component'

let isPlaying = ref(false)
let sortedTimeStamps = ref([])
let intervalId = ref(null)
let timeToMoveObj = ref(null);
let carData = ref({});
let minValue = ref(0)
let maxValue = ref(1);
let sliderValue = ref(0)
let carList = ref({});
let loadedFile = ref(false)
let fertigungsPunkte = ref(['5499', '7924', '1800', '1802', '1805', '1808', '1809', '1828', '1893', '2940'])


const togglePlay = (() => {
    isPlaying.value = !isPlaying.value;
    if (isPlaying.value) {
        startSlider();
    } else {
        pauseSlider();
    }
})

const startSlider = (() => {
    intervalId.value = setInterval(() => {
        if (sliderValue.value < maxValue.value) {
            updateCar();
            sliderValue.value += 10000;
        } else {
            pauseSlider(); // Stop when it reaches the max value
        }
    }, 100); // Adjust the interval duration as needed
})

const pauseSlider = (() => {
    clearInterval(intervalId.value);
    intervalId.value = null;
})

function updateCar() {
    Object.entries(timeToMoveObj.value).map((cars) => {
        if (parseInt(cars[0]) === sliderValue.value) {
            for (let item in cars[1]) {
                let shiftValue = carData.value[cars[1][item]]
                const count = {};

                for (const key in carList.value) {
                    if (carList.value.hasOwnProperty(key)) {
                        const leftPadding = carList.value[key].leftPadding;
                        if (count[leftPadding]) {
                            count[leftPadding]++;
                        } else {
                            count[leftPadding] = 1;
                        }
                    }
                }

                carList.value[cars[1][item]].leftPadding = fertigungsPunkte.value.indexOf(shiftValue[sliderValue.value]) * 10 + '%'
                carList.value[cars[1][item]].topPadding = count[fertigungsPunkte.value.indexOf(shiftValue[sliderValue.value]) * 10 + '%'] != undefined ? count[fertigungsPunkte.value.indexOf(shiftValue[sliderValue.value]) * 10 + '%'] * 30 + 'px' : 0 + 'px'

                console.log(carList.value);
            }
        }
    })
}

const onFileChanged = ((event) => {
    const files = event.target.files
    Papa.parse(files[0], {
        header: true,
        skipEmptyLines: true,
        complete: function (results) {
            processData(results.data);
        },
    })
})

function processData(data) {
    // Initialize a Set to hold unique 'FertigungspunktName' values
    const uniqueFertigungspunktNames = new Set();
    // Initialize a Set to hold unique 'FertigungspunktName' values
    let uniqueTimeStamps = new Set();

    data.forEach((entry) => {
        const {
            ProdNr,
            KarNr,
            Fertigungspunkt,
            FertigungspunktName,
            Erfassungszeit,
        } = entry;

        // Create a unique key for each car
        const carKey = `${KarNr}`;

        //Add FertigunspunktName to set
        uniqueFertigungspunktNames.add(FertigungspunktName);

        if (fertigungsPunkte.value.includes(Fertigungspunkt)) {
            //add timestamp value to set
            const timeInMillis = Date.parse(Erfassungszeit);
            uniqueTimeStamps.add(timeInMillis);

            // Initialize car entry if it doesn't exist
            if (!carData.value[carKey]) {
                carData.value[carKey] = {};
            }

            if (!carList.value[carKey]) {
                carList.value[carKey] = { leftPadding: 0 + '%', topPadding: '' };
            }

            // Initialize FertigungspunktName entry if it doesn't exist
            if (!carData.value[carKey][timeInMillis]) {
                carData.value[carKey][timeInMillis] = Fertigungspunkt;
            }
        }

    });

    console.log(carList.value)

    //Sort uniqueTimeStamp Set
    sortedTimeStamps.value = Array.from(uniqueTimeStamps).sort(
        (a, b) => a - b
    );

    timeToMoveObj.value = formTimeToMove(carData.value);
    minValue.value = sortedTimeStamps.value[0]
    sliderValue.value = sortedTimeStamps.value[0];
    maxValue.value = sortedTimeStamps.value[sortedTimeStamps.value.length - 1]
    loadedFile.value = true;
}

function formTimeToMove(inputObj) {
    // Initialize the new object
    let timeToMove = {};

    // Iterate over each ID in the input object
    for (let id in inputObj) {
        // Get the events for the current ID
        let events = inputObj[id];

        // Iterate over each timestamp in the events
        for (let timestamp in events) {
            // If the timestamp is not yet a key in timeToMove, create an empty array
            if (!timeToMove[timestamp]) {
                timeToMove[timestamp] = [];
            }

            // Add the current ID to the array for this timestamp
            timeToMove[timestamp].push(id);
        }
    }

    // Return the new object
    return timeToMove;
}
</script>

<style>
#road {
    width: 100%;
    height: 40vh;
    position: relative;
    background-color: #e0e0e0;
    margin-top: 50px;
}

.station {
    width: 50px;
    height: 8vh;
    background-color: red;
    margin-right: 2rem;
    position: absolute;
    top: 25vh;
}

.car {
    width: 50px;
    height: 25px;
    top: 29vh;
    background-color: blue;
    position: absolute;
    transition: left 0.5s, top 0.5s;
}
</style>