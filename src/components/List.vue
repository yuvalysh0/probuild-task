<template>
  <div class="container">
    <div class="head">
      <h2 class="text-center">List Of Breweries</h2>
      <v-btn
          color="pink"
          dark
          elevation="2"
          rounded
          small
          @click="dialog = true"
      >Add New Brewery
      </v-btn>
    </div>

    <v-expansion-panels
        class="rounded-xl"
        accordion
    >
      <v-expansion-panel
          v-for="(item,i) in arr"
          :key="i"
      >
        <v-expansion-panel-header>
          {{ item.stateName }}
        </v-expansion-panel-header>
        <v-expansion-panel-content>
          <v-simple-table class="table">
            <template v-slot:default>
              <thead>
              <tr>
                <th class="text-left">
                  City
                </th>
                <th class="text-left">
                  Street
                </th>
                <th class="text-left">
                  Actions
                </th>
              </tr>
              </thead>
              <tbody>
              <tr
                  v-for="(obj,i) of item.breweries"
                  :key="i"
              >
                <td>{{ obj.city }}</td>
                <td>{{ obj.street }}</td>
                <td class="actions">
                  <v-btn
                      @click='openUpdateDialog(item, obj)'
                      icon
                      color="pink darken-2"
                  >
                    <v-icon>mdi-pencil</v-icon>
                  </v-btn>
                  <v-btn
                      @click=deleteBrewerie(item.stateName,obj)
                      icon
                      color="grey darken-1"
                  >
                    <v-icon>mdi-delete</v-icon>
                  </v-btn>
                </td>
              </tr>
              </tbody>
            </template>
          </v-simple-table>
        </v-expansion-panel-content>
      </v-expansion-panel
      >
    </v-expansion-panels>

    <v-row justify="center">
      <v-dialog
          v-model="dialog"
          persistent
          max-width="600px"
      >
        <v-card>
          <v-card-title>
            <span class="text-h5">Add new brewerie</span>
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col
                    cols="12"
                    sm="6"
                >
                  <v-text-field
                      v-model="breweieForm.state"
                      label="State"
                      required
                  ></v-text-field>
                </v-col>
                <v-col
                    cols="12"
                    sm="6"
                >
                  <v-text-field
                      v-model="breweieForm.city"
                      label="City"
                  ></v-text-field>
                </v-col>
                <v-col
                    cols="12"
                    sm="6"
                >
                  <v-text-field
                      v-model="breweieForm.street"
                      label="Street"
                  ></v-text-field>
                </v-col>
                <v-col
                    cols="12"
                    sm="6"
                >
                  <v-text-field
                      label="Name"
                      v-model="breweieForm.name"
                      required
                  ></v-text-field>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
                color="blue darken-1"
                text
                @click="dialog = false"
            >
              Close
            </v-btn>
            <v-btn
                color="blue darken-1"
                text
                @click="createNewBrewerie()"
            >
              Add
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-row>

    <v-row justify="center">
      <v-dialog
          v-model="updateDialog"
          persistent
          max-width="600px"
      >
        <v-card>
          <v-card-title>
            <span class="text-h5">Update brewerie</span>
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col
                    cols="12"
                    sm="6"
                >
                  <v-text-field
                      v-model="breweieForm.state"
                      label="State"
                      required
                  ></v-text-field>
                </v-col>
                <v-col
                    cols="12"
                    sm="6"
                >
                  <v-text-field
                      v-model="breweieForm.city"
                      label="City"
                  ></v-text-field>
                </v-col>
                <v-col
                    cols="12"
                >
                  <v-text-field
                      v-model="breweieForm.street"
                      label="Street"
                  ></v-text-field>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
                color="blue darken-1"
                text
                @click="resetFields()"
            >
              Close
            </v-btn>
            <v-btn
                color="blue darken-1"
                text
                @click="updateBrewerie()"
            >
              Update
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-row>

  </div>
</template>

<script>
import Axios from 'axios'

