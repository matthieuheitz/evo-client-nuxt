<template>
  <v-container class="license-plate-search">
    <v-row justify="center" class="mt-4">
      <v-col cols="12" sm="8" md="6">
        <v-card class="pa-6">
          <v-card-title class="justify-center mb-4">
            <h2>Find Vehicle by License Plate</h2>
          </v-card-title>

          <v-form @submit.prevent="searchPlate">
            <v-text-field
              v-model="plateInput"
              label="Enter License Plate"
              placeholder="e.g., ABC 1234"
              outlined
              clearable
              @keyup.enter="searchPlate"
              :disabled="searching"
              :rules="[plateRules.required, plateRules.format]"
              hint="Enter a valid license plate (letters and numbers)"
              persistent-hint
            />

            <v-btn
              type="submit"
              color="primary"
              block
              large
              class="mt-4"
              :loading="searching"
              :disabled="!isValidPlate"
            >
              Search
            </v-btn>
          </v-form>

          <!-- Search Results -->
          <v-divider class="my-6" v-if="searchPerformed" />

          <div v-if="searchPerformed">
            <v-alert
              v-if="searchError"
              type="error"
              dismissible
              class="mb-4"
            >
              {{ searchError }}
            </v-alert>

            <v-alert
              v-if="!foundVehicle && !searchError"
              type="info"
              dismissible
              class="mb-4"
            >
              No vehicle found with license plate "{{ lastSearchedPlate }}". Make sure the plate is currently available.
            </v-alert>

            <v-card v-if="foundVehicle" class="mt-4 vehicle-result" elevation="4">
              <v-card-title>Vehicle Found!</v-card-title>
              <v-divider />
              <v-card-text>
                <v-list dense>
                  <v-list-item>
                    <v-list-item-icon>
                      <v-icon>mdi-license-plate</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                      <v-list-item-title>License Plate</v-list-item-title>
                      <v-list-item-subtitle>{{ foundVehicle.description.plate }}</v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>

                  <v-list-item>
                    <v-list-item-icon>
                      <v-icon>mdi-car</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                      <v-list-item-title>Model</v-list-item-title>
                      <v-list-item-subtitle>{{ foundVehicle.description.model }}</v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>

                  <v-list-item v-if="foundVehicle.distance">
                    <v-list-item-icon>
                      <v-icon>mdi-map-marker-distance</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                      <v-list-item-title>Distance from You</v-list-item-title>
                      <v-list-item-subtitle>{{ distanceString(foundVehicle.distance) }}</v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>

                  <v-list-item>
                    <v-list-item-icon>
                      <v-icon>mdi-gas-station</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                      <v-list-item-title>Fuel Level</v-list-item-title>
                      <v-list-item-subtitle>{{ foundVehicle.status.energyLevel }}%</v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>

                  <v-list-item>
                    <v-list-item-icon>
                      <v-icon>mdi-map-marker</v-icon>
                    </v-list-item-icon>
                    <v-list-item-content>
                      <v-list-item-title>Location</v-list-item-title>
                      <v-list-item-subtitle>
                        {{ foundVehicle.location.position.lat.toFixed(5) }},
                        {{ foundVehicle.location.position.lon.toFixed(5) }}
                      </v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>
                </v-list>
              </v-card-text>
              <v-card-actions>
                <v-spacer />
                <v-btn color="primary" :to="{ name: 'index', query: { plate: foundVehicle.description.plate } }">
                  <v-icon left>mdi-map</v-icon>
                  View on Map
                </v-btn>
              </v-card-actions>
            </v-card>
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { mapState } from 'vuex'

export default {
  name: 'LicensePlateSearch',
  data () {
    return {
      plateInput: '',
      searching: false,
      searchPerformed: false,
      searchError: '',
      foundVehicle: null,
      lastSearchedPlate: '',
      plateRules: {
        required: v => !!v || 'License plate is required',
        format: v => !v || /^[a-zA-Z0-9\s\-]{2,}$/.test(v) || 'Enter a valid license plate'
      }
    }
  },
  computed: {
    ...mapState(['vehicles']),
    isValidPlate () {
      return this.plateInput && this.plateRules.format(this.plateInput) === true
    }
  },
  methods: {
    /**
     * Search for a vehicle by license plate
     */
    searchPlate () {
      if (!this.isValidPlate) {
        return
      }

      this.searching = true
      this.searchError = ''
      this.foundVehicle = null
      this.lastSearchedPlate = this.plateInput.toUpperCase().trim()

      // Simulate a small delay for better UX
      setTimeout(() => {
        try {
          const found = this.vehicles.find(
            vehicle => vehicle.description.plate.toUpperCase() === this.lastSearchedPlate
          )

          if (found) {
            this.foundVehicle = found
          } else {
            this.foundVehicle = null
          }
        } catch (error) {
          this.searchError = 'An error occurred while searching. Please try again.'
          console.error('License plate search error:', error)
        } finally {
          this.searching = false
          this.searchPerformed = true
        }
      }, 300)
    },

    /**
     * Format distance for display
     * @param {number} distance - Distance in meters
     * @returns {string}
     */
    distanceString (distance) {
      if (!distance) return 'N/A'

      let output = `${distance} m`
      if (distance > 999) {
        const km = Math.round(distance / 10) / 100
        output = `${km} km`
      }
      return output
    }
  }
}
</script>

<style scoped lang="scss">
.license-plate-search {
  min-height: calc(100vh - 56px);
  padding-top: 2rem;
  padding-bottom: 2rem;

  @media screen and (min-width: 950px) {
    min-height: calc(100vh - 64px);
  }
}

.vehicle-result {
  animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>