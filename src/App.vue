<template>
  <v-app>
    <v-app-bar app>
      <v-spacer></v-spacer>
      <v-toolbar-title>Public APIs 🌐</v-toolbar-title>
      <v-spacer></v-spacer>
    </v-app-bar>
    <v-main class="grey lighten-2">
      <v-alert tile v-if="errorAlert" type="error">
        Unknown error occured
      </v-alert>
      <v-container class="mt-10">
        <v-row>
          <v-col cols="12">
            <div v-if="loading" class="text-center">
              <v-progress-circular
                indeterminate
                color="primary"
              ></v-progress-circular>
            </div>
            <v-fade-transition hide-on-leave>
              <div v-if="!loading && !errorAlert">
                <v-data-table
                  :headers="headers"
                  :items="apis"
                  hide-default-footer
                  :page.sync="page"
                  :search="search"
                  :loading="loadingTable"
                  :items-per-page="itemsPerPage"
                  @page-count="pageCount = $event"
                >
                  <template #top>
                    <v-row class="pa-2">
                      <v-col cols="12" xl="3" lg="4" md="5" sm="6">
                        <v-text-field
                          label="Search"
                          v-model="search"
                          :disabled="loadingTable"
                          append-icon="mdi-magnify"
                        ></v-text-field>
                      </v-col>
                      <v-spacer></v-spacer>
                      <v-col cols="12" xl="3" lg="4" md="5" sm="6">
                        <v-autocomplete
                          label="Category"
                          v-model="selectedCategory"
                          :disabled="loadingTable"
                          :items="categoryItems"
                          @change="changeCategory"
                        ></v-autocomplete>
                      </v-col>
                    </v-row>
                  </template>
                  <template #[`item.Cors`]="{ item }">
                    <div v-if="item.Cors === 'yes'">
                      <v-icon color="success"> {{ "mdi-check" }} </v-icon>
                    </div>
                    <div v-else-if="item.Cors === 'no'">
                      <v-icon color="error"> {{ "mdi-close" }} </v-icon>
                    </div>
                    <div v-else>
                      {{ item.Cors }}
                    </div>
                  </template>
                  <template #[`item.HTTPS`]="{ item }">
                    <div v-if="item.HTTPS === true">
                      <v-icon color="success"> {{ "mdi-check" }} </v-icon>
                    </div>
                    <div v-else>
                      <v-icon color="red"> {{ "mdi-close" }} </v-icon>
                    </div>
                  </template>
                  <template #[`item.Link`]="{ item }">
                    <a :href="item.Link" target="_blank">{{ item.Link }}</a>
                  </template>
                </v-data-table>
                <v-pagination
                  v-model="page"
                  :length="pageCount"
                  :total-visible="11"
                ></v-pagination>
              </div>
            </v-fade-transition>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  data() {
    return {
      apis: [],
      search: "",
      loading: false,
      headers: [
        {
          text: "API",
          value: "API",
          sortable: false,
        },
        {
          text: "Auth",
          value: "Auth",
          sortable: false,
          filterable: false,
        },
        {
          text: "Category",
          value: "Category",
          sortable: false,
          filterable: false,
        },
        {
          text: "Cors",
          value: "Cors",
          sortable: false,
          filterable: false,
        },
        {
          text: "Description",
          value: "Description",
          sortable: false,
          filterable: false,
        },
        {
          text: "HTTPS",
          value: "HTTPS",
          sortable: false,
          filterable: false,
        },
        {
          text: "Link",
          value: "Link",
          sortable: false,
          filterable: false,
        },
      ],
      page: 1,
      pageCount: 0,
      itemsPerPage: 10,
      categoryItems: ["All"],
      selectedCategory: null,
      loadingTable: false,
      errorAlert: false,
    };
  },
  async created() {
    this.selectedCategory = this.categoryItems[0];
    this.loading = true;
    await this.loadApis();
    this.loading = false;
  },
  methods: {
    async loadApis() {
      this.errorAlert = false;
      try {
        const response = await axios.get("https://api.publicapis.org/entries");
        this.apis = response.data.entries;
        this.apis.forEach((api) => {
          if (!this.categoryItems.includes(api.Category)) {
            this.categoryItems.push(api.Category);
          }
        });
      } catch (error) {
        this.errorAlert = true;
        console.log(error);
      }
    },
    async changeCategory() {
      this.loadingTable = true;
      await this.loadApis();
      if (!this.selectedCategory.includes("All")) {
        this.apis = this.apis.filter((api) => {
          if (api.Category.includes(this.selectedCategory)) {
            return api;
          }
        });
      }
      this.loadingTable = false;
    },
  },
};
</script>
