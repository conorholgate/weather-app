<template>
  <div class="flex flex-col min-h-full">
    <div class="bg-indigo-600">
        <Disclosure as="nav" class="bg-indigo-600 border-b border-indigo-300 border-opacity-25 lg:border-none" v-slot="{ open }">
            <div class="px-2 mx-auto max-w-7xl sm:px-4 lg:px-8">
                <div class="relative flex items-center justify-between h-16 lg:border-b lg:border-indigo-400 lg:border-opacity-25">
                    <div class="flex items-center lg:px-0">
                        <div class="hidden text-white lg:block">
                            WeatherApp
                            <!-- <img class="block w-8 h-8" src="https://tailwindui.com/img/logos/workflow-mark-indigo-300.svg" alt="Workflow" /> -->
                        </div>
                    </div>
                    <div class="flex justify-center flex-1 px-2 lg:px-0 lg:ml-6 lg:justify-end">
                        <div class="w-full max-w-lg lg:max-w-lg">
                            <label for="search" class="sr-only">Search</label>
                            <div class="relative text-gray-400 focus-within:text-gray-600">
                                <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
                                    <SearchIcon class="w-5 h-5" aria-hidden="true" />
                                </div>
                                <input id="search" class="block w-full py-2 pl-10 pr-3 text-sm leading-5 text-gray-900 placeholder-gray-500 bg-white focus:outline-none lg:text-md" 
                                    :class="searching && searchInput != '' ? 'border border-transparent rounded-t-md' : 'border border-transparent rounded-md'"
                                    placeholder="Enter post code, city or town" 
                                    type="search" 
                                    name="search" 
                                    v-model="searchInput"
                                    @keyup.enter="getSearchedLocations"/>

                                <template v-if="searching && searchInput">
                                    <div class="absolute w-full h-32 overflow-y-scroll text-gray-900 bg-white border border-transparent rounded-b-md border-t-gray-200">
                                        <div v-if="searchingLocations.length" class="">
                                            <div v-for="(location,key) in searchingLocations" :key="key">
                                                <p class="px-4 py-1.5 cursor-pointer hover:bg-gray-200" @click="getSearchedLocation(location,key)">{{ location.formatted }}</p>
                                            </div>
                                        </div>
                                        <div v-if="noLocationsFound">
                                            <p class="px-4 py-1.5">No location found, please try again</p>
                                        </div>
                                        <div v-if="!noLocationsFound && !searchingLocations.length">
                                            <p class="px-4 py-1.5">Fetching Locations...</p>
                                        </div>
                                    </div>
                                </template>

                            </div>
                        </div>
                        <button class="px-2 ml-2 leading-5 text-white border-2 rounded-md cursor-pointer hover:bg-white hover:text-indigo-500"
                            @click="getSearchedLocations"
                        >Search</button>
                        <div class="items-center hidden p-2 ml-4 text-white border-2 border-white rounded-full cursor-pointer md:flex hover:bg-white hover:text-indigo-500">
                            <LocationMarkerIcon class="w-5 h-5" aria-hidden="true" @click="getCurrentLocation"/>
                        </div>
                    </div>
                </div>
            </div>
        </Disclosure>
    </div>

    <main class="mt-8">
        <div class="px-4 pb-12 mx-auto max-w-7xl sm:px-6 lg:px-8">
            <div v-if="locationResults.city">
                <div>
                    <h1 class="mb-2 text-2xl font-bold text-gray-700">{{ locationResults ? locationResults.city +',' : 'Finding Location' }} {{ locationResults.county ? locationResults.county +',' : '' }} {{ locationResults.country ? locationResults.country : '' }}</h1>
                </div>

                <div class="max-w-full border border-gray-200 rounded-md">
                    <div v-if="viewToday" class="">
                        <div class="p-4 text-white rounded-t-md" :class="getColour(weather.description)">
                            Today - {{ weather.description }}
                        </div>
                        <div  class="px-4 py-2 text-gray-700 text-md">
                            <div class="flex items-center">
                                <span class="mr-4 text-7xl">{{ weather.temp ? weather.temp.toFixed() + '°' : ''}}</span>
                                <div v-if="weather.max && weather.min" class="lg:grid lg:grid-cols-2">
                                    <div>
                                        <span>Rain {{ weather.rain ? weather.rain + 'mm' : 0 + 'mm' }}</span>
                                    </div>
                                    <div>
                                        <span>Humidity {{ weather.humidity + '%'}}</span>
                                    </div>
                                    <div>
                                        <span>Sunrise {{ String(new Date((weather.sunrise)*1000)).substring(15,21) }} | </span>
                                        <span> Sunset {{ String(new Date((weather.sunset)*1000)).substring(15,21) }}</span>
                                    </div>
                                    <div>
                                        <span> Wind speed {{ weather.windspeed + 'mph' }} | </span>
                                        <span> Wind gusts {{ weather.windgust + 'mph' }}</span>
                                    </div>
                                </div>
                            </div>

                            <div class="items-center grid-flow-col gap-2 pb-4 mt-4">
                                <div class="grid grid-flow-col gap-2 overflow-y-scroll">
                                    <div class="w-1/2" v-for="(hour, key) in weather.hourly" :key="key">
                                    <div v-if="checkTime(hour.dt)">
                                        <div>
                                            {{ String(new Date((hour.dt)*1000)).substring(15,21)}}
                                            </div>
                                            <div class="p-4 border border-gray-400 rounded-md w-36 h-36">
                                                {{ hour.temp.toFixed() + '°' }} <br>
                                                {{ hour.weather[0].description }}
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div v-else>
                        <div class="p-4 text-white rounded-t-md" :class="getColour(selectedDay.weather[0].main)">
                            {{ String(new Date((selectedDay.dt)*1000)).substring(0,10) }} - {{ selectedDay.weather[0].main }}
                        </div>
                        <div  class="px-4 py-2 text-gray-700 text-md">
                            <div class="flex items-center">
                                <span class="mr-4 text-7xl">{{ selectedDay.temp.max.toFixed() + '°' }}</span>
                                <div v-if="selectedDay.temp.max && selectedDay.temp.min" class="lg:grid lg:grid-cols-2">
                                    <div>
                                        <span>Rain {{ selectedDay.rain ? selectedDay.rain + 'mm' : '0mm' }}</span>
                                    </div>
                                    <div>
                                         <span>Humidity {{ selectedDay.humidity + '%'}}</span>
                                    </div>
                                    <div>
                                        <span>Sunrise {{ String(new Date((selectedDay.sunrise)*1000)).substring(15,21) }} | </span>
                                        <span> Sunset {{ String(new Date((selectedDay.sunset)*1000)).substring(15,21) }}</span>
                                    </div>
                                    <div>
                                        <span> Wind speed {{ selectedDay.wind_speed + 'mph' }} | </span>
                                        <span> Wind gusts {{ selectedDay.wind_gust + 'mph' }}</span>
                                    </div>
                                </div>
                            </div>
                            
                        </div>
                    </div>
                </div>
                <div>
                    <div class="grid grid-flow-col gap-2 mt-4 overflow-x-scroll">
                        <div class="" v-for="(day, key) in weather.forcast.daily" :key="key">
                            <div v-if="key <= 0" class="flex flex-col h-48 border border-gray-200 rounded-md cursor-pointer w-36" @click="viewToday = true">
                                <div class="h-24 p-4 text-center text-white rounded-t-md" :class="getColour(weather.description)">
                                    Today
                                </div>
                                <div class="h-full p-2">
                                    {{ weather.temp.toFixed() + '°' }} / {{ weather.feels_like.toFixed()+ '°' }} - {{ weather.description }}
                                </div>
                                <div class="flex justify-center pb-5">
                                    <img v-if="weather.description === 'Clouds'" src="/src/assets/icons8-clouds-32.png" alt="">
                                    <img v-if="weather.description === 'Rain'" src="/src/assets/icons8-rain-32.png" alt="">
                                    <img v-if="weather.description === 'Clear'" src="/src/assets/icons8-sun-32.png" alt="">
                                </div>
                            </div>

                            <div v-else class="flex flex-col h-48 border border-gray-200 rounded-md cursor-pointer w-36" @click="selectDay(day)">
                                <div class="h-24 p-4 text-center text-white rounded-t-md" :class="getColour(day.weather[0].main)">
                                    {{ String(new Date((day.dt)*1000)).substring(0,10) }}
                                </div>
                                <div class="h-full p-2">
                                    {{ day.temp.max.toFixed() + '°' }} / {{ day.temp.min.toFixed() + '°' }} - {{ day.weather[0].main }}
                                </div>
                                <div class="flex justify-center pb-5">
                                    <img v-if="day.weather[0].main === 'Clouds'" src="/src/assets/icons8-clouds-32.png" alt="">
                                    <img v-if="day.weather[0].main === 'Rain'" src="/src/assets/icons8-rain-32.png" alt="">
                                    <img v-if="day.weather[0].main === 'Clear'" src="/src/assets/icons8-sun-32.png" alt="">
                                </div>
                            </div>

                        </div>
                        
                    </div>
                </div>
            </div>
            <div v-else class="flex justify-center text-lg font-bold text-gray-700">
                <h1>Getting Forcast</h1>
            </div>
        </div>
    </main>
        <footer class="flex justify-center px-4 py-2">
            <div class="text-gray-400 ">
                <div>
                    <span>© 2022 Copyright All Rights Reserved</span>
                </div>
            </div>
        </footer>
    </div>
