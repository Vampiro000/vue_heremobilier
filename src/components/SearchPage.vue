<template>
  <v-card>
    <v-app-bar
      color="indigo"
      dark
      hide-on-scroll
      prominent
      scroll-target="#scrolling-techniques-4"
    >
      <v-flex column>
        <v-flex style="height: 45px">
          <v-autocomplete
            v-model="selectedLocation"
            :search-input.sync="searchedLocation"
            :items="searchedResultLocation"
            chips
            label="Ville, Département, Code Postale"
            single-line
            close
            :item-text="nameprocess"
            return-object
            small-chips
            rounded
            clearable
            filled
            dense
            prepend-inner-icon="mdi-map-search"
            multiple
          >
            <template v-slot:selection="data">
              <v-chip
                v-bind="data.attrs"
                :input-value="data.selected"
                close
                @click="data.select"
                @click:close="remove(data.item)"
              >
                <div style="margin-right: 3px">{{ data.item.nom }}</div>
                <!-- <v-chip color="grey" small>{{ data.item.codepostal }}</v-chip> -->
                <span style="font-weight: bold; margin-left: 3px">
                  {{ data.item.codepostal }}</span
                >
              </v-chip>
            </template>
          </v-autocomplete>
        </v-flex>
        <v-flex style="height: 45px">
          <div>
            <v-combobox
              v-model="selectedTypeOfHouses"
              :items="typeOfHouses"
              label="  Type de Bien"
              single-line
              multiple
              clearable
              rounded
              prepend-inner-icon="mdi-home-city"
              chips
              small-chips
              filled
              dense
            ></v-combobox>
          </div>
        </v-flex>
        <v-flex>
          <div style="display: flex; justify-content: space-around">
            <div style="display: flex">
              <v-dialog v-model="dialog" persistent width="500">
                <template v-slot:activator="{ on, attrs }">
                  <v-btn
                    color="white"
                    dark
                    outlined
                    rounded
                    v-bind="attrs"
                    small
                    v-on="on"
                  >
                    <v-icon left dark> mdi-filter-plus </v-icon> Filtre
                  </v-btn>
                </template>

                <v-card>
                  <v-card-title class="headline grey lighten-2">
                    Filtre sur Prix,Surface,Date
                  </v-card-title>

                  <v-card-text>
                    <v-row>
                      <v-col class="px-4">
                        <v-card-subtitle> Intervalle de Prix </v-card-subtitle>
                        <v-range-slider
                          v-model="valuePrice"
                          :max="maxPrice"
                          :min="minPrice"
                          hide-details
                          class="align-center"
                        >
                          <template v-slot:prepend>
                            <v-chip>
                              {{ valuePrice[0] }}
                            </v-chip>
                          </template>
                          <template v-slot:append>
                            <v-chip> {{ valuePrice[1] }}</v-chip>
                          </template>
                        </v-range-slider>
                        <v-card-subtitle>
                          Intervalle de surface en metre carré
                        </v-card-subtitle>
                        <v-range-slider
                          v-model="valueSurface"
                          :max="maxSurface"
                          :min="minSurface"
                          hide-details
                          class="align-center"
                        >
                          <template v-slot:prepend>
                            <v-chip>
                              {{ valueSurface[0] }}
                            </v-chip>
                          </template>
                          <template v-slot:append>
                            <v-chip> {{ valueSurface[1] }}</v-chip>
                          </template>
                        </v-range-slider>
                        <v-flex
                          style="
                            padding-top: 20px;
                            display: flex;
                            justify-content: space-around;
                          "
                        >
                          <div style="max-width: 150px; display: inline-block">
                            <v-menu
                              v-model="menu"
                              :close-on-content-click="false"
                              :nudge-right="40"
                              transition="scale-transition"
                              offset-y
                              min-width="auto"
                            >
                              <template v-slot:activator="{ on, attrs }">
                                <v-text-field
                                  v-model="from"
                                  label="Depuis"
                                  prepend-icon="mdi-calendar"
                                  readonly
                                  v-bind="attrs"
                                  v-on="on"
                                ></v-text-field>
                              </template>
                              <v-date-picker v-model="from"></v-date-picker>
                            </v-menu>
                          </div>
                          <div style="max-width: 150px; display: inline-block">
                            <v-menu
                              v-model="menu2"
                              :close-on-content-click="false"
                              :nudge-right="40"
                              transition="scale-transition"
                              offset-y
                              min-width="auto"
                            >
                              <template v-slot:activator="{ on, attrs }">
                                <v-text-field
                                  v-model="to"
                                  label="Jusqu'à"
                                  prepend-icon="mdi-calendar"
                                  readonly
                                  v-bind="attrs"
                                  v-on="on"
                                ></v-text-field>
                              </template>
                              <v-date-picker v-model="to"></v-date-picker>
                            </v-menu>
                          </div>
                        </v-flex>
                      </v-col>
                    </v-row>
                  </v-card-text>

                  <v-divider></v-divider>

                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="primary" text @click="dialog = false">
                      Valider
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </div>
            <div style="display: flex">
              <v-btn
                :loading="loadingHomes"
                :disabled="loadingHomes"
                rounded
                small
                dense
                color="info"
                @click="(loader = 'lodingHomes'), requestHomes()"
              >
                Rechercher
                <template v-slot:loader>
                  <span>
                    <v-icon light>mdi-cached</v-icon>
                  </span>
                </template>
              </v-btn>
            </div>
          </div>
        </v-flex>
        <div>
          {{ searchedLocation }}
          {{ selectedLocation }}
          {{ searchedResultLocation }}
        </div>
      </v-flex>
    </v-app-bar>
  </v-card>
