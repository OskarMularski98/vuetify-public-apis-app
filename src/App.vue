<template>
  <v-app>
    <v-app-bar app>
      <v-spacer></v-spacer>
      <v-toolbar-title>Public APIs</v-toolbar-title>
      <v-spacer></v-spacer>
    </v-app-bar>
    <v-main>
      <v-container>
        <v-row>
          <v-col cols="12">
            <div v-if="loading" class="text-center">
              <v-progress-circular
                indeterminate
                color="primary"
              ></v-progress-circular>
            </div>
            <v-fade-transition hide-on-leave>
              <div v-if="!loading">
                <v-data-table
                  :headers="headers"
                  :items="apis"
                  hide-default-footer
                  :page.sync="page"
                  :search="search"
                  :items-per-page="itemsPerPage"
                  @page-count="pageCount = $event"
                >
                  <template #top>
                    <v-col cols="3">
                      <v-text-field
                        label="Search"
                        v-model="search"
                      ></v-text-field>
                    </v-col>
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
                  :total-visible="10"
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
      length: null,
      page: 1,
      pageCount: 0,
      itemsPerPage: 10,
      total: null,
    };
  },
  async created() {
    this.loading = true;
    await this.loadApis();
    this.loading = false;
  },
  methods: {
    async loadApis() {
      try {
        const response = await axios.get("https://api.publicapis.org/entries");
        this.total = response.data.count;
        this.apis = response.data.entries;
        this.length = Math.ceil(this.total / this.itemsPerPage);
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>