export default {

  data() {
    return {
      dialog: false,
      updateDialog: false,
      mainObj: {},
      arr: [],
      isArr: false,
      updatedItem: {},
      updatedBrewerie: {},
      breweieForm: {
        state: '',
        city: '',
        street: '',
        name: ''
      }
    }
  },

  methods: {

    fetchApi() {
      return Axios.get('https://api.openbrewerydb.org/breweries')
          .then(response => {
            return response;
          })
          .catch(error => {
            console.log(error)
          });
    },

    createUniqueArr(arr) {
      return [...new Set(arr)];
    },

    sortingArray(breweries) {
      breweries.sort((a, b) => {
        let fa = a.city.toLowerCase(),
            fb = b.city.toLowerCase();

        if (fa < fb) {
          return -1;
        }
        if (fa > fb) {
          return 1;
        }
        return 0;
      });
      return breweries
    },

    createArr(arr, obj) {
      let newArr = []

      for (let state of arr) {
        let breweries = []
        let allStateBreweries = Object.entries(obj[state]['breweries'])
        for (let i = 0; i < allStateBreweries.length; i++) {
          breweries.push(allStateBreweries[i][1])
        }
        breweries = this.sortingArray(breweries)
        newArr.push({
          stateName: state,
          breweries
        })
      }
      this.arr = newArr
    },

    openUpdateDialog(item, brewerie) {
      console.log('Update brewerie', item, brewerie)
      this.breweieForm.city = brewerie.city
      this.breweieForm.street = brewerie.street
      this.breweieForm.state = item.stateName
      this.updateDialog = true

      this.updatedItem = item
      this.updatedBrewerie = brewerie
    },

    updateBrewerie() {
      this.updateDialog = false
      //Checking if the user changed the state
      if (this.breweieForm.state !== this.updatedItem.stateName) {
        this.deleteBrewerie(this.updatedItem.stateName, this.updatedBrewerie)
        this.createNewBrewerie()
        this.resetFields()
        return
      }

      const updateFields = {
        city: this.breweieForm.city,
        street: this.breweieForm.street
      }

      // Updating the bewerie in the mainObj
      for (const key in this.mainObj[this.updatedItem.stateName]['breweries']) {
        if (this.updatedBrewerie === this.mainObj[this.updatedItem.stateName]['breweries'][key]) {
          this.mainObj[this.updatedItem.stateName]['breweries'][key] = updateFields
        }
      }

      //Find the index of the brewerie state
      let index = this.arr.findIndex(x => x.stateName === this.breweieForm.state)

      // Updating the brewerie in the Array
      for (let i = 0; i < this.arr[index].breweries.length; i++) {
        if (this.arr[index].breweries[i] === this.updatedBrewerie) {
          this.arr[index].breweries[i] = updateFields
        }
      }

      this.resetFields()

    },

    deleteBrewerie(state, brewerie) {

      // Deleting the bewerie from the mainObj
      for (const key in this.mainObj[state]['breweries']) {
        if (brewerie === this.mainObj[state]['breweries'][key]) {
          delete this.mainObj[state]['breweries'][key]
        }
      }

      //Find the index of the brewerie state
      let index = this.arr.findIndex(x => x.stateName === state)

      //If there is only one brewerie, the state will be removed
      if (this.arr[index].breweries.length === 1) {
        console.log('The state we are deleting from the array: ', this.arr[index])
        this.arr.splice(index, 1)
        delete this.mainObj[state]
      }

      //Removing the brewerie from the array
      else {
        for (let i = 0; i < this.arr[index].breweries.length; i++) {
          if (this.arr[index].breweries[i] === brewerie) {
            console.log('The brewerie we are deleting from the array: ', this.arr[index].breweries[i])
            this.arr[index].breweries.splice(i, 1)
          }
        }
      }
    },

    createNewBrewerie() {
      this.dialog = false
      const state = this.breweieForm.state
      const brewerie = {
        city: this.breweieForm.city,
        street: this.breweieForm.street
      }
      let newState = false

      //Check if there is already the state
      if (!this.mainObj[state]) {
        newState = true
        this.mainObj[state] = {
          stateName: state,
          breweries: {}
        }
      }

      //Add the new brewerie to the mainObj
      this.mainObj[state]['breweries'][this.breweieForm.name] = brewerie


      if (newState) {
        let breweries = []
        let allStateBreweries = Object.entries(this.mainObj[state]['breweries'])
        for (let i = 0; i < allStateBreweries.length; i++) {
          breweries.push(allStateBreweries[i][1])
        }
        breweries = this.sortingArray(breweries)
        this.arr.push({
          stateName: state,
          breweries
        })
      } else {

        //finding the index and pushing the new brewerie
        let index = this.arr.findIndex(x => x.stateName === this.breweieForm.state)

        console.log(this.arr[index].breweries)
        this.arr[index].breweries.push(brewerie)

        //Sorting the breweries by names
        this.arr[index].breweries.sort((a, b) => {
          let fa = a.city.toLowerCase(),
              fb = b.city.toLowerCase();
          if (fa < fb) {
            return -1;
          }
          if (fa > fb) {
            return 1;
          }
          return 0;
        });
      }

      //Sorting the table by state names
      this.arr.sort((a, b) => {
        let fa = a.stateName.toLowerCase(),
            fb = b.stateName.toLowerCase();

        if (fa < fb) {
          return -1;
        }
        if (fa > fb) {
          return 1;
        }
        return 0;
      });


      //reset the brewerieForm fields
      this.resetFields()
    },

    resetFields() {
      //Reset all the fields
      this.updateDialog = false
      this.breweieForm.street = ''
      this.breweieForm.city = ''
      this.breweieForm.state = ''
      this.breweieForm.name = ''
      this.updatedItem = {}
      this.updatedBrewerie = {}
    }

  },

  created() {
    this.fetchApi().then((result) => {
      this.mainObj = result.data

      let newObj = {}
      let statesArr = []

      //Creating the hashmap object
      for (let obj of this.mainObj) {

        //Check if there is any state
        if (obj.state === null)
          continue

        //Pushing the name of the state to the array
        statesArr.push(obj.state)

        if (!newObj[obj.state]) {
          newObj[obj.state] = {
            stateName: obj.state,
            breweries: {}
          }
        }

        newObj[obj.state]['breweries'][obj.id] = {
          city: obj.city,
          street: obj.street
        }

        // Sorting the object by city names
        Object.keys(newObj[obj.state]['breweries']).sort(function (a, b) {
          return newObj[obj.state]['breweries'][a] - newObj[obj.state]['breweries'][b]
        })
      }

      //Removing all the duplicates states from the array
      statesArr = this.createUniqueArr(statesArr)

      //Sorting the array by state
      statesArr.sort()

      this.createArr(statesArr, newObj)
      this.mainObj = newObj
    })

  }

}
</script>


<style scoped>

.container {
  max-width: 1400px;
  margin: 1em auto;
  padding: 1em;
}

.head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1em;
  margin-bottom: 1em;
}

.table {
  width: 100%;
  margin: 0 auto;
}

.actions {
  display: flex;
  gap: 1em;
  align-items: center;
  justify-content: flex-start;
}
</style>