</template>

<script>
import axios from "axios";
// const apiUrl = "http://192.168.137.1:3000/adressHelper?adress=";
export default {
  name: "HelloWorld",
  methods: {
    remove(item) {
      const index = this.selectedLocation.indexOf(item);
      if (index >= 0) this.selectedLocation.splice(index, 1);
    },
    requestHomes() {
      alert("Lance la Recherche");
      let HomesFilter = {
        selectedLocation: this.selectedLocation,
        selectedTypeOfHouses: this.selectedTypeOfHouses.length
          ? this.selectedTypeOfHouses
          : undefined,
        valueSurface: this.valueSurface,
        valuePrice: this.valuePrice,
        from: this.from,
        to: this.to,
      };

      axios
        .post("http://localhost:3000/requestHomes", HomesFilter)
        .then((response) => {
          console.log("Response", response);
          this.loadingHomes = false;
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
  data: () => ({
    from: new Date().toISOString().substr(0, 10),
    to: new Date().toISOString().substr(0, 10),
    menu: false,
    menu2: false,
    minPrice: 0,
    Homes: [],
    loadingHomes: false,
    maxPrice: 10000000,
    minSurface: 9,
    maxSurface: 10000,
    valuePrice: [0, 10000000],
    valueSurface: [9, 10000],
    dialog: false,
    searchedResultLocation: [],
    searchedLocation: "",
    selectedLocation: [],
    typeOfHouses: ["Terrain", "Maison", "Appartement"],
    selectedTypeOfHouses: [],
    items: null,
    nameprocess: (item) => {
      if (item.codepostal) {
        return item.nom + "  " + item.codepostal;
      } else {
        return item.nom;
      }
    },
  }),
  computed: {
    item: () => {},
  },
  watch: {
    searchedLocation(val) {
      console.log("HERE", val);
      if (val && val.length >= 3) {
        this.searchedResultLocation.length = 0;
        this.selectedLocation.length = 0;
        axios
          .get("http://localhost:3000/adressHelper?adress=" + val)
          .then((response) => {
            console.log("Response", response);
            for (const city of response.data) {
              if (city.codesPostaux) {
                for (const codepostal of city.codesPostaux) {
                  console.log("codePostal", codepostal);
                  this.searchedResultLocation.push({
                    nom: city.nom,
                    codepostal: codepostal,
                  });
                }
                this.searchedResultLocation.push({ nom: city.nom });
              }
            }
            // this.wholeResponse = response.data.Search
          })
          .catch((error) => {
            console.log(error);
          });
      }
    },
  },
};
</script>
