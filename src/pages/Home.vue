<template>
  <q-page class="container">
    <div class="drawer">
      <div class="column ">
        <div class="col-2 addressForm rounded-borders q-pa-sm">
          <q-form @submit="onSubmit">
            <q-input v-if="errorMessage" filled v-model="errorMessage" dense disable />
            <q-input for="autocomplete" filled bottom-slots v-model="address" label="Enter address" dense>
              <template v-slot:append>
                <q-btn flat round color="primary" icon="place" @click="locate"/>
              </template>
            </q-input>
              <div class="row">
                <div>
                    <q-btn label="Find Bathrooms" type="submit" color="primary" @click="findNearbyPlaces"/>
                  </div>

                <q-space />

                <div>
                  <q-select filled v-model="distance" :options="distanceDropdown" label="miles" dense class="milesDropdown"/>
                </div>
              </div>
          </q-form>
        </div>
        
        <div class="placesForm">
          <div class="row q-pa-sm">
                  <div>
                      <q-toggle v-model="menuState" label="Hide Results" />
                    </div>

                  <q-space />
                  <div class="q-pr-sm">
                    <q-btn-dropdown color="primary" label="Sort">
                    <q-list>
                      <q-item v-for="value in sortValues" :key="value" clickable v-close-popup @click="onItemClick">
                        <q-item-section>
                            <q-item-label>{{ value }}</q-item-label>
                          </q-item-section>
                      </q-item>
                    </q-list>
                  </q-btn-dropdown>
                  </div>
                  
                  <div>
                    <q-btn-dropdown color="primary" label="Filters">
                      <q-list class="filterWindow">
                        <q-item clickable v-close-popup @click="onItemClick">
                          <div class="q-pa-md filterSlider" dense>
                            <div>Rating</div>
                            <q-slider
                              v-model="ratingSlider"
                              :min="0"
                              :max="5"
                              :step="1"
                              snap
                              label
                              label-always
                              color="purple"
                            />
                          </div>
                        </q-item>

                        <q-item clickable v-close-popup @click="onItemClick">
                          <div class="q-pa-md filterSlider" dense>
                            <div>Toilet Paper Ply</div>
                            <q-slider
                              v-model="plySlider"
                              :min="1"
                              :max="3"
                              :step="1"
                              snap
                              label
                              label-always
                              color="accent"
                            />
                          </div>
                        </q-item>

                        <q-item clickable v-close-popup @click="onItemClick">
                          <div class="q-pa-sm">
                            <q-option-group
                              v-model="group"
                              :options="options"
                              color="accent"
                              type="toggle"
                            />
                          </div>
                        </q-item>
                      </q-list>
                      <div class="row q-pa-sm">
                        <q-space />
                        <div class="q-pr-sm">
                          <q-btn label="Reset" type="submit" color="primary" @click=""/>
                        </div>
                        <div>
                          <q-btn label="Apply" type="submit" color="primary" @click=""/>
                        </div>
                      </div>
                    </q-btn-dropdown>
                  </div>
                </div>
          <div v-show="places" class="col-10 placesList rounded-borders q-pa-sm">
            <q-list separator  class="item" v-for="(place, index) in places" :key="place.id" @click="goToMarker(index)">
              <q-item clickable ripple>
                <q-item-section>
                  <q-item-label class="header text-h6">{{ place.name }}
                    <span class="text-caption" v-if="place.opening_hours.open_now" style="color:#00DB00;"> 
                      Open
                    </span>
                    <span class="text-caption" v-else style="color:red;"> 
                      Closed
                    </span>

                      </q-item-label>
                    <q-item-label caption class="details">{{ place.vicinity }}</q-item-label>
                    <q-item-label caption class="details">
                      <q-rating
                        v-model="place.rating"
                        size="2em"
                        :max="5"
                        color="accent"
                        disable>
                        <template v-slot:tip-1>
                          <q-tooltip>Dirty</q-tooltip>
                        </template>
                        <template v-slot:tip-2>
                          <q-tooltip>Not Clean</q-tooltip>
                        </template>
                        <template v-slot:tip-3>
                          <q-tooltip>Somewhat Clean</q-tooltip>
                        </template>
                        <template v-slot:tip-4>
                          <q-tooltip>Clean</q-tooltip>
                        </template>
                        <template v-slot:tip-5>
                          <q-tooltip>The Cleanest</q-tooltip>
                        </template>
                      </q-rating>({{ place.user_ratings_total }} reviews)
                    </q-item-label>
                  </q-item-section>
                  
                </q-item>
              </q-list>
          </div>
      </div>
    </div>
    
      
      <div id="map"/>   
    </div>
  </q-page>
</template>

<script>
import axios from 'axios'

