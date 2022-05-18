<template>
  <div class="min-h-full">
    <div class="bg-indigo-600">
      <Disclosure as="nav" class="bg-indigo-600 border-b border-indigo-300 border-opacity-25 lg:border-none" v-slot="{ open }">
        <div class="px-2 mx-auto max-w-7xl sm:px-4 lg:px-8">
          <div class="relative flex items-center justify-between h-16 lg:border-b lg:border-indigo-400 lg:border-opacity-25">
            <div class="flex items-center px-2 lg:px-0">
                <div class="hidden text-white lg:block">
                    WeatherApp
                    <!-- <img class="block w-8 h-8" src="https://tailwindui.com/img/logos/workflow-mark-indigo-300.svg" alt="Workflow" /> -->
                </div>
                <!-- <div class="hidden lg:block lg:ml-10">
                <div class="flex space-x-4">
                  <a v-for="item in navigation" :key="item.name" :href="item.href" :class="[item.current ? 'bg-indigo-700 text-white' : 'text-white hover:bg-indigo-500 hover:bg-opacity-75', 'rounded-md py-2 px-3 text-sm font-medium']" :aria-current="item.current ? 'page' : undefined">
                    {{ item.name }}
                  </a>
                </div>
              </div> -->
            </div>
            <div class="flex justify-center flex-1 lg:ml-6 lg:justify-end">
              <div class="w-full max-w-lg lg:max-w-lg">
                <label for="search" class="sr-only">Search</label>
                <div class="relative text-gray-400 focus-within:text-gray-600">
                    <div class="absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none">
                        <SearchIcon class="w-5 h-5" aria-hidden="true" />
                    </div>
                    <input id="search" class="block w-full py-2 pl-10 pr-3 leading-5 text-gray-900 placeholder-gray-500 bg-white focus:outline-none sm:text-sm" 
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
              <div class="items-center hidden p-2 ml-4 text-white border-2 border-white rounded-full cursor-pointer md:flex hover:bg-white hover:text-indigo-500"
                    >
                    <LocationMarkerIcon class="w-5 h-5" aria-hidden="true" @click="getCurrentLocation"/>
              </div>
            </div>

            <div class="flex lg:hidden">
              <!-- Mobile menu button -->
              <DisclosureButton class="inline-flex items-center justify-center p-2 text-indigo-200 bg-indigo-600 rounded-md hover:text-white hover:bg-indigo-500 hover:bg-opacity-75 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-offset-indigo-600 focus:ring-white">
                <span class="sr-only">Open main menu</span>
                <MenuIcon v-if="!open" class="block w-6 h-6" aria-hidden="true" />
                <XIcon v-else class="block w-6 h-6" aria-hidden="true" />
              </DisclosureButton>
            </div>

            <!-- <div class="hidden lg:block lg:ml-4">
              <div class="flex items-center">
                <button type="button" class="flex-shrink-0 p-1 text-indigo-200 bg-indigo-600 rounded-full hover:text-white focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-offset-indigo-600 focus:ring-white">
                  <span class="sr-only">View notifications</span>
                  <BellIcon class="w-6 h-6" aria-hidden="true" />
                </button>

                <Menu as="div" class="relative flex-shrink-0 ml-3">
                  <div>
                    <MenuButton class="flex text-sm text-white bg-indigo-600 rounded-full focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-offset-indigo-600 focus:ring-white">
                      <span class="sr-only">Open user menu</span>
                      <img class="w-8 h-8 rounded-full" :src="user.imageUrl" alt="" />
                    </MenuButton>
                  </div>
                  <transition enter-active-class="transition duration-100 ease-out" enter-from-class="transform scale-95 opacity-0" enter-to-class="transform scale-100 opacity-100" leave-active-class="transition duration-75 ease-in" leave-from-class="transform scale-100 opacity-100" leave-to-class="transform scale-95 opacity-0">
                    <MenuItems class="absolute right-0 w-48 py-1 mt-2 origin-top-right bg-white rounded-md shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none">
                      <MenuItem v-for="item in userNavigation" :key="item.name" v-slot="{ active }">
                        <a :href="item.href" :class="[active ? 'bg-gray-100' : '', 'block py-2 px-4 text-sm text-gray-700']">
                          {{ item.name }}
                        </a>
                      </MenuItem>
                    </MenuItems>
                  </transition>
                </Menu>
              </div>
            </div> -->
          </div>
        </div>

        <DisclosurePanel class="lg:hidden">
          <div class="px-2 pt-2 pb-3 space-y-1">
            <DisclosureButton v-for="item in navigation" :key="item.name" as="a" :href="item.href" :class="[item.current ? 'bg-indigo-700 text-white' : 'text-white hover:bg-indigo-500 hover:bg-opacity-75', 'block rounded-md py-2 px-3 text-base font-medium']" :aria-current="item.current ? 'page' : undefined">
              {{ item.name }}
            </DisclosureButton>
          </div>
          <div class="pt-4 pb-3 border-t border-indigo-700">
            <div class="flex items-center px-5">
              <div class="flex-shrink-0">
                <img class="w-10 h-10 rounded-full" :src="user.imageUrl" alt="" />
              </div>
              <div class="ml-3">
                <div class="text-base font-medium text-white">{{ user.name }}</div>
                <div class="text-sm font-medium text-indigo-300">{{ user.email }}</div>
              </div>
              <button type="button" class="flex-shrink-0 p-1 ml-auto text-indigo-200 bg-indigo-600 rounded-full hover:text-white focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-offset-indigo-600 focus:ring-white">
                <span class="sr-only">View notifications</span>
                <BellIcon class="w-6 h-6" aria-hidden="true" />
              </button>
            </div>
            <div class="px-2 mt-3 space-y-1">
              <DisclosureButton v-for="item in userNavigation" :key="item.name" as="a" :href="item.href" class="block px-3 py-2 text-base font-medium text-white rounded-md hover:bg-indigo-500 hover:bg-opacity-75">
                {{ item.name }}
              </DisclosureButton>
            </div>
          </div>
        </DisclosurePanel>
        
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
                                <div v-if="weather.max && weather.min" class="flex w-full justify-evenly">
                                    <div>
                                        <span>Highs of {{ weather.max.toFixed() + '°'}} | </span>
                                        <span>Lows of {{ weather.min.toFixed() + '°'}}</span>
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

                            <div class="items-center hidden h-64 grid-flow-col gap-2 md:grid">
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
                        <div  class="flex flex-col px-4 py-2 text-gray-700 text-md">

                            <div class="flex items-center">
                                <div v-if="selectedDay.temp.max && selectedDay.temp.min" class="flex w-full justify-evenly">
                                    <div>
                                        <span>Highs of {{selectedDay.temp.max.toFixed() + '°'}} | </span>
                                        <span>Lows of {{ selectedDay.temp.min.toFixed() + '°'}}</span>
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
  </div>