</template>

<script>
//import { ref, onMounted } from 'vue'
import { Disclosure, DisclosureButton, DisclosurePanel, Menu, MenuButton, MenuItem, MenuItems } from '@headlessui/vue'
import { SearchIcon } from '@heroicons/vue/solid'
import { BellIcon, MenuIcon, XIcon, LocationMarkerIcon } from '@heroicons/vue/outline'
import axios from 'axios'

export default {
    components: {
    Disclosure,
    DisclosureButton,
    DisclosurePanel,
    Menu,
    MenuButton,
    MenuItem,
    MenuItems,
    BellIcon,
    MenuIcon,
    SearchIcon,
    XIcon,
    LocationMarkerIcon
    },
    data () {
        return {
            searching: false,
            noLocationsFound: false,
            searchInput: null,
            searchingLocations: [],
            geoLocation: null,
            viewToday: true,
            locationResults: {},
            weather: {
                temp: null,
                humidity: null,
                rain: null,
                sunrise: null,
                sunset: null,
                windspeed: null,
                windgust: null,
                min: null,
                max: null,
                feels_like: null,
                description: null,
                hourly: [],
                forcast: {
                    daily: []
                }
            },
            selectedDay: {
                temp: {
                    max: null,
                    min: null
                },
                weather: [
                    {
                        main: null,
                        description: null
                    }
                ]
            },
            units: 'metric'
        }
    },
    methods: {
        getCurrentLocation() {
            let vm = this;
            var options = {
                enableHighAccuracy: true,
                timeout: 5000,
                maximumAge: 0
            };
            function success(pos) {
                var data = {
                    latitude: pos.coords.latitude,
                    longitude: pos.coords.longitude
                }
                vm.getCurrentLocationName(data)
            }
            function error(err) {
                alert(err.message + '. Please search for a location.')
               // console.warn(`ERROR(${err.code}): ${err.message}`);
                axios.get('https://ip.seeip.org/geoip').then(res => {
                    console.log(res.data);
                    var data = {
                        latitude: res.data.latitude,
                        longitude: res.data.longitude
                    }
                    vm.getCurrentLocationName(data)
                })
            }
            navigator.geolocation.getCurrentPosition(success, error, options);
        },
        getCurrentLocationName(location){
            var searchQuery = location.latitude + ' ' + location.longitude
            axios.get(`https://api.geoapify.com/v1/geocode/search?text=${searchQuery}&format=json&apiKey=${import.meta.env.VITE_LOCATION_API_KEY}`).then(res => {
                this.locationResults = res.data.results[0]
                this.searchInput = res.data.results[0].formatted
                this.getWeather()
            })
        },
        getSearchedLocations() {
            this.noLocationsFound = false
            this.searchingLocations = []
            this.searching = true
            axios.get(`https://api.geoapify.com/v1/geocode/autocomplete?text=${this.searchInput}&format=json&apiKey=${import.meta.env.VITE_LOCATION_API_KEY}`).then(res => {
                if (!res.data.results.length) {
                    this.noLocationsFound = true
                } else {
                    this.searchingLocations = res.data.results
                }
            })
        },
        getSearchedLocation(location, key) {
            this.searching = false
            this.searchInput = location.formatted
            this.locationResults = location
            this.getWeather()

        },
        getWeather() {
            axios.get(`https://api.openweathermap.org/data/2.5/onecall?lat=${this.locationResults.bbox.lat1}&lon=${this.locationResults.bbox.lon1}&units=${this.units}&appid=${import.meta.env.VITE_WEATHER_API_KEY}`).then(res => {
                this.weather.temp = res.data.current.temp
                this.weather.humidity = res.data.current.humidity
                this.weather.sunrise = res.data.current.sunrise
                this.weather.sunset = res.data.current.sunset
                this.weather.feels_like = res.data.current.feels_like
                this.weather.description = res.data.current.weather[0].main
                this.weather.hourly = res.data.hourly
                this.weather.forcast.daily = res.data.daily
                this.weather.max = res.data.daily[0].temp.max
                this.weather.min = res.data.daily[0].temp.min
                this.weather.windspeed = res.data.daily[0].wind_speed
                this.weather.windgust = res.data.daily[0].wind_gust
                this.weather.rain = res.data.daily[0].rain
            })
        },
        selectDay(day) {
            this.viewToday = false
            this.selectedDay = day
            console.log(this.selectedDay.weather);
        },
        getColour(text) {
            //console.log(text);
            var banner = 'bg-gray-400'
            if (text === 'Rain') {
                banner = 'bg-blue-400'
            } else if (text === 'Clouds') {
                banner = 'bg-gray-400'
            } else {
                banner = 'bg-yellow-400'
            }
            return banner
        },
        checkTime(hour) {
            var today = new Date((hour)*1000)
            var tomorrow = new Date()
            tomorrow.setHours(24,0,0,0)
            if (today < tomorrow) {
                return true
            } else {
                return false
            }
        }
        
    },
    computed: {
        
    },
    watch: {
        searchInput(newValue, oldValue) {
            this.noLocationsFound = false
            this.searching = false
        }
    },
    mounted() {
        this.getCurrentLocation()
        this.checkTime()
    }
}
</script>

<style>
body {
    height: 100%;
}
</style>