const addressData = ''
export default {
  data () {
    return {
      name: null,
      age: null,
      isError:false,
      errorMessage:'',
      apiKey: 'AIzaSyAY6CeOoc2nfaGqlughybBkOMbhkxDlXhE',
      address:'',
      lat:'',
      lng:'',
      distanceDropdown: [
        '1', '2', '3', '5', '10', '15'
      ],
      distance: 1,
      type: 'restaurant',
      places: [],
      ratingModel:'',
      markers:[],
      menuState: false,
      sortValues: ['Rating', 'Distance'],
      ratingSlider:3,
      plySlider:2,
      group: [ 'op1' ],
      options: [
        {
          label: 'Handicap Accessible',
          value: '1'
        },
        {
          label: 'Public Bathroom',
          value: '2'
        },
        {
          label: 'More Than 1 Person Allowed',
          value: '3'
        }
      ]
    }
  },
  mounted() {
    this.locate();
    //autocompletes address field based on addresses near the user's lat lng
    let autocomplete = new google.maps.places.Autocomplete(document.getElementById("autocomplete"), {
      bounds: new google.maps.LatLngBounds( 
        new google.maps.LatLng (this.lat, this.lng)
      )
    })
    autocomplete.addListener("place_changed", ()=> {
      let place = autocomplete.getPlace();
      this.showUserOnMap(place.geometry.location.lat(), place.geometry.location.lng())
    })
  },
  methods: {
    onSubmit () {

    },
    locate() {
      //check if browser is compatible with google api
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          position => {
            this.lat = position.coords.latitude;
            this.lng = position.coords.longitude;

            this.getAddressFromLatLng(this.lat, this.lng);
            this.showUserOnMap(this.lat, this.lng);
            this.findNearbyPlaces();
          },
          error => {
            this.errorMessage = error.message;
            console.log(error.message);
          }
        )
      }
      else {
        this.errorMessage = error.message;
      }
    },
    getAddressFromLatLng(lat, lng) {
      axios.get("https://maps.googleapis.com/maps/api/geocode/json?latlng=" + lat + "," + lng + "&key=" + this.apiKey)
      .then (response => {
        if(response.data.error_message) {
          console.log(response.data.error_message);
          this.errorMessage = response.data.error_message;
        }
        else {
          this.address = response.data.results[0].formatted_address;
        }
      })
      .catch (error => {
        this.errorMessage = error.message;
      })
    },
    showUserOnMap(lat, lng) {
      //create the map 
      const map = new google.maps.Map(document.getElementById("map"), {
        zoom:15,
        center: new google.maps.LatLng(lat, lng),
        mapTypeId:google.maps.MapTypeId.ROADMAP
      });
      //show user marker
      new google.maps.Marker({
        position: new google.maps.LatLng(lat, lng),
        map:map
      })
    },
    findNearbyPlaces() {
      const URL = `https://cors-anywhere.herokuapp.com/https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=${this.lat}
      ,${this.lng}&type=${this.type}&radius=${this.distance * 1000}&key=${this.apiKey}`
      axios.get(URL).then(response => {
        
        this.places = response.data.results;
        this.showPlaceMarkers();
      })
      .catch(error => {
        this.errorMessage = error.message;
      })
    },
    showPlaceMarkers() {
      const map = new google.maps.Map(document.getElementById("map"), {
        zoom:15,
        center: new google.maps.LatLng(this.lat, this.lng),
        mapTypeId:google.maps.MapTypeId.ROADMAP
      });
      
      const detailWindow = new google.maps.InfoWindow();
      const image = 'https://icon-icons.com/icons2/2064/PNG/48/toilet_bathroom_restroom_wc_icon_124711.png';
      
      for(let i=0; i < this.places.length; i++) {
        const lat = this.places[i].geometry.location.lat;
        const lng = this.places[i].geometry.location.lng;
        const placeID = this.places[i].place_id;

        const marker = new google.maps.Marker({
          position: new google.maps.LatLng(lat, lng),
          map: map,
          animation: google.maps.Animation.DROP,
          icon: image
        });
        
        //add marker into markers array
        this.markers.push(marker);

        //show user marker
        new google.maps.Marker({
        position: new google.maps.LatLng(this.lat, this.lng),
        map:map
      });
        //use arrow function so you can access data from ourside this method
        google.maps.event.addListener(marker, "click", ()=>{
          const URL = `https://cors-anywhere.herokuapp.com/https://maps.googleapis.com/maps/api/place/details/json?key=${this.apiKey}
          &place_id=${placeID}`;

          axios
            .get(URL)
            .then(response => {
              if (response.data.error_message) {
                this.errorMessage = response.data.error_message;
              } else {
                const place = response.data.result;

                detailWindow.setContent(
                  `<div class="text-h6">${place.name}</div>
                  ${place.formatted_address} <br>
                  ${place.formatted_phone_number} <br>
                  <a href="${place.website}" target="_blank">${place.website}</a> <br>
                  Rating: ${place.rating} (${place.user_ratings_total} reviews)`
                );
                detailWindow.open(map, marker);
              }
            })
            .catch(error => {
              this.errorMessage = error.message;
            });
        });
      }
    },
    goToMarker(index) {
      this.activeIndex = index;
      new google.maps.event.trigger(this.markers[index], "click");
    }
  }
}
</script>

<style lang="scss">
.addressForm {
  height:130px;
  background:white;
  border: 2px solid #cccccc;
  z-index:0;
}

.milesDropdown {
  width:100px;
}

.placesForm {
  width:500px;
  background:white;
  border: 2px solid #cccccc;
  z-index:0;
}

.placesList {
  overflow:auto;
  height:calc(100vh - 231px);
}

.selectedItem {
  background: #c8f9ff !important;
}

.drawer {
  width:500px;
}

.filterWindow {
  width:350px;
}

.filterSlider {
  width:350px;
  height:60px;
}
#map {
  position:absolute;
  top:0;
  left:500px;
  bottom:0;
  right:0;
  height:calc(100vh - 53px);
  z-index:0;
  background-attachment: fixed;
}
</style>