</template>

<script>
//import { ref, onMounted } from 'vue'
import { Disclosure, DisclosureButton, DisclosurePanel, Menu, MenuButton, MenuItem, MenuItems } from '@headlessui/vue'
import { SearchIcon } from '@heroicons/vue/solid'
import { BellIcon, MenuIcon, XIcon, LocationMarkerIcon } from '@heroicons/vue/outline'
import axios from 'axios'

const user = {
  name: 'Tom Cook',
  email: 'tom@example.com',
  imageUrl:
    'https://images.unsplash.com/photo-1472099645785-5658abf4ff4e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=facearea&facepad=2&w=256&h=256&q=80',
}
const navigation = [
  { name: 'Dashboard', href: '#', current: true },
  { name: 'Team', href: '#', current: false },
  { name: 'Projects', href: '#', current: false },
  { name: 'Calendar', href: '#', current: false },
  { name: 'Reports', href: '#', current: false },
]
const userNavigation = [
  { name: 'Your Profile', href: '#' },
  { name: 'Settings', href: '#' },
  { name: 'Sign out', href: '#' },
]

// api key 8ace09d069a441f3aa8225743221504

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
                console.warn(`ERROR(${err.code}): ${err.message}`);
                axios.get('https://ip.seeip.org/geoip').then(res => {
                    //console.log(res.data);
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
            axios.get(`https://api.geoapify.com/v1/geocode/search?text=${searchQuery}&format=json&apiKey=${'7a651eee8519432596636c4cc8ef8de7'}`).then(res => {
                this.locationResults = res.data.results[0]
                this.getWeather()
            })
        },
        getSearchedLocations() {
            this.noLocationsFound = false
            this.searchingLocations = []
            this.searching = true
            axios.get(`https://api.geoapify.com/v1/geocode/autocomplete?text=${this.searchInput}&format=json&apiKey=${'7a651eee8519432596636c4cc8ef8de7'}`).then(res => {
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
            axios.get(`https://api.openweathermap.org/data/2.5/onecall?lat=${this.locationResults.bbox.lat1}&lon=${this.locationResults.bbox.lon1}&units=${this.units}&appid=1a9be1417e0b10b37966c6b492063917`).then(res => {
                this.weather.temp = res.data.current.temp
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