<template>
  <v-data-table
    :headers="headers"
    :items="users"
    sort-by="calories"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar flat color="white">
        <div class="flex-grow-1"></div>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on }">
            <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6" md="6">
                    <v-text-field v-model="editedItem.name" label="Name"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="6">
                    <v-text-field v-model="editedItem.lastname" label="Lastname"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-select v-model="editedItem.sex" :items="items" :rules="[v => !!v || 'Item is required']" label="Sex" required></v-select>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-menu
                      ref="menu"
                      v-model="menu"
                      :close-on-content-click="false"
                      :return-value.sync="date"
                      transition="scale-transition"
                      offset-y
                      full-width
                      min-width="290px"
                    >
                      <template v-slot:activator="{ on }">
                        <v-text-field
                          v-model="editedItem.birthday"
                          label="Birthday"
                          readonly
                          v-on="on"
                        ></v-text-field>
                      </template>
                      <v-date-picker v-model="editedItem.birthday" no-title scrollable>
                        <div class="flex-grow-1"></div>
                        <v-btn text color="primary" @click="menu = false">Cancel</v-btn>
                        <v-btn text color="primary" @click="$refs.menu.save(editedItem.birthday)">OK</v-btn>
                      </v-date-picker>
                    </v-menu>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.email" :rules="emailRules" label="Email" required></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="12">
                    <v-textarea v-model="editedItem.address" label="Address"></v-textarea>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field v-model="editedItem.telnumber" label="Telnumber"></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <div class="flex-grow-1"></div>
              <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="save">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.action="{ item }">
      <v-btn style="right:1%; top:3px;" color="success" dark class="mb-2" @click="editItem(item)">Edit</v-btn>
      <v-btn style="top:3px;" color="red" dark class="mb-2" @click="deleteItem(item)">Delete</v-btn>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize">Reset</v-btn>
    </template>
  </v-data-table>
</template>

<script>
  import axios from "axios";
  export default {
    data: () => ({
      date: new Date().toISOString().substr(0, 10),
      menu: false,
      modal: false,
      menu2: false,
      email: '',
      emailRules: [
        v => !!v || 'E-mail is required',
        v => /.+@.+\..+/.test(v) || 'E-mail must be valid',
      ],
      select: null,
      items: [
        'male',
        'female',
      ],
      dialog: false,
      headers: [
          { text: 'Name', value: 'name' },
          { text: 'Lastname', value: 'lastname' },
          { text: 'Sex', value: 'sex' },
          { text: 'Birthday', value: 'birthday' },
          { text: 'Email', value: 'email' },
          { text: 'Address', width: '200', value: 'address' },
          { text: 'Telnumber', value: 'telnumber' },
          { text: 'Actions', width: '250', value: 'action', sortable: false },
        ],
      users: [],
      editedIndex: -1,
      editedItem: {
        name: '',
        lastname: '',
        sex: '',
        birthday: '',
        email: '',
        address: '',
        telnumber: '',
      },
      defaultItem: {
        name: '',
        lastname: '',
        sex: '',
        birthday: '',
        email: '',
        address: '',
        telnumber: '',
      },
    }),

    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
    },

    created () {
      this.initialize()
    },

    methods: {
      initialize () {
        axios.get("http://localhost/user_management_backend/v1/public/user")
        .then(response => {
           this.users = response.data.data
        })
      },

      editItem (item) {
        this.editedIndex = this.users.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        const index = this.users.indexOf(item)
        const jsondata = {
            id: item.id,
        };
        confirm('Are you sure you want to delete this item?') && axios.post('http://localhost/user_management_backend/v1/public/deluser', jsondata)
        .then(response => {
          window.location.reload()
        });
      },

      close () {
        this.dialog = false
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        }, 300)
      },

      save () {
        if (this.editedIndex > -1) {
          const jsondata = {
              id: this.editedItem.id,
              name: this.editedItem.name,
              lastname: this.editedItem.lastname,
              sex: this.editedItem.sex,
              birthday: this.editedItem.birthday,
              email: this.editedItem.email,
              address: this.editedItem.address,
              telnumber: this.editedItem.telnumber,
          };
          axios.post('http://localhost/user_management_backend/v1/public/updateuser', jsondata)
          .then(response => {
            window.location.reload()
          });
        } else {
          const jsondata = {
              name: this.editedItem.name,
              lastname: this.editedItem.lastname,
              sex: this.editedItem.sex,
              birthday: this.editedItem.birthday,
              email: this.editedItem.email,
              address: this.editedItem.address,
              telnumber: this.editedItem.telnumber,
          };
          axios.post('http://localhost/user_management_backend/v1/public/adduser', jsondata)
          .then(response => {
            window.location.reload()
          });
        }
        this.close()
      },
    },
  }
</script